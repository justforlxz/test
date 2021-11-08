---
title: debug了两天，只删了一行代码
date: 2017-08-16 18:25:39
tags:
---

前言： 项目一定要留一些文档！！ 修bug前一定要知道所有的流程！！！

<!-- more -->

这两天一直在修一个用户切换的bug，众所周知，deepin的多用户切换一直都不是正常工作的，确切来说是压根没有正常工作，还好用户不是经常切换，不然早就收到一大波的报告了。

dde-session-ui项目中包含了以下软件：

* dde-lock
* dde-shutdown
* dde-osd
* lightdm-deepin-greeter
* dde-switchtogreeter
* dde-suspend-dialog
* dde-warning-dialog
* dde-welcome
* dde-wm-chooser
* dde-lowpower
* dde-offline-upgrader

大部分项目根据名字就可以知道是做什么的，这是一个软件组的集合。
而dde-lock和lightdm-deepin-greeter二者有大量重复的功能和代码，这是它俩的工作性质决定的。

>lightdm-deepin-greeter: display-manager启动的实体，登录的界面是它负责的。

>dde-lock： 用户层面的屏幕锁定，基于我们的设计，和lightdm-deepin-greeter是大致相同的布局。

而且都包含了用户密码的验证，用户的切换，但是二者工作的层面是不同的，为了方便切换，就有了dde-switchtogreeter，用来协调二者的工作，只需要提供用户名就可以切换。

然而，虽然这样的想法是很好的，可是当初并没有人写文档，随着人员的变动，现在公司应该没有一个人是比较完整了解整个的工作流程了，用户切换的bug也就这样被留下来了。

上次修复用户切换的问题，是发现登录以后lightdm-deepin-greeter没有退出，由于不是很清楚linux的登录流程，再加上代码中有不工作的退出代码，当时就改好了退出的问题，这样就引入了第二个问题，而这个问题，就导致了两天三个人在一直查找问题所在。

这次的问题是发现一直切换greeter，会导致Xorg一直在开新的display，这就很奇怪了，正常来说是不会一直创建才对。

最开始以为是dde-switchtogreeter的问题，毕竟切换是它在做，dde-switchtogreeter是单文件的c代码，代码没有任何的说明，真的是为切换而生，我从main函数开始自己走了好几遍的流程，一边看着d-feet的数据来验证，然而只发现了一个小问题，整个代码是没啥问题的。

最后在后端大佬的帮助下，知道了display-manager会自己退掉greeter，不需要自己退，然后我就想起来了以前改的地方，赶紧把退出代码删掉，重新编译，问题得到了解决。

如果我知道display-manager的工作流程，也许这个问题就不会拖两天了。