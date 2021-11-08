---
title: 记录一个坑爹的usb网卡
date: 2019-12-09 19:31:04
tags: Linux
categories: Linux
---

网卡型号是Realtek RTL8811CU/RTL8821CU USB Wi-Fi adapter，买来是为了让黑苹果上网的，windows下也会自动下载和安装驱动，但是linux比较难受，内核不提供这样的驱动，只能去官方拿源码搞，今天在arch上打算装一下驱动，结果遇到了很多问题。

wiki上推荐的8821应该使用rtl88xxau-aircrack-dkms-git，但是我安装以后压根不能用，一点变化都没有，而且modprobe也没有作者给出的88XXau，无奈只得放弃。

继续谷歌之，在[https://forum.mxlinux.org/viewtopic.php?f=107&t=50579](https://forum.mxlinux.org/viewtopic.php?f=107&t=50579)看到了别人给的方案，然后果断clone并make,然后就因为没有适配5.x的内核编译失败，这可不行，翻了一下issue，看到作者在[https://github.com/whitebatman2/rtl8821CU/issues/33](https://github.com/whitebatman2/rtl8821CU/issues/33)提到了一个[#23](https://github.com/whitebatman2/rtl8821CU/issues/23)，这标题写的够可以，`Newer version 5.4.1 (Support Linux versions from 4.4.x up to 5.4.x) `，赶紧搞起，去源地址clone和make,成功使用上了驱动，按照作者提到的安装`usb_modeswitch`，并切换usb模式，我成功的使用上了这个usb网卡。

> 吐槽一下，开发环境还是linux下舒服，仓库的包安装一下就可以开发了，windows下要自己写路径，mac下brew限制太死，一些库安装以后还要自己手动做些处理，一不小心就把shell的环境变量搞不行了，或者压根不能正常工作。
