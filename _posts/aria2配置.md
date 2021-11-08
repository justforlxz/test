---
layout:     post
title:      "aria2配置"
subtitle:   "以前总是忘了aria2的配置"
date:       2016-05-25 23:56
author:     "kirigaya"
header-img: "home-bg.jpg"
tags:
    - 教程
---


安装好aria2，然后执行一下内容

    $ sudo nano /etc/systemd/user/aria2.service  

    [Unit]
    Description=Aria2 Service
    After=network.target

    [Service]
    ExecStart=/usr/bin/aria2c --enable-rpc --rpc-listen-all=true   --rpc-secret=secret  --rpc-allow-origin-all  --conf-path=/home/用户名字/.config/aria2/aria2.conf  --input-file /home/用户名字/.config/aria2/session.lock --disk-cache=100M --max-overall-download-limit=0  --split=10 --max-connection-per-server=10 --max-concurrent-downloads=4   --dir=/home/用户名字/Downloads/
    [Install]
    WantedBy=default.target

<!--more-->

# 注意
以上内容需要把用户名字更改成自己的

在用户目录新建三个文件

    touch ~/.config/aria2.conf  
    touch ~/.config/aria2.session  
    touch ~/.config/session.lock

~/.config/aria2.conf 里面需要填写以下内容，其他两个文件保持空。

    dir=下载目录【需要自行修改】
    enable-rpc=true

启动服务  

    systemctl --user enable aria2
    systemctl --user start aria

浏览器打开：[http://yaaw.qiniudn.com/](http://yaaw.qiniudn.com/ "aria2")
将服务器地址改成

    http://token:secret@127.0.0.1:6800/jsonrpc

然后应该页面的右上角就显示网速了。
