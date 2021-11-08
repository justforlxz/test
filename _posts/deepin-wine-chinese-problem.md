---
title: deepin-wine中文乱码
date: 2020-09-17 16:32:12
tags: ['Linux']
categories: ['技术']
---

众所周知，使用wine来运行windows下的一些软件是linux用户的常用操作，deepin为社区贡献了好几款中国用户必备的软件，例如QQ、微信、企业微信，以此来让更多的人无痛的切换到linux来。近年来value也一直在linux上布局，先后推出了steam主机和proton,前者是基于kvm和steam大屏幕模式的系统，而proton则是wine的分支，value提供了几组补丁用来提高性能和与Windows游戏的兼容性。

Arch上有非常方便的aur仓库，有很多用户都自己提交一些软件到aur上面，就有几个维护者将deepin打包的deepin wine和对应的软件包移植到Arch上面来，已经是很多人在Arch上面运行QQ和微信的首选方案。

但是使用的过程中我遇到了以下几个问题：

- 输入框中文乱码
- 在DDE桌面使用kwin的情况下最小化会卡死
- KDE桌面环境无法使用deepin wine的程序

第二个问题比较特殊，因为dde在deepin和uos下运行的是fork版本的kwin,而Arch上运行的则是原版的kwin,一些操作代码并不具备，所以会出现一些奇葩的状况。

第三个问题则是xsettings的原因，在移植者的仓库有对应的issue讨论 [《KDE环境完全无法使用wine-tim》](https://github.com/wszqkzqk/deepin-wine-ubuntu/issues/12)

第一个问题解决起来也比较简单，是因为缺少了宋体文件，从而无法正确的渲染中文字体。而比较奇葩的是，把方块复制再粘贴，就可以正确的渲染了。

解决的方法也很简单，把windows的的字体复制过来就可以了。由于版权的问题，没办法直接提供文件，需要各位自己复制了。

> 引用资料

[https://github.com/wszqkzqk/deepin-wine-ubuntu/issues/136](https://github.com/wszqkzqk/deepin-wine-ubuntu/issues/136#issuecomment-517576733)
