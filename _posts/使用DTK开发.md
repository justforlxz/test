---
title: 使用DTK开发
date: 2018-01-12 11:05:26
tags: Linux DTK
---


**在阅读本篇文章之前，你需要掌握基本的Qt/C++开发知识。**

<!-- more -->

> **注意：本篇文章基于Deepin平台，其他平台请自行补充依赖关系。**

先安装DTK的依赖关系。

```
sudo apt install libdtkwidget2 libdtkcore2
```

新建Qt项目，编辑pro文件，添加项目依赖。

```
CONFIG += c++14 link_pkgconfig
PKGCONFIG += dtkwidget
```

DTK目前有两个组件，一个是提供库功能的core，一个是提供控件的widget。

修改main.cpp,删除QApplication的相关内容，改为DApplication。

> 注意： 使用DTK的组件，需要使用DTK的宏,根据使用的文件来选择对应的宏。

```
DWIDGET_USE_NAMESPACE
DCORE_USE_NAMESPACE
```

DTK使用了deepin自己的qt插件，需要在DApplication前调用。

```
    DApplication::loadDXcbPlugin();
    DApplication app(argc, argv);
```

DApplication中提供了很多方法来设置程序的各种信息，具体请看头文件的定义。

主窗口由DMainWindow提供，新建类，然后添加DMainWindow的头文件和DTKWIDGET的宏。

```
#include <DMainWindow>

DWIDGET_USE_NAMESPACE
```

然后修改继承关系，改为继承DMainWindow。DMainWindow提供了一些我们封装的方法。目前为止，该程序的界面已经符合Deepin程序的风格了，我们封装了一些其他控件，使其样式符合我们的风格，如果要在其他Qt程序中使用，也是同样的步骤，载入Qt插件，添加对应的头文件和DTK的宏。


