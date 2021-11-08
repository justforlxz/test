---
title: 使用swapfile来休眠
s: hibernate for swapfile
date: 2018-12-12 11:01:55
tags: Linux
categories: Linux
---

最近deepin要添加休眠功能，但是之前测试的通过swapfile来休眠失败了，所以对正在使用swap分区的用户提供休眠功能。但是昨天我在askubuntu上看到有人发了在ubuntu下通过swapfile休眠的方案，今天试了一下，效果良好，觉得可以考虑给deepin也加上这样的功能。

<!-- more -->

原文链接: [Hibernate and resume from a swap file](https://askubuntu.com/questions/6769/hibernate-and-resume-from-a-swap-file)

具体步骤是通过uswsusp这个包来做的，uswsusp是一组用户空间工具，用于Linux系统上的休眠(挂起到磁盘)和挂起(挂起到RAM或待机)。详细内容可以在ArchWiki上参考。[点这里](https://wiki.archlinux.org/index.php/Uswsusp)


先创建一个和内存同等大小的swapfile，为了确保休眠成功，不能小于内存的容量。

```
sudo fallocate -l 16g /swapfile # 我的机子是16G，具体自己修改
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
echo '/swapfile swap swap defaults 0 0' | sudo tee -a /etc/fstab
```

安装用户空间软休眠(Userspace Software Suspend)包

```
sudo apt install uswsusp
```

创建需要的配置文件，只需要创建文件即可。

```
sudo touch /etc/uswsusp.conf
sudo dpkg-reconfigure -pmedium uswsusp
```

这时候终端会提醒是否继续，选择Yes，然后会要求你创建一个密码，设置一个密码继续即可。

此时就可以测试一下功能了，不过我是跳过这个步骤了(比较喜欢作死)。

修改systemd的hibernate服务，使用uswsusp来代替systemd的功能。

```
sudo systemctl edit systemd-hibernate.service
```

写入以下内容:

```
[Service]
ExecStart=
ExecStart=/usr/sbin/s2disk
ExecStartPost=/bin/run-parts -a post /lib/systemd/system-sleep
```

这时候可以使用systemd的命令来测试的，我表示工作的非常正常。

```
systemctl hibernate
```

执行以后可以看到屏幕上会打印当前保存的进度，然后设备就关机了，此时再开机，等待一会儿以后就看到了背景是我漂亮老婆的锁屏，解锁以后看到工作区还是执行命令前的，一切ok。


参考以下内容:
> https://askubuntu.com/questions/6769/hibernate-and-resume-from-a-swap-file

> https://wiki.archlinux.org/index.php/Uswsusp
