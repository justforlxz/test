---
title: 在ArchLinux上开发startdde
date: 2020-08-06 16:14:26
tags: ['dde', 'go']
categories: ["Linux"]
---

dde 后端使用 go 作为主要的开发语言，使用 dbus 提供接口，主要使用 gsettings 来保存配置。 所以在进行后端开发前需要对以上内容有基本的了解，这里假定本文档的阅读者熟悉 dbus 和 gsettings，并有一定的开发经验。

<!-- more -->

## 安装依赖

虽然本项目是go语言开发的，但是我们并没有直接使用go的mod作为依赖管理方案，而是走系统包管理器的方式，所以要先安装startdde的编译依赖。

```shell
sudo pacman -Sy golang-github-linuxdeepin-go-dbus-factory golang-deepin-gir golang-deepin-lib golang-deepin-dde-api go git jq golang-golang-x-net golang-github-linuxdeepin-go-x11-client
```

这些包会被安装到系统的/usr/share/gocode目录下。还需要手动go get一个依赖到本地的GOPATH中。

```go
go get -v github.com/cryptix/wav
```

## 设置GOPATH

为了方便以后的开发，可以将GOPATH环境变量定义到~/.xprofile等文件中，或者shell的配置文件。例如我使用的zsh：

```shell
export GOPATH=$HOME/Develop/Go:/usr/share/gocode
export PATH=$HOME/Develop/Go/bin:$PATH
```

## 设置项目目录

go要求项目目录必须在GOPATH中，所以要将startdde放到GOPATH的pkg.deepin.io/dde/目录下，但是GOPATH每次进入不方便，可以采用软链的形式将startdde的目录链接到GOPATH下。

```shell
cd ~/Develop/Deepin
git clone https://github.com/linuxdeepin/startdde
mkdir -p ~/Develop/Go/src/pkg.deepin.io/dde/
ln -sf ~/Develop/Deepin/startdde ~/Develop/Go/src/pkg.deepin.io/dde/startdde
```

这样就可以在一个方便的目录进行开发了。

## vscode开发工具

我个人推荐使用vscode当作开发工具，打开vscode安装go的插件，打开startdde目录，vscode会提示安装一些go的工具，选择全部安装即可。
