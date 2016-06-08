---
title: 内存分配
date: 2016-05-13 10:58:26
categories: 作业
tags: Song
comments: true

---
***

## 项目要求
### 基本任务
+ 数据结构、分配算法
+ 加深对动态分区存储管理方式及其实现过程的理解。
### 基本需求
假设初始态下，可用内存空间为640K，并有下列请求序列，请分别用首次适应算法和最佳适应算法进行内存块的分配和回收，并显示出每次分配和回收后的空闲分区链的情况来。

<!-- more -->

+ 作业1申请130K
+ 作业2申请60K
+ 作业3申请100k
+ 作业2释放60K
+ 作业4申请200K
+ 作业3释放100K
+ 作业1释放130K
+ 作业5申请140K
+ 作业6申请60K
+ 作业7申请50K
+ 作业6释放60K
## 项目实现
### 开发环境
**C++** + **cocos2dx**

### 分配算法设计
#### 分配算法分析
+ **最先适配算法**
	+ **算法思想：**按分区先后次序从头查找，找到符合要求的第一个分区。
	+ **算法实质：**尽可能利用存储区低地址空闲区，尽量在高地址部分保存较大空闲区，以便一旦有分配大空闲区要求时，容易得到满足
	+ **算法优点：**分配简单，合并相邻空闲区也比较容易
	+ **算法缺点：**查找总是从表首开始，前面空闲区往往被分割的很小，满足分配要求的可能性较小，查找次数较多。
+ **循环最先适配算法**
	+ **算法思想：**按分区先后次序，从上次分配的分区起查找（到最后分区时再回到开头），找到符合要求的第一个分区
	+ **算法特点：**算法的分配和释放的时间性能较好，使空闲分区分布得更均匀，但较大的空闲分区不易保留。
+ **最佳适配算法**
	+ **算法思想：**在所有大于或者等于要求分配长度的空闲区中挑选一个最小的分区，即对该分区所要求分配的大小来说，是最合适的。分配后，所剩余的块会最小。
   	+ **算法实现：**空闲存储区管理表采用从小到大的顺序结构
   	+ **优点：**较大的空闲分区可以被保留。
   	+ **缺点：**空闲区是按大小而不是按地址顺序排列的 ，因此释放时，要在整个链表上搜索地址相邻的空闲区，合并后，又要插入到合适的位置。
+ 最坏适配算法
	+ **算法思想：**分区时取所有空闲区中最大的一块，把剩余的块再变成一个新的小一点的空闲区。
	+ **算法实现：**空闲区按由大到小排序。
	+ **优点：**分配时，只需查找一次就可成功，分配算法很快。
	+ **缺点：**最后剩余分区会越来越小，无法运行大程序
### 代码实现
在实现过程当中，我需要把每一步任务的操作都反映在列表和视图模块当中。**所以我定义了task类来描述所有的任务，freeMemory类来描述所有的空内存块。**

```c++

	class task :
	public Node
	{
	private:
		//任务的编号
		int _taskNum;
		//任务占内存的大小
		int _taskLong;
		//任务的操作类型
		int _type;
	public:
		//带参数的构造函数
		task(int taskNum, int taskLong, int type);
		//获取任务的编号
		int getTaskNum();
		//获取任务的内存大小
		int getTaskLong();
		//获取任务的操作类型
		int getType();
		task();
		~task();
	};

	class freeMemory :
	public Node
	{
	private:
		//内存块的开始位置
		int _start;
		//内存块的大小
		int _long;
		//内存块的类型  0 为空内存 1 为被占内存 2 为非内存
		int _type;
	public:
		//获取内存块的大小
		int getLong();
		//获取内存块的开始位置
		int getStart();
		//获取内存块的类型
		int getType();
		//修改内存块的开始位置
		void setStart(int start);
		//修改内存块的大小
		void setLong(int l);
		//修改内存块的类型
		void setType(int type);
		//带参数的构造函数
		freeMemory(int start, int l,int type);
		freeMemory();	
		~freeMemory();
	};
```

在实现过程当中，设置了：

+ **taskArray数组 :** 储存任务
+ **freeMemoryArray数组 :** 储存空内存块
+ **nodeArray数组 :** 储存内存块的显示
+ **workArrayNum :** 该映射表用来储存这个任务在nodeArray中的位置
+ **taskArrayNum :** 该映射表用来储存这个任务在freeMemoryArray中的位置

#### 首次适应算法
+ **增加任务时：**扫描freeMemoryArray数组，找到第一个能够存放该任务的空内存块。然后将其放入该内存块当中，freeMemoryArray数组的这个位置分解为两部分内存块，一部分为该任务所占据的内存块，一部分为剩下的空内存块。然后在视图区添加一个有色方块，并利用操作后的freeMemoryArray数组更新列表区。

#### 最佳适应算法
+ **增加任务时：**扫描freeMemoryArray数组，找到能存放该任务的空内存块中最小的一个。然后将其放入该内存块当中，freeMemoryArray数组的这个位置分解为两部分内存块，一部分为该任务所占据的内存块，一部分为剩下的空内存块。然后在视图区添加一个有色方块，并利用操作后的freeMemoryArray数组更新列表区。

#### 释放任务
通过workArrayNum数组和taskArrayNum数组，分别找到该任务在nodeArray数组和freeMemoryArray数组中的位置。然后在nodeArray数组中删除该方块，<font color = red>**在freeMemoryArray数组中从任务所在位置往前扫描找到第一个类型不为2的方块，如果这个类型不为2的方块的类型为0，这两个方块合并。再从任务所在位置往后扫描找到第一个类型不为2的方块，如果这个类型不为2的方块的类型为0，这两个方块合并。**</font>

### 界面展示

#### 界面功能
+ **主页面**<p>之前本来以为要做两种分区，所以做了主页面，但是现在只用完成一种算法，所以只为第一个按钮做了功能，点击之后进入动态分区模拟的页面。<p>
![pic1](http://o6wcj5apx.bkt.clouddn.com/memoryDesk.png)
+ **动态分区模拟页面**
	+ **返回按钮：**左上角为返回按钮，点击后退回主页面。
	+ **算法切换按钮：**右上角两个为算法切换按钮，可以点击切换算法。
	+ **执行任务按钮：**点击一次后执行一条任务。
	+ **内存展示列表：**会展示出所有空内存块，会列出内存块标号，起始位置，内存块大小。
	+ **任务展示列表：**会展示出所有已经执行的任务，会列出任务标号，需要内存大小，操作类型。

![pic2](http://o6wcj5apx.bkt.clouddn.com/memory1-0.png)

#### 首次适应算法
+ **作业1申请130K**<p>
![pic3](http://o6wcj5apx.bkt.clouddn.com/memory1-1.png)
+ **作业2申请60K**<p>
![pic4](http://o6wcj5apx.bkt.clouddn.com/memory1-2.png)
+ **作业3申请100k**<p>
![pic5](http://o6wcj5apx.bkt.clouddn.com/memory1-3.png)
+ **作业2释放60K**<p>
![pic6](http://o6wcj5apx.bkt.clouddn.com/memory1-4.png)
+ **作业4申请200K**<p>
![pic7](http://o6wcj5apx.bkt.clouddn.com/memory1-5.png)
+ **作业3释放100K**<p>
![pic8](http://o6wcj5apx.bkt.clouddn.com/memory1-6.png)
+ **作业1释放130K**<p>
![pic9](http://o6wcj5apx.bkt.clouddn.com/memory1-7.png)
+ **作业5申请140K**<p>
![pic10](http://o6wcj5apx.bkt.clouddn.com/memory1-8.png)
+ **作业6申请60K**<p>
![pic11](http://o6wcj5apx.bkt.clouddn.com/memory1-9.png)
+ **作业7申请50K**<p>
![pic12](http://o6wcj5apx.bkt.clouddn.com/memory1-10.png)
+ **作业6释放60K**<p>
![pic13](http://o6wcj5apx.bkt.clouddn.com/memory1-11.png)

#### 最佳适应算法
+ **作业1申请130K**<p>
![pic14](http://o6wcj5apx.bkt.clouddn.com/memory2-1.png)
+ **作业2申请60K**<p>
![pic15](http://o6wcj5apx.bkt.clouddn.com/memory2-2.png)
+ **作业3申请100k**<p>
![pic16](http://o6wcj5apx.bkt.clouddn.com/memory2-3.png)
+ **作业2释放60K**<p>
![pic17](http://o6wcj5apx.bkt.clouddn.com/memory2-4.png)
+ **作业4申请200K**<p>
![pic18](http://o6wcj5apx.bkt.clouddn.com/memory2-5.png)
+ **作业3释放100K**<p>
![pic19](http://o6wcj5apx.bkt.clouddn.com/memory2-6.png)
+ **作业1释放130K**<p>
![pic20](http://o6wcj5apx.bkt.clouddn.com/memory2-7.png)
+ **作业5申请140K**<p>
![pic21](http://o6wcj5apx.bkt.clouddn.com/memory2-8.png)
+ **作业6申请60K**<p>
![pic22](http://o6wcj5apx.bkt.clouddn.com/memory2-9.png)
+ **作业7申请50K**<p>
![pic23](http://o6wcj5apx.bkt.clouddn.com/memory2-10.png)
+ **作业6释放60K**<p>
![pic24](http://o6wcj5apx.bkt.clouddn.com/memory2-12.png)