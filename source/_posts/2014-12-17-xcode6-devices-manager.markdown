---
layout: post
title: "Xcode6 - Devices Manager"
date: 2014-12-17 17:33:07 +0800
comments: true
categories: Xcode
---
#背景

今天用XCode6 建了一个工程，在编译的时候发现target只能选择'iOS Devices',没有任何的simulator可以选择。如下图所示

![](/images/2014-12-17.0.png)

在手动打开simulator的时候出现错误对话框“Unable to determine device.” 升级了mac os系统以及重装了XCode都不能解决。最后是通过Simulator的‘Device Management' 解决的。
<!--more-->

#错误产生原因
在Xcode6中, application 的 folder改变了。在iOS8 之前，app documents location 如下：

<code>~/Library/Application Support/iPhone Simulator/ {iOS Version} /Applications/ {Application ID} /Documents </code>

在Xcode6，simulator的folder改为：

<code>~/Library/Developer/CoreSimulator/Devices/XXXXXXXX</code>

Devices 下对应着各个版本的simulator folder。我在想清空app documents folder的时候，误操作，把整个devices folder给清空了。就导致Xcode无法加载simulator folder，所以无法启动相应的simulator

#解决方案
通过Simulator的Device Manger可以解决。打开Device Manger 通过 打开Simulator，菜单栏Hardware->Device->Manage Devices... 如下图所示。

![](/images/2014-12-17.1.png)

在这里可以添加Xcode6支持的simulator。添加完毕之后查看’~/Library/Developer/CoreSimulator/Devices‘ folder，可见被误删掉的simulator devices都添加回来。重装了Xcode6之后，并没有自动添加这些devices，还是需要手动在Devices Manager中添加。

![](/images/2014-12-17.2.png)

如果想要查看当前支持的simulator types，可以用如下命令

<code>
:~$ xcrun simctl list
</code>

App Documents folder 是在：

<code>~/Library/Developer/CoreSimulator/Devices/XXXXX/data/Containers/Data/Application/XXXXX/Documents/</code>