---
title: Debian虚拟机安装
date: 2016-04-21 21:56:26
categories: Linux
tags: Song
comments: true

---
<font color = gray>第一次好好装linux系统，崩了无数次又挽救了回来。</font>
***
## 原料
+ VMware Workstation（虚拟机软件）
+ Debian 8 iso（原始系统是只用下载xxx-1，其他的包是拓展软件）
	+ 32位 DVD版:<http://cdimage.debian.org/debian-cd/8.4.0/i386/bt-dvd/>
	+ 32位 CD版：<http://cdimage.debian.org/debian-cd/8.4.0/i386/bt-cd/>

<!-- more -->

## 安装虚拟机
网上VMware的教程太多了，而且很简单，这里就不赘述。

## 配置Debian
刚安好的Debian真是什么都没有，所以需要我们从头开始配置。
### 简单介绍几个vi命令
	
	//输入
	i 在光标处输入
    shift+i 在行首输入
	//删除
	x 删除光标处
	//保存
	:w 保存当前修改文本
	//退出
	:q 直接退出
	:q! 不保存退出

### 修改源
源是你Debian下载很多软件的一个入口，需要好好的配置。
	
	//进入源配置
	vi /etc/apt/sources.list
	//运用vi命令添加以下源
	deb http://mirrors.163.com/debian jessie main non-free contrib 
	deb http://mirrors.163.com/debian jessie-proposed-updates main contrib non-free 
	deb http://mirrors.163.com/debian-security jessie/updates main contrib non-free 
	deb http://security.debian.org jessie/updates main contrib non-free
	//注释命令
	（如果apt-get install xxx 时，报错 Media change: please insert the disc labeled）
	在cdrom那行前面加 # 进行注释
	
	$sudo aptitude update
	$sudo aptitude upgrade
	$sudo aptitude dist-upgrade

### root

	su root
	输入root密码

### 安装桌面环境
	
	//先安装X系统
	apt-get install x-window-system-core
	//安装GNOME桌面环境
	apt-get install gnome
	//运行gdm3或者重启进入图形界面

### 安装VMware Tools

	//进入虚拟系统中
	//点击虚拟机->安装VMwareTools
	//进入虚拟系统中的光驱VMwareTools
	//将压缩文件拖到tmp文件夹下
	//打开终端，进入root权限
	
	cd tmp
	tar xvf 压缩文件

	//进入解压的文件夹
	./vmware-install.pl
	//一路回车就行
	
	
	