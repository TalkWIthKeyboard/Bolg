---
title: 电梯调度
date: 2016-04-26 21:02:34
categories: 作业
tags: Song
comments: true

---
***
## 项目要求
### 基本任务
某一栋楼20层，有五部互联的电梯。基于**线程思想**，编写一个**电梯调度**程序。
### 基本需求
+ 五部电梯互相联结，即当一个电梯按钮按下去时，其他电梯相应按钮同时点亮，表示也按下去了。
+ 电梯调度算法：
	+ 所有电梯初始状态都在第一层
	+ 每个电梯没有相应请求的情况下，则应该在原地保持不动
	+ 电梯调度算法自行设计

<!-- more -->

## 项目实现
### 开发环境
**C++** + **cocos2dx**

### 调度算法设计
#### 算法初步分析
本项目存在三个调度问题。

+ 当多个楼层都在叫电梯时，如果都应该其中一个电梯响应，那么它应该以怎样的顺序进行接。**<font color = red>(电梯的接调度)</font>**
+ 当一辆电梯上有多个从不同楼层到达不同楼层的人时，电梯应该以怎样的顺序进行送。**<font color = red>（电梯的送调度)</font>**
+ 当人在一个楼层等电梯的时候，五部电梯当中的哪一部应该来响应人的请求。**<font color = red>（人的电梯调度)</font>**

#### 调度算法分析
我们先来对最常用的五种调度算法进行分析。

+ 先来先服务（FCFS）算法：**按照请求次序进行调度。**实现简单不用对队列进行调整，但是可能因某个元素处理时间过长，出现等待超时。
+ 最短作业优先（SJF）算法：**按照作业执行的消耗时间，消耗最短时间的先执行。**需要预先估计作业执行时间，耗时较长的作业可能一直得不到执行。
+ 基于优先权的调度（FPPS）算法：**给作业赋予一定优先权，每次从最高优先权的元素中取出一个并执行。**
+ 时间片轮转（RR）算法：**给每个作业赋予一个运行时间片，当时间片到则取下一个元素。**是对FCFS和SJF算法的折衷，不会出现超时。
+ 多级反馈队列（MFQ）算法：**对任务进行分类，不同分类放置到不同队列中，可能会采用不同的调度算法，队列中元素视情况会在不同队列之间迁移。**

通过分析可以发现，仅仅一、两种调度算法完全不适用于我们项目要求，所以我们需要对他们进行组合创新。
#### 算法设计
为了后文叙述的方便，我们先来设计多个变量。
***
+ 人所在的楼层 **humanMyFloor**
+ 人将去的楼层 **humanFinalFloor**
+ 人的行驶方向 **humanDirection** <font color = red>（-1 向下 1向上）</font>
+ 电梯所在的楼层 **liftMyFloor**
+ 电梯将去的楼层 **liftFinalFloor**
+ 电梯的行驶方向 **liftDirection** <font color = red>（-1 向下 0 静止 1向上）</font>
+ 电梯的运行队列 **dl**
+ 队列头指针 **op1**
+ 队列尾指针 **op2**
+ 为了方便再定义一种运算 **a\*(b > c) = (a\*b > a\*c)**

<font color = gray>(虽然行驶方向可以用其他变量计算获得，这里为了方便表达就先设置出来)</font>
***
现在我们来设计调度算法，通过分析我们可以发现这样的一个大问题，我们可以抽象成对运行队列的维护模型。所以后面的三种情况下的调度，我们就都将它们转化为对队列的维护来描述。<p>**这个队列定义为op1一直指向电梯的目标楼层，当op1>op2时，电梯静止。**

<div></div>
##### 电梯的接调度

根据我们的分析可以看出，电梯在接到一个人后，如果以同一行驶方向行驶能接到其他人的话，则去接。如果不行则先将此人送到目标楼层再去接人。<p><font color = blue>这时候其实就是考虑**humanMyFloor**这个值在队列中的插入位置，这时候需要遍历队列**op1-op2**，假设遍历指针是**sp**</font>

+ (dl[sp] - dl[sp-1]) * humanDirection > 0 && humanDirection * (dl[sp] > humanMyFloor > dl[sp-1]) **时将humanMyFloor插入到sp处，如果没找到则添加到末尾**

##### 电梯的送调度

根据我们的分析可以看出，电梯会沿着一个方向将能送的人送完后再调换方向。<p><font color = blue>这时候其实就是考虑**humanFinalFloor**这个值在队列中的插入位置，这时候需要遍历队列**op1-op2**，假设遍历指针是**sp**</font>

+ (dl[sp] - dl[sp-1]) * humanDirection > 0 && humanDirection * (dl[sp] > humanMyFloor > dl[sp-1]) **时将humanFinalFloor插入到sp处，如果没找到则添加到末尾**<p>

***
分析到现在我们可以发现一个问题，我们的队列的结构会非常的有规律。要么是一个单调不下降队列后紧接一个单调不上升队列、要么是一个单调不上升队列后紧接一个单调不下降队列。<p>![pic1](http://7xteei.com2.z0.glb.clouddn.com/1.png)<p>那这样的话，队列中仅维持在三个元素的情况下也就能解决问题，<font color = red>**换句话说中间的行驶过程我们只需要考虑在哪个节点转换方向即可，这样在大数据量的时候可以远远减少队列的长度。**</font>

***
根据以上的规律我们可以对队列结构进行优化**op2<=op1+1**,那么我们的队列会出现下方4种情况。<p>![pic2](http://7xteei.com2.z0.glb.clouddn.com/2.png)<p>![pic3](http://7xteei.com2.z0.glb.clouddn.com/3.png)<p>![pic4](http://7xteei.com2.z0.glb.clouddn.com/4.png)<p>![pic5](http://7xteei.com2.z0.glb.clouddn.com/5.png)<p>
根据这样的规律我们可以优化前面的两个调度算法，以电梯的接调度为例，送调度类似：

``` C++
	
	//处理电梯现在的行驶方向
	if (liftMyDirection == 0)
	{
		direction = (humanMyFloor - liftMyFloor) / abs(humanMyFloor - liftMyFloor)
	}
	else direction = liftMyDirection;

	//电梯行驶方向与人的行驶方向相同
	if (dl[op1] - liftMyDirection)*humanDirection > 0 && direction * (dl[op1] < humanMyFloor) 
	{
		dl[op1] = humanMyFloor;
	}
	//电梯行驶方向与人的行驶方向不同
	if (dl[op1] - liftMyDirection)*humanDirection <= 0 
	{
		if (op1 <= op2)
		{
			op2++;
			dl[op2] = humanMyFloor;
		}
		else if (humanDirection*(humanMyFloor > dl[op2]))
		{
			dl[op2] = humanMyFloor;
		}
	} 

```
##### 人的电梯调度
这样分析下来，人的电梯调度就很简单了，我们只需要将人与电梯的距离定义出来。

``` C++

	if (liftDirection == 0 || ( liftDirection != 0 && (dl[op1] - liftMyDirection)*humanDirection > 0) && (humanDirection * (liftMyDirection < humanMyFloor)))
	{
			distance = abs(humanMyFloor - liftMyDirection);			
	}
	else 
	{
			distance = abs(2 * dl[op1] - liftMyDirection - humanMyFloor);
	}
	
```

先回过头来想想，我们调度算法设计的过程当中体现了哪些调度思想呢。可以看到我们的调度算法中，使用的最频繁的一个思想就是**先来先服务**，虽然我们一直是在维护一个队列，但是可以看到我们添加队列元素的时候是按照先来先添加，先添加的元素会影响后面添加元素的决策的。这也和现实中的电梯调度非常的吻合。

<font color = green>**至此我们的调度算法也设计完成，我们就可以依次为理论基础来完成后面的实现。**</font><p>

<div></div>
### 代码实现

为了在图形界面上显示的简介一点，我利用电梯的楼层显示的变化来代替上下楼的概念。这样我设计了三个类：

+ **Human类**，用来描述乘坐电梯的人。
+ **Lift类**，用来描述电梯。
+ **numShow类**，用来描述电梯的楼层显示，一个numShow实例与一个Lift实例绑定。

整个程序的流程如下：

+ 在左上角两个输入框中输入，乘坐电梯的人现在在几楼，要去几楼。输入完成后点击右上角的确认按钮。<font color = blue>（这里利用TextFieldTFF、Menu控件配合点击事件和回调函数实现）</font>
+ 画面中会出现乘坐电梯的人，为这个人进行**人的电梯调度**，确定哪部电梯来接他，并进行这部**电梯的接调度**。然后他开始等待电梯。<font color = blue>（这里利用Sprite类和定时器实现）</font>
+ 等到该电梯到达该层时，电梯开门，人向电梯移动。进入电梯之后进行这部**电梯的送调度**<font color = blue>（这里利用CCAnimation类、CCFiniteTimeAction类和定时器实现）</font>
+ 电梯按照调度队列进行接送，完成所有任务后停止在某楼层。<font color = blue>（这里利用定时器实现）</font>
+ 在电梯的运行过程当中，电梯的楼层显示会一直跟随变化。<font color = blue>（这里利用CCAnimation类、CCFiniteTimeAction类和定时器实现）</font>


  
