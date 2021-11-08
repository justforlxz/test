---
title: 重构dde-session-ui
date: 2018-03-11 11:48:01
tags: Linux
---

dde-session-ui里面包含了很多项目，是一个集合，但是其中的代码缺少合理的维护，以至于已经到了必须重构才能继续开发和维护，在支持AD域登录的时候，如果强制加上功能，代码会变得更加糟糕，所以和石博文一块重构了其中非常重要的UserWidget。

<!-- more -->

## 重构前的设计

重构前的dde-lock和lightdm-deepin-greeter是非常混乱的，处理逻辑都混杂在一块，虽然能看出有基本的结构，但是整体并未解耦。

## 重构后的设计

- 基于User类的处理
- UserWidget负责提供对用户的处理，暴露出基本的currentUser和LogindUsers。
- Lock和Greeter的Manager从UserWidget、SessionWidget中获取用户和用户的会话。
- Manager只负责控件的位置和用户的验证。
- 背景修改为Manager提供模糊的壁纸，FullBackground只提供绘制。

重构以后用了大概原代码的1/3，启动速度也快了，感觉世界充满了美好… 就是重构历程太辛苦…

本次也发现了很多自身的问题，基础并没有学好，很多地方都可以使用更好的处理方式【就是管不住这手…】

