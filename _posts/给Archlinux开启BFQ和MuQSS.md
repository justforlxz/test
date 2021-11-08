---
title: 给Archlinux开启BFQ和MuQSS
date: 2019-10-24 13:19:21
tags: Linux
categories: Linux
---

最近在Arch上更新系统的时候，总是遇到图形完全卡住的情况，今天上午突然想起来自己曾经设置了使用noop的IO调度，猜测是因为这个。然后本着不折腾不舒服的原则，打算使用ck内核上MuQSS的进程调度和BFQ的IO调度。

<!-- more -->

ck内核并没有在arch的仓库，但是aur有linux-ck的包，安装一下就可以了。

```
yay -S linux-ck linux-ck-headers
```

编译需要一些时间，在我的破本子i7-8550U编译了一顿过桥米线的时间，然后成功使用了ck内核。

### 开启MuQSS

ck内核默认使用的就是MuQSS调度，并不需要修改什么，开机即可。

### 开启BFQ

开启BFQ需要一些手动设置。分为两步：

1. 修改grub，给内核提供新的参数
2. 使用udev开启动态调整

**修改grub**

编辑`/etc/default/grub`中`GRUB_CMDLINE_LINUX_DEFAULT`，增加一行内容:
```
GRUB_CMDLINE_LINUX_DEFAULT="quiet scsi_mod.use_blk_mq=1"
```

然后更新grub配置文件:

```
sudo grub-mkconfig -o /boot/grub/grub.cfg
```

**创建udev规则**

创建并编辑`/etc/udev/rules.d/60-scheduler.rules`

```
# set deadline scheduler for non-rotating disks
ACTION=="add|change", KERNEL=="sd[a-z]", TEST!="queue/rotational", ATTR{queue/scheduler}="deadline"
ACTION=="add|change", KERNEL=="sd[a-z]", ATTR{queue/rotational}=="0", ATTR{queue/scheduler}="bfq"

# set cfq scheduler for rotating disks
ACTION=="add|change", KERNEL=="sd[a-z]", ATTR{queue/rotational}=="1", ATTR{queue/scheduler}="bfq"
ACTION=="add|change", KERNEL=="nvme[0-9]n1", ATTR{queue/rotational}=="0", ATTR{queue/scheduler}="bfq"
```

上面的配置是给固态硬盘使用deadline，给机械盘使用bfq，给nvme盘bfq。

本着电脑只有ssd，所以天不怕地不怕的原则，我选择全部使用bfq。

然后重启电脑，查看所有硬盘的调度器：

```
# justforlxz @ archlinux in ~ [13:29:04]
$ cat /sys/block/*/queue/scheduler
mq-deadline kyber [bfq] none
mq-deadline kyber [bfq] none
```

通过dmesg查看MuQSS是否开启：

```
$ sudo dmesg | grep -i scheduler
Alias tip: _ dmesg | grep -i scheduler
[    0.295872] rcu: RCU calculated value of scheduler-enlistment delay is 10 jiffies.
[    1.223982] io scheduler mq-deadline registered
[    1.223984] io scheduler kyber registered
[    1.224038] io scheduler bfq registered
[    1.586191] MuQSS CPU scheduler v0.193 by Con Kolivas.
```

## 总结

MuQSS是BFS(脑残调度器)的进化版，主要是改进了BFS的O(n)复杂度，BFS适用于桌面环境用户，可以提供较好的进程切换和延迟。
BFQ是针对硬盘的IO调度，它通过预先分配一定的IO吞吐量来合理安排每个进程的IO操作。我需要用几天来感受一下MuQSS和CFQ的好处。
