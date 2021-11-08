---
title: Dock插件开发<等待填坑>
date: 2018-05-23 05:22:12
tags: Linux
---

从零构建 dde-dock 的插件
本教程将展示一个简单的 dde-dock 插件的开发过程，插件开发者可跟随此步骤为 dde-dock 创造出更多具有丰富功能的插件。

<!-- more -->

在本教程中，将创建一个可以实时显示用户家目录(~/)使用情况的小工具。

插件的工作原理
dde-dock 插件本质是一个按 Qt 插件标准所开发的共享库文件(so)。通过 dde-dock 预定的规范与提供的接口，共同完成 dde-dock 的功能扩展。

准备环境
插件的开发环境可以是任意的，只要是符合 Qt 插件规范及 dde-dock 插件规范的共享库文件，都可以被当作 dde-dock 插件载入。下面以 Qt + qmake 为例进行说明：

安装依赖
以 Deepin 15.5.1 环境为基础，至少先安装如下的包：

- dde-dock-dev
- qt5-qmake
- qtbase5-dev-tools
- libqt5core5a
- libqt5widgets5
- pkg-config

基本的项目结构
创建必需的项目目录与文件
插件名称叫做home_monitor，所以创建以下的目录结构：

```
home_monitor
├── home_monitor.json
├── homemonitorplugin.cpp
├── homemonitorplugin.h
└── home_monitor.pro
 ```

