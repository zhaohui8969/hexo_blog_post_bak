---
title: 在OSX上用automator实现VPN断线自动重连
date: 2016-02-23 17:11:43
tags: 
  - Automator
  - OSX
---


　　由于某些原因，平时需要使用到VPN，但是和这个服务器之间的链路呢总是有些不稳定，经常会断掉，然而OSX网络设置面板没有找到自动重播的选项，而且有时这个连接只是假死，就是还显示状态是连接上的，但是就是没有流量，得重播。可是手工去重播的话很麻烦，之前有试过在路由器上写过python脚本来实现链路连通状况检测自动重播的脚本，不过后来呢，那个路由器退役了，现在在OSX平台下想寻找到类似的解决办法，Google了解到OSX下有automator这样的神器。倒腾了一会解决了问题，在这里记录下来。

　　首先打开automator，他是这个样子的：　　

{% asset_img l2bf-010.png img1 %}

然后我们要制作一个应用程序，就像这样：

{% asset_img l2bf-020.png img2 %}

在“资料库”中选择“实用工具”分栏，找到“运行 AppleScript”这个东西，拖到右边，然后我们的代码编辑区就出现了，之后的所有代码都写在这里：

{% asset_img l2bf-030.png img3 %}

然后想想我需要实现什么，首先需要实现控制VPN连接的动作，Google到相关的资料，代码如下

```python
tell application "System Events"
	tell network preferences
		disconnect service "L2BF"
		connect service "L2BF"
	end tell
end tell
```
这段意思呢就是告诉"System Events模块"（我叫他模块，不知道对不对）里面的"network preferences模块"：我要断开一个叫L2BF的连接，然后重连一下，这里测试结果没有问题。

那么下一个部分就是网络的状态监测部分，这里呢我想到的最简单的办法就是用ping命令检测，然后我就找到一个在AppleScript里运行python程序的办法，代码如下：

```python
do shell script "ping -c 1 -W 3000 baidu.com"
```
这样就完成了一条python"语句."执行，对baidu.com发一个ping包，3000秒超时。然后就是关于逻辑判断的部分了，找到了这些我需要的部分:

### Try catch语句

```python
try
	{"语句."}
on error
	{"语句."}
end try
```
### 循环语句

```python
repeat
	{"语句."}
end repeat
```
当然，这里也可以有循环附加条件，比如这样：

```python
repeat 2 times
repeat with counter from 1 to 3
repeat while counter ≠ 10
repeat until counter = 10
repeat with 单个元素 in 集合
```
### 判断语句
也就类似这样

```python
if flag = 0 then
	{"语句."}
else if flag = -1 then
	{"语句."}
end if
```

### 变量什么的
```python
set flag to 0
```

然后代码就这样写出来了：

```python
on run {input, parameters}
	
	set flag to 0
	repeat
		try
			do shell script "ping -c 1 -W 3000 baidu.com"
			delay 30
		on error
			beep
			tell application "System Events"
				tell network preferences
					disconnect service "L2BF"
					connect service "L2BF"
				end tell
			end tell
			delay 5
		end try
	end repeat
	
	return input
end run
```
每隔30秒检测一下连通性，出现问题就重拨一下，看起来很不错，点一下这个按钮测试一下：

结果很不错，至少我把连接断掉他能自动给我连上。到这里就功德圆满了吗？不不不，在查找资料的过程中我看到了这个逼格很高的东西，有人在代码里写了一句这个:

```python
say "Good~"
```
我也试了一下，当时就懵逼了，这还能语音说话。。。然后就打算提高一下这个程序的逼格，加入语音的部分，还去系统语音里面选了一个不错的女声Vicki：

加了语音和提示音后的程序代码：

```python
on run {input, parameters}
	
	say "Demon started."
	set flag to 0
	repeat
		try
			do shell script "ping -c 1 -W 3000 baidu.com"
			if flag = 0 then
				say "Connected."
			else if flag = -1 then
				say "Reconnected!"
			end if
			set flag to 1
			delay 10
		on error
			beep
			set flag to -1
			say "Connection error! Try reconnect..."
			tell application "System Events"
				tell network preferences
					disconnect service "L2BF"
					connect service "L2BF"
				end tell
			end tell
			delay 5
		end try
	end repeat
	
	return input
end run
```
运行之后还有语音提示，666.

### 标准结局
Good！然后在Spotlight里搜索程序便可以直接打开：

{% asset_img l2bf-040.png img4 %}

运行之后在顶栏会有这样一个小齿轮，点击可以选择退出：

{% asset_img l2bf-050.png img5 %}


最后附上一些资料站：

- [AppleScript Language Guide](https://developer.apple.com/library/mac/documentation/AppleScript/Conceptual/AppleScriptLangGuide/introduction/ASLR_intro.html)
- [https://developer.apple.com/search/](https://developer.apple.com/search/)

Over
