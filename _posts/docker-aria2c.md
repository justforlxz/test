---
layout: post
title: docker-aria2c
subtitle: docker-aria2c
author: kirigaya
header-img: home-bg.jpg
tags:
  - 实验室
date: 2016-05-31 22:43:54
---

该项目是将aria2c封装进docker并提供服务。


    docker pull kirigayakazushin/docker-aria2c

<!--more-->

下载好镜像，然后保存一份运行

    docker run -p 6800:6800 --name docker-aria2c -d \
    -v {下载目录的绝对路径}:/aria2/downloads \
    imashiro/docker-aria2c

打开浏览器，访问[http://yaaw.qiniudn.com/](http://yaaw.qiniudn.com/)
输入

    http://token:secret@127.0.0.1:6800/jsonrpc

注意，暂时还无法处理文件的所有权，目前下载好的文件归属root。
