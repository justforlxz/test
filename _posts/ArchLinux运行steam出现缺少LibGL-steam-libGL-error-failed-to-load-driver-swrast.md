---
title: 'ArchLinux运行steam出现缺少LibGL--steam libGL error: failed to load driver: swrast'
subtitle:   "arch库太新，steam库太旧导致的问题。"
date: 2016-07-15 07:18:53
author:     "kirigaya"
header-img: "home-bg.jpg"
tags:
    - 实验室
---
其实arch的wiki已经提到了，而且这个问题是比较常见的，只需要删除steam的库就行。

[wiki原文链接](https://wiki.archlinux.org/index.php/Steam/Troubleshooting#Deleting_the_runtime_libraries)

<!--more-->

删除运行库
运行此命令，删除在issues上已知的运行库问题:

    find ~/.steam/root/ \( -name "libgcc_s.so*" -o -name "libstdc++.so*" -o -name "libxcb.so*" -o -name "libgpg-error.so*" \) -print -delete


如果上面的命令不工作，则再次运行上面的命令，然后运行此命令。

    find ~/.local/share/Steam/ \( -name "libgcc_s.so*" -o -name "libstdc++.so*" -o -name "libxcb.so*" -o -name "libgpg-error.so*" \) -print -delete
