---
title: Raspberry Pi（1）：启动和远程访问
urlname: raspberry-pi-1-boot-and-remote-access
toc: true
date: 2018-10-01 16:18:00
updated: 2018-10-01 16:49:00
tags: [Raspberry Pi]
---

今年由于各种各样的原因选了《嵌入式系统》这门课，一部分原因是因为我小时候总觉得搞计算机的都应该会用这种开发板搞各种神奇的嵌入式开发。事实可能不是这样，不过这门课大概还挺有意思的，难度应该比OS和计原稍微小一点吧……

![老师发的Raspberry Pi套件](raspberry-suite.png)

套件包括：

* Raspberry Pi 3 Model B+一个，包括四个USB口、一个网线接口、一个microUSB电源接口、一个HDMI口和一个耳机（大概）接口，还有摄像头
* 32GB TF卡一个
* USB读卡器一个
* 电源线一个
* 电源线转接头一个

## 第一次启动树莓派

首先需要把系统映像烧到TF卡里，可以使用[Etcher](https://etcher.io/)或者[Win32DiskImager](http://sourceforge.net/projects/win32diskimager/)。结果在这里我干了一件蠢事。我不小心在官网上下载了[Raspberry Pi Desktop (for PC and Mac)](https://www.raspberrypi.org/downloads/)，而不是用于树莓派自己的[Raspbian](https://www.raspberrypi.org/downloads/raspbian/)。结果往TF卡上烧了几次，树莓派都没什么反应。至于“反应”……树莓派左下角有两个灯，“PWR”和“ACT”，分别表示是否通电和工作状态。正常工作状态下，PWR应该常亮。树莓派没有开关机按钮，它会在加电时直接自动启动。如果能够读取TF卡，则ACT会开始闪烁；否则ACT会保持不动。[^booting]

[^booting]: [STICKY: Is your Pi not booting?](https://www.raspberrypi.org/forums/viewtopic.php?t=58151)

总之最后我终于发现自己下错镜像了。

第一次启动树莓派的时候必须要连接显示器和鼠标键盘，否则不能完成必要的设置。键盘和鼠标直接插USB就行；一般来说，需要一个VGA转HDMI转接头。

![显示器上的树莓派桌面](desktop.jpg)

设置主要包括：

* 地点和时区
* 密码
* Wifi
* 打开串口、Camera、SSH等开关
* （好像没了）

其中，我开始的时候没有连上Wifi，不过也算是设置完了。

## 使用VNC Viewer远程访问树莓派

树莓派和电脑如何通过网络连接是一个比较大的问题。这次的推荐方法是把树莓派和电脑都连接到手机热点上，然后在电脑上用VNC Viewer远程访问树莓派。所以首先需要连接到手机热点。但是在这一步我又因为热点名称里有中文字符而踩了一点坑，最后只好改了手机的名称。[^wifi]

[^wifi]: [Raspberry Pi 3 can't connect to iOS' Personal Hotspot](https://stackoverflow.com/questions/45836528/raspberry-pi-3-cant-connect-to-ios-personal-hotspot)

通过在树莓派上执行`ifconfig wlan0`，可以知道树莓派在局域网内的地址。然后就可以在电脑端的VNCViewer上输入地址进行远程访问了。

![VNC Viewer](vnc.png)

不过我现在的问题是，这个地址是否会变化。重启了一次之后，发现暂时没有发生变化。不知道是为什么。还需要配置静态IP之类的吗？
