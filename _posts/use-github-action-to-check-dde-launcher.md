---
title: use github action to check dde-launcher
date: 2020-07-27 18:14:21
tags: [ "Linux" ]
categories: [ "技术" ]
---

本来打算7月份给dde添加github action验证，但是被各种事情耽误了，然后发现麒麟居然抢在我前面部署了全套的github action，这不能忍，赶紧把dde的github action也提上日程。并且打算听[肥肥猫](https://github.com/felixonmars)大佬的话，在aur给dde弄一套commit构建包，这样就可以在arch上使用比testing仓库更testing的dde了！

<!-- more -->

github actions是github官方出的持续集成功能，以前大家在github上的项目都使用的第三方的Travis CI或者自建jenkins构建，但是github被微软收购以后，微软为了表现出给社区和用户的诚意，将大量github的付费功能免费公开给开发者使用，希望能将github打造成开发者中心，于是在2019年微软推出了免费的github actions，每个项目都可以免费使用官方提供的持续集成和持续部署功能，这对第三方业务无疑是个巨大的打击，虽然Travis CI和jenkins等方式仍然有一定的市场，但是对于中小项目的开源项目，使用官方提供的功能无疑是方便的。

github actions的配置十分简单，只需要几个简单的步骤就可以实现构建、执行和测试代码。并且可以使用Linux、Windows和MacOS环境，机器性能也十分强劲，编译速度非常的快。

这是给dde-launcher的一份基础配置，需要将配置文件放在.github/workflows/目录下，以build.yaml文件名保存。

```yaml
name: CI Build

on:
  push:
    branches:
      - uos

  pull_request:
    branches:
      - uos

jobs:
  archlinux:
    name: Archlinux Build Check
    runs-on: ubuntu-latest
    container: docker.io/library/archlinux:latest
    steps:
      - name: Checkout branch
        uses: actions/checkout@v2
      - name: Refresh pacman repository
        run: pacman -Syy
      - name: Install build dependencies
        run: pacman -S --noconfirm base-devel cmake ninja qt5-tools deepin-qt-dbus-factory dtkwidget
      - name: CMake & Make
        run: |
          mkdir build
          cd build
          cmake ../ -G Ninja
          ninja
```

介绍一下配置文件吧，name是设置ci的名字，github允许有多个ci存在，可以做不同的事情，例如部署三个ci，一个做语法检查，一个做静态检查，一个做编译检查。name就是用来在界面上显示的。on是设置ci对哪些事件感兴趣，在这里我设置了push和pull_request，当发生push和pull request时，这个ci就会被启动，执行接下来的jobs的内容。jobs里是可以设置多个任务的，同样name字段也是用来展示本次动作的名称。runs-on是设置该job工作的环境，ubuntu-latest是linux环境，container是指使用哪个docker容器，github actions是可以使用docker的，也可以将自己的ci配置共享给其他人使用。run就是执行命令了，在配置文件中我手动运行了刷新仓库和编译项目所需的命令。job的steps可以理解成shell中一次动作的执行，uses是使用其他人封装好的命令，run则是执行本地命令。

可以看出github actions的配置是十分简单的，并且构建速度也非常的快，并且构建环境是使用的arch linux环境，为什么要选择arch作为ci的基础构建环境呢，原因当然不是因为和肥肥猫有py交易，arch上的dde更新速度很快，并且很多用户都使用arch+dde的方式使用linux，deepin自己维护的发行版因为基础仓库更新较慢，不适合一些用户，所以为了能让dde被更多的人接受和使用，在arch上及时更新dde是十分有必要的。所以才选择actions的环境为arch linux。