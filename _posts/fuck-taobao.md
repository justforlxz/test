---
title: 解决用了xposed后淘宝闪退
s: fuck_taobao
date: 2019-01-23 10:27:52
tags:
categories:
---

反正都用xposed了，肯定也有root权限。
删除/data/data/com.taobao.taobao/files/bundleBaseline/里的文件，然后设置该目录为500。
