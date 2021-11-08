---
title: Qt basic widgets
tags: [
    Qt
]
categories: Qt
---

Qt提供了一些很基础的控件用来构建桌面风格的用户界面，Qt Widgets是传统的用户界面元素，通常在桌面环境中看到，这些小部件很好的集成到底层平台，在Windows、Linux和macOS上提供本机外观。Widgets是成熟的，具有丰富的用户界面元素，适用于大多数静态用户界面。

Qt Widgets中的重要概念：

- 应用主窗口
- 桌面集成
- 对话窗口
- 布局管理
- 模型/视图编程
- 富文本处理
- 拖放
- 国际化

## 主窗口概念

这些类提供了典型的现代主应用程序窗口所需的一切，如主窗口本身，菜单和工具栏等。

- QAction: 抽象的用户界面操作
- QActionGroup： 将操作组合在一起
- QWidgetAction： 通过接口扩展QAction，以将自定义小部件插入基于操作的容器
- QDockWidget： 可以停靠在QMainWindow中的窗口小部件，也可以作为桌面上的顶级窗口浮动。
- QMainWindow： 主要应用窗口
- QMidArea： 显示MID窗口的区域
- QMidSubWindow： QMidArea的子窗口类
- QMenu： 用于菜单栏，上下文菜单和其他弹出菜单的菜单小部件
- QMenuBar： 水平菜单栏
- QSizeGrip： 调整大小句柄以调整顶级窗口的大小
- QStatusBar： 适合呈现状态信息的水平条
- QToolBar： 包含一组控件的可移动面板


## 主窗口类

Qt提供了以下用于管理主窗口和相关用户界面组件的类：

- QMainWindow是围绕着其构建应用程序的中心类。与配套的QDockWidget和QToolBar类一起，它代表了应用程序的顶级用户界面。

- QDockWidget提供了一个小部件，可用于创建可拆卸的工具调色板或辅助窗口。Dock窗口小部件跟踪它们自己的属性，它们可以作为外部窗口移动，关闭和浮动。

- QToolBar提供了一个通用的工具栏小部件，可以容纳许多不同的的与动作相关的小部件，例如按钮、下拉菜单、组合框和旋转框。在Qt中强调统一的动作模型意味着工具栏和键盘快捷键配合的很好。

## 桌面集成

Qt程序在用户桌面环境中表现良好，但某些集成需要额外的，有时是平台特定的技术。

### 有用的类

Qt中各种类旨在帮助开发人员将应用程序集成到用户的桌面环境中。这些类使开发人员能够使用跨平台API来使用本机服务。

- QDesktopServices 访问常见桌面服务的方法
- QSystemTrayIcon 系统托盘中应用程序的图标

### 打开外部资源

尽管Qt提供了处理和显示资源的工具，例如常见的图像格式和HTML，但是有时候需要使用外部应用程序打开文件和外部资源。

QDesktopServices为用户桌面环境提供服务接口。特别是openUrl()函数用于使用适当的应用程序打开资源。该应用程序可能已有用户专门配置。

### 系统托盘图标