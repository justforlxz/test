---
title: 在ArchLinux通过串口调试VMware虚拟机中的deepin
s: use-serial-port-debug-deepin-on-archlinux
date: 2019-12-26 17:26:13
tags: Linux
categories: Linux
---

电脑主板上的接口：进行串行传输的接口，它一次只能传输1Bit。串行端口可以用于连接外置调制解调器、绘图仪或串行打印机。它也可以控制台连接的方式连接网络设备，例如路由器和交换机，主要用来配置它们。消费性电子已经由USB取代串列接口；但在非消费性用途，如网络设备等，串列接口仍是主要的传输控制方式。

<!-- more -->

首先给虚拟机分配一个串口设备，选择Settings->Add->Serial Port。分配好串口设备以后，我们需要选择一个串口设备的调试方式，一个是将输出转向一个文件，或者是通过socket。

如果只是查看方式，选择outpu file即可。如果需要调试，则可以通过socket方式来进行。

socket方式需要给一个固定的路径分配/tmp/<socket>，我调试的时候给出的是/tmp/vhost，From选择Server，To选择An Application。From的意思是信息从哪里来，信息是虚拟机里的系统发出的，所以这里选择的是Server，如果是反向操作，需要选择Client。To也是有两个选项，第一个是An Virtual Machine，第二个是An Application。用于把消息发送给另外的虚拟机，或者是宿主机的一个应用程序。

安装minicom包，用于进行调试，minicom这个东西，不是太好用，退出方式是先按Ctrl+A，然后按q，有时候还不一定管用，不知道是没接受到，还是按错了。

先minicom -s 进行初始化，选择`Serial port setup`，按A编辑`Serial Device`，这里需要注意一下，通过socket进行调试，需要使用`unix#`前缀，然后加上在虚拟机里写的路径 `unix#/tmp/vhost`。然后保存，选择Exit，退出以后其实重启minicom，就进入minicom的调试界面了，然后此时开启虚拟机，给内核添加一个console=ttyS0的参数，就看到minicom显示输出的信息了，还可以交互。


```
[    3.855725] [drm:vmw_fb_setcolreg [vmwgfx]] *ERROR* Bad regno 254.
[    3.857125] [drm:vmw_fb_setcolreg [vmwgfx]] *ERROR* Bad regno 255.
deepin Login:

CTRL-A Z for help | unix-socket | NOR | Minicom 2.7.1 | VT102 | Offline | unix#/tmp/vhost
```

此时就可以交互了，用法和tty一样，最后一行是minicom的输出，可以看到CTRL-A Z可以看help，minicom的版本，和访问的串口socket。
