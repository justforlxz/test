---
layout:     post
title:      "Archlinux 添加漂亮的字体"
subtitle:   "想要漂亮的字体吗"
date:       2016-04-08 16:54:26
author:     "kirigaya"
header-img: "home-bg.jpg"
tags:
    - 教程
---
该教程不能保证适用于所有人的情况。
字体也不是配置，而是补充了字体。使用的是第三方的源。

<!--more-->

打开/etc/pacman.conf文件，添加以下内容到最底部。

    [infinality-bundle]
    SigLevel = Never
    Server = http://bohoomil.com/repo/$arch
    [infinality-bundle-multilib]
    SigLevel = Never
    Server = http://bohoomil.com/repo/multilib/$arch
    [infinality-bundle-fonts]
    SigLevel = Never
    Server = http://bohoomil.com/repo/fonts

执行安装命令:

    sudo pacman -S infinality-bundle infinality-bundle-multilib ibfonts-meta-extended  #（用于64位系统）
    sudo pacman -S infinality-bundle ibfonts-meta-extended #（用于32位系统）

如果有遇到错误，可以手动添加hosts：

    188.226.219.173 bohoomil.com

会出现很多冲突，选择Y，然后安装。如果中断了，重新执行安装命令。

来自：[如何给任意一款 Linux 发行版添加漂亮的字体-桌面应用|Linux.中国-开源社区][1]

[1]: https://linux.cn/article-3019-1.html "如何给任意一款 Linux 发行版添加漂亮的字体-桌面应用|Linux.中国-开源社区"
