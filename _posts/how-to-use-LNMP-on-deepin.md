---
title: 如何在Deepin上使用LNMP
date: 2019-02-21 10:11:15
tags:
  - LNMP
  - Linux
  - Deepin
  - Web
categories:
  - Linux
author: Lorem Ipsum
url: http://generator.lorem-ipsum.info
---

为了节省读者的时间，我先简述一下阅读这篇文章需要了解的知识。

这篇文章将基于Docker来构建nginx、php和mysql来搭建LNMP环境，和其他教程有所不同的是，需要有一定的Docker基础。

<!-- more -->

Docker是一个不错的工具，使我们不需要虚拟机那样的庞然大物就可以轻松的隔离运行的程序，这要感谢Linux的资源分离机制，避免启动一个虚拟机造成了大量资源浪费。

首先需要在Deepin上安装Docker，添加Docker的deb仓库，并安装docker-ce。

创建文件
```
sudo nano /etc/apt/sources.list.d/docker.list
```
写入
```
deb [arch=amd64] https://download.docker.com/linux/debian jessie edge
```

刷新一下仓库就可以安装了。

```
sudo apt update && sudo apt install docker-ce docker-compose
```

安装完成后重启一下系统，准备工作就算完成了一半了。

在家目录创建一个Projects目录，当做我们LNMP的工作目录，创建一个名叫*docker-compose.yaml*的文件，这是docker-compose的配置文件，我们通过docker-compose这个工具来管理我们的Docker容器。

所有的镜像均采用最新版本，nginx(1.15.8)，php(7.3.2)，mysql(8.0.15)，如有需要，自行选择不同版本的镜像。

注意PHP7已经不支持mysql扩展，使用内置的MySQLnd。

写入以下配置文件：

```
version: '3'

services:
  nginx:
    # 设置容器名字
    container_name: "nginx"
    # 采用最新的nginx
    image: nginx:latest
    # 绑定80端口
    ports:
        - "80:80"
    # 添加php容器的依赖
    depends_on:
        - "php"
    # 绑定数据目录
    volumes:
        - "./volumes/nginx/conf.d:/etc/nginx/conf.d"
        - "./volumes/html:/usr/share/nginx/html"
    restart: always

  php:
    # 设置容器名字
    container_name: "php"
    # 采用最新的php
    image: php:fpm
    # 绑定端口
    ports:
        - "9000:9000"
    # 绑定数据目录
    volumes:
        - "./volumes/html:/var/www/html"
    restart: always

  mysql:
    # 设置容器名字
    container_name: "mysql"
    # 采用最新的mysql
    image: mysql:latest
    # 绑定端口
    ports:
        - "3306:3306"
    # 设置环境变量
    environment:
        - MYSQL_ROOT_PASSWORD=(自己设置密码)
    # 绑定数据目录
    volumes:
        - "./volumes/mysql:/var/lib/mysql"
    restart: always
```

创建nginx的配置文件，编辑 *./volumes/nginx/conf.d/nginx.conf* ：

```
server {
    listen       80;
    server_name  localhost;
    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm index.php;
    }
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
    location ~ \.php$ {
        fastcgi_pass   php:9000;
        fastcgi_index  index.php;
        fastcgi_param  SCRIPT_FILENAME /var/www/html/$fastcgi_script_name;
        include        fastcgi_params;
    }
}
```

创建php测试文件，编辑 *./volumes/html/index.php* :

```
<?php
phpinfo();
?>
```

启动docker，第一次需要拉取一下镜像:

```
docker-compose up --build -d
```

等全部结束以后，就可以访问localhost看到php的信息了。

通过Docker的方法来使用LNMP，不污染宿主机环境，不会再因为各种依赖问题而搞坏系统，这恰恰是新手容易犯的错误，使用Docker，方便你我。
