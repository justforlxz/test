---
title: samba
date: 2021-02-27 10:42:53
tags: [ Linux ]
categories:
---
samba 是一个linux下开源的SMB/CIFS协议的免费软件，最初 SMB 协议是 Microsoft 开发的网络通讯协议，后来微软又把 SMB 改名为 CIFS（Common Internet File System），即公共 Internet 文件系统，并且加入了许多新的功能，这样一来，使得Samba具有了更强大的功能。

samba目前最大的作用是文件共享，服务器只需要提供账号和密码，就可以由 samba 客户端访问，并挂载到本地系统上。

我在家用了一个 macmini 当做家庭 nas，并且使用 archlinux 当做服务器系统，我在上面部署了 samba 服务，seafile 服务，nextcloud 服务，plex 服务，arai2c 下载机。

除了 samba 服务，其他服务都是运行在 docker 中，所以 samba 服务需要我手动配置。

# 安装

archlinux 安装 samba 服务很简单，只需要使用 pacman 安装 samba 包即可。

```shell
pacman -S samba
```

安装完成以后我们就可以创建用户和配置共享目录了。

# 配置

由于 samba 不提供配置文件，所以我们要从网上下载基本的 [配置文件](https://man.archlinux.org/man/smb.conf.5)。

## 创建用户
我们需要先在系统创建对应的用户，以我的用户名 lxz 为例。

```shell
useradd -m -g users -G wheel lxz
```

## 禁用用户登录

由于该用户只是为了权限相关的操作，所以我并没有计划使用该账户，如果需要在系统中使用该账户，则不需要进行接下来的禁用操作。

**禁用本地登录**

```shell
usermod --shell /usr/bin/nologin --lock lxz
```

**禁用ssh登录**

修改 `/etc/ssh/sshd_config` ，修改 `AllowUsers`。

## 创建 samba 用户

我们需要区分的是，尽管 samba 的用户和 Linux 系统本身的用户是同一个，但是密码则是分开存放的，我们需要使用 samba 提供的命令修改用户的密码。

```shell
smbpasswd -a lxz
```

修改密码则不使用 `-a` 参数：

```shell
smbpasswd lxz
```

# 测试

到此，我们已经完成了基本的 samba 配置，配置文件里默认开启了用户的家目录访问，所以我们使用 smb 客户端进行登录的时候，可以直接访问家目录。

## 开启服务

```shell
systemctl enable smb
systemctl start smb
```

使用 smbclient 可以验证本机服务。

```shell
smbclient -L localhost -U%
```

```shell
[root@LXZ-MacMini ~]# smbclient -L localhost -U%

	Sharename       Type      Comment
	---------       ----      -------
	timemachine     Disk      macos time machine backup
	IPC$            IPC       IPC Service (Samba Server)
SMB1 disabled -- no workgroup available
```

# 添加其他的共享目录

除了用户家目录，我们还可以共享特定的目录。

例如我创建一个 data 目录，允许任何人只读访问。

修改 `/etc/samba/smb.conf` 文件，在末尾处添加配置。

```shell
[Guests]
  comment = Allow all users to read
  path = /data
  public = yes
  only guest = yes
  writable = no
  printable = no
```

然后重启 samba 服务，使用 smbclient 验证。

```shell
[root@LXZ-MacMini ~]# systemctl restart smb
[root@LXZ-MacMini ~]# smbclient -L localhost -U%

	Sharename       Type      Comment
	---------       ----      -------
	timemachine     Disk      macos time machine backup
	Guests          Disk      Allow all users to read
	IPC$            IPC       IPC Service (Samba Server)
SMB1 disabled -- no workgroup available
```
