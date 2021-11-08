---
title: 解决NVIDIA重新启动以后系统冻结
date: 2017-09-01 17:01:47
tags: linux
---

分期买了一台神舟 Z6-kp5s1，配置还不错，够用三年了，但是在linux下使用bumblebee的时候，发生了问题，折腾了好久，现在把解决方法写出来。

<!-- more -->

先说一下问题吧，正常安装bumblebee、bbswitch和nvidia驱动，重新启动系统以后，系统出现冻结，没有任何的输入输出，没有任何日志产生。问题似乎是固件错误，详情查看[讨论](https://github.com/Bumblebee-Project/Bumblebee/issues/764)和Linux的bug[讨论](https://bugzilla.kernel.org/show_bug.cgi?id=156341)。

解决方法是看的[Witiko](https://witiko.github.io/Optimus-on-Linux/)的博客，通过给内核传递参数来防止系统出现冻结。修改/etc/default/grub,在文件底部追加以下内容：

```
# Bumblebee 3.2.1 fix (see https://github.com/Bumblebee-Project/Bumblebee/issues/764)
GRUB_CMDLINE_LINUX_DEFAULT="$GRUB_CMDLINE_LINUX_DEFAULT "'acpi_osi=! acpi_osi="Windows 2009"'
```

如果不放心，请先禁用登录管理器，防止开机就出现冻结，然后尝试手动启动登录管理器。在tty登录，然后执行：

```
sudo systemctl start display-manager.service
```

如果一切正常，你将会看到图形，并且lspci -v中能看到nvidia已经被禁用，然后使用提供的测试方法进行测试，可以看到nvidia被启用，关闭测试成功，nvidia被禁用。


提供一下我关闭nvidia以后的使用和续航时间吧。亮度调节为50%，cpu设置为powersave，运行了一下程序：

- telegram
- chrome
- dde-file-manager
- vs code
- meow
- 若干ss client
- 还有一大堆乱七八糟的服务，懒得写了

从14:35开始断电测试，到17:17还有23%的电量。