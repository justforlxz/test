---
title: deepin待机后键盘和触摸板无法使用的解决方法
date: 2018-06-25 06:01:22
tags: Linux
---

笔记本一直使用的bumblebee来省电，毕竟我也不想笔记本的电只够从一张桌子移动到另一张桌子，但是今天在调待机唤醒后dde-dock崩溃的问题，我需要切换到私有驱动下，因为笔记本使用bumblebee需要使用acpi的参数，否则会见图形就死。

<!-- more -->

一切准备就绪以后，我开始调试dde-dock，通过codedump已经知道崩溃在wifi列表为空时访问了first节点，但是当我开始测试修复的代码时，发生了很意外的事情，恢复待机以后键盘和触摸板无法使用了。

虽然之前我也偶尔会用用私有驱动，但是还没遇到过无法键盘和触摸板无法使用的情况。想到论坛好像也有人报了类似的问题，恢复待机以后无wifi和外置键盘无法使用，刚好可以趁这个机会调试一下。

/var/log/Xorg.0.log里看到了大量的synaptics错误，然后该模块被卸载，键盘则是没看到什么信息。

尝试重新modprobe synaptics模块，但是失败了，然后在/etc/modprobe.d/nvidia.conf里看到了几行配置。

```
# These aliases are defined in *all* nvidia modules.
# Duplicating them here sets higher precedence and ensures the selected
# module gets loaded instead of a random first match if more than one
# version is installed. See #798207.
#alias	pci:v000010DEd00000E00sv*sd*bc04sc80i00*	nvidia
#alias	pci:v000010DEd00000AA3sv*sd*bc0Bsc40i00*	nvidia
#alias	pci:v000010DEd*sv*sd*bc03sc02i00*		nvidia
#alias	pci:v000010DEd*sv*sd*bc03sc00i00*		nvidia
```

似乎是通配出错了，匹配到了键盘和触摸板，然后就无法使用了。刚好deepin 15.6升级了nvidia驱动，所以是现在才会出这个问题。