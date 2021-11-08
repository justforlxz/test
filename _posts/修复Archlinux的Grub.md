---
title: 修复Archlinux的Grub
date: 2017-12-18 09:44:55
tags: Linux
---

又双叒叕不知道怎么搞的，就把arch的grub给弄坏了，但是在重新安装grub的时候，提示了这么一个错误:

```
Could not prepare Boot variable: No space left on device
```

诶不对啊，boot分区还有800M呢，怎么这么快没空间了，根目录也有52G呢，于是谷歌了一把，找到了解决办法.

```
sudo rm /sys/firmware/efi/efivars/dump-*
```

>新式 efivarfs (EFI VARiable FileSystem) 接口 (CONFIG_EFIVAR_FS) - 由位于 /sys/firmware/efi/efivars 的 efivarfs 内核模块挂载使用 - 老式 sysfs-efivars 接口的替代品，不限制变量数据大小，支持 UEFI Secure Boot 变量并被上游推荐使用。在3.8版的内核中引入，新的 efivarfs 模块在3.10版内核中从旧的 efivars 内核模块中分离。

删掉dump文件，就可以正常安装了【有点迷，不应该啊。

参考资料 : [Unified_Extensible_Firmware_Interface](https://wiki.archlinux.org/index.php/Unified_Extensible_Firmware_Interface_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87))

