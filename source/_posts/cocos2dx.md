---
title: cocos2dx
date: 2016-04-20 00:50:26
categories: C++
tags: Song
comments: true

---
<font color = gray>第二次使用cocos2dx引擎，写完项目赶紧补一篇博客以免自己忘记，下次又要重头看起，以下均为自己的理解，欢迎指针。以后学习到新的cocos2dx引擎知识会继续来维护这篇博客。</font>
***

## cocos2dx引擎
+ cocos2dx通过一个导演类来播放整个游戏场景
+ 每个游戏场景是一个Scene类
+ 一个场景下可以添加多个幕布Layer类切换（每次只能展示一个幕布）
+ 一个幕布上可以添加多个精灵Sprite类
+ 所有用户交互都是通过添加监听来实现，一个监听布局一个线程

<!-- more -->

## Sprite类
精灵类是cocos2dx当中用的最多的类，继承自Node类。可以作为背景、NPI、路人甲乙丙丁等等。

+ ***Sprite类创建***

	```C++

		//创建空纹理的精灵
		Sprite * sprite = Sprite::create();
		//用文件名创建Sprite（注意这里是std::string）
		Sprite * sprite = Sprite::create(const std::string &filename);
		
		//利用缓存纹理图重新给精灵贴图
        CCTexture2D* texture = CCTextureCache::sharedTextureCache()->addImage(filename);
		sprite = setTexture(texture);

		//设置精灵是否可见
		sprite->setVisible(true/false);

	```
+ ***Sprite类动画***
	
	```C++
		
		//利用自带的帧动画类
		CCAnimation* animation = CCAnimation::create();
		//将4帧图片加入帧动画
		animation->addSpriteFrameWithFileName(filename_1);
		animation->addSpriteFrameWithFileName(filename_2);
		animation->addSpriteFrameWithFileName(filename_3);
		animation->addSpriteFrameWithFileName(filename_4);
		//4帧播放2秒
		animation->setDelayPerUnit(2.0f / 4.0f);
		//在动画结束后是否回到第一张图
		animation->setRestoreOriginalFrame(true);
		//循环次数，-1是无限循环
		animation->setLoops(-1);
		//这里新建一个播放帧动画的anction
		CCFiniteTimeAction * animate = CCAnimate::create(animation);
		sprite->runAction(animate);

		//cocos2dx自带的动作
		//移动到point需要5s,reverse可以获取反动作
		CCActionInterval * anction = CCMoveBy::create(5,point);
		CCActionInterval * reAnction = anction->reverse();
		//延迟3s
		CCActionInterval * anction = CCDelayTime::create(3);
		//多个动作依次执行
		CCActionInterval * move = CCMoveTo::create(10,point);
		CCActionInterval * scale = CCScaleTo::create(2,3);
		CCFiniteTimeAction * spawn = CCSpawn::create(move,scale,NULL);
		//重复动作
		CCFiniteTimeAction * repeat = CCRepeat::create(spawn,3);
		//无限重复
		CCFiniteTimeAction *repeatForever = CCRepeatForever::create((CCActionIntervar *)spawn);
		//回调动作（执行完动作后响应事件）
		CCCallFuncND * funcall = CCCallFuncND::create(this,callfuncND_selector(HelloWorld::callbackND));
		CCFiniteTimeAction * seq = CCSequence::create(move,funcall,NULL);
		//带一个参数的回调函数（可以不带）
		void HelloWorld::callbackN(CCNode* sender)
		{
			CCLOG("hhh");
		}

		//停止制定动作
		sprite->stopAction(animate);
		//停止所有动作
		sprite->stopAllAction();
		(官方文档中说的并可以删除动作,但是我好像并没起到删除的效果)

		//因为不会做骨骼动画，项目里遇到一个尴尬的问题，我需要精灵一边播放帧动画，一边进行移动。也就是有动作的走路，23333
		//这里我使用的办法是，让精灵播放帧动画然后添加一个定时器，每一帧都改变精灵的坐标。
    ```
+ ***Sprite类继承***

	很多时候都需要自定义一个继承Sprite类的精灵，那要怎么重新封装呢？
	
	```C++
	
		Class Human:public Sprite
		{
			//重构create方法
			public: static  Human * create(const char *filename);
		}
		
		Human * Human::create(const char *filename)
		{
			Human *sprite = new Human();
			if (sprite && sprite->initWithFile(filename))
			{
				sprite->autorelease();
				return sprite;
			}
			CC_SAFE_DELETE(sprite);
			return nullptr;
		}
	```

## 定时器
这个东西的出场率也是非常的爆炸，可以完成你想要的很多功能。虽然cocos2dx官方给出了三种定时器，但是我觉得用schedule足够了。将它添加到当前节点中，程序会每帧都自动执行update函数，现在update函数已经可以自己取名了，schedule都认识。如果想自己设定间隔时间也是可以自己设定的。

+ ***schedule定时器***

	```C++
		
		//每隔1s执行一次updata函数
		this->schedule(schedule_selector(HelloWorld::updata),1.0f);

		void HelloWorld::updata(float dt)
		{
			CCLOG("hhhh");
		}
		
		//关闭定时器
		this->unschedule(schedule_selector(HelloWorld::updata));
		
		(可以在一个定时器当中关闭自己，但是如果这样做，他会把这一次自己做完再关闭)
	```

## TextField
TextFieldTTF类提供给开发者一个文本显示、输入框。

+ ***TextFieldTTF创建***
	
	```C++

		//默认内容为“value”，字体为“arial”，字体大小为20号
		TextFieldTTF *ttf = TextFieldTTF::textFieldWithPlaceHolder("Value", "arial", 20);

		（字体是Resource文件夹下fonts文件夹下的ttf，可以自己添加）

	```
+ ***TextFieldTTF事件响应***

	```C++
		
		//有两个输入框nowTF和finalTF
		void HelloWorld::addListener()
		{
			//获得导演
			auto director = Director::getInstance();
			//触控事件处理闭包
			auto handler = [=](Touch *t,Event *e)
			{
				auto target = e->getCurrentTarget();
				//如果触控点在cocos2dx的框体内
				if (target->getBoundingBox().containsPoint(t->getLocation()))
				{
					//如果在触控点在nowTF输入框内，执行输入（键盘输入）
					if (nowTF == target)
					{
						nowTF->attachWithIME();
					}
					else if (finalTF == target)
					{
						finalTF->attachWithIME();
					}
				}
				return false;
			};

			//新建touch事件，单点触控
			auto aTf = EventListenerTouchOneByOne::create();
			aTf->onTouchBegan = handler;
			//在该导演节点中添加事件响应
			director->getEventDispatcher()->addEventListenerWithSceneGraphPriority(aTf, nowTF);

			auto bTf = EventListenerTouchOneByOne::create();
			bTf->onTouchBegan = handler;
			director->getEventDispatcher()->addEventListenerWithSceneGraphPriority(bTf, finalTF);
		}
	```

## Cocos2d::Vector
这是cocos2dx自己重新封装的Vector类，感觉还是挺好用的。

+ ***添加删除新元素***
	
	```C++
		
		Sprite * sprite = Sprite::create();
		Vector <Sprite *> vector;
		Vector <Sprite *> vector_2;
		//添加到末尾
		vector.pushBack(sprite);
		//在一个确定的位置插入对象
		vector.insert(0,sprite);
		//插入另一个vector
		vector.pushBack(*vector_2);

		//移除指定位置元素
		vector.erase(1);
		//移除最后一个元素
		vector.popBack();
		//移除某个元素
		vector.eraseObject(sprite,true);
		//移除所有
		vector.clear();	
	```
+ ***其他常用操作***
	
	```C++
		
		//交换两个元素,用下标
		vector.swap(0,1);
		//也可以不用下标
		vector.swap(vector.front(),vector.back());
		//查找某元素
		Int wz = vector.find(sprite);
		//询问某元素是否在vector中
		Bool flag = vector.contains(sprite);
	```

# 未完待续