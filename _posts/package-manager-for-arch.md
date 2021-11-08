---
title: 如何给Arch打一个包
date: 2021-03-04 11:23:42
tags: [ Linux ]
categories:
---

Arch 使用的是 pacman 包管理器，包格式是 tar.zst。Arch 提供了一些工具用于创建 tar.zst 包，首先需要安装 base-devel 包和 devtools 包。

```shell
pacman -S base-devel devtools
```

Arch 的打包流程是这样的，要先写一个 PKGBUILD 文件，这个文件描述了构建一个包所需的全部信息，如从哪里下载源码，依赖有哪些，构建的版本是多少，如何进行构建等。

# PKGBUILD

一个基本的 PKGBUILD 格式如下：

```text
# Maintainer: justforlxz <justforlxz@gmail.com>

pkgname=dtkcore-git
pkgver=5.4.0.r1.ge7e7a99
pkgrel=1
pkgdesc='DTK core modules'
arch=('x86_64')
url="https://github.com/linuxdeepin/dtkcore"
license=('LGPL3')
depends=('dconf' 'deepin-desktop-base-git' 'python' 'gsettings-qt' 'lshw')
makedepends=('git' 'qt5-tools' 'gtest')
conflicts=('dtkcore')
provides=('dtkcore')
groups=('deepin-git')
source=("$pkgname::git://github.com/linuxdeepin/dtkcore.git")
sha512sums=('SKIP')

pkgver() {
    cd $pkgname
    git describe --long --tags | sed 's/\([^-]*-g\)/r\1/;s/-/./g'
}

build() {
  cd $pkgname
  qmake-qt5 PREFIX=/usr DTK_VERSION=$pkgver LIB_INSTALL_DIR=/usr/lib
  make
}

package() {
  cd $pkgname
  make INSTALL_ROOT="$pkgdir" install
}
```

以 dtkcore 为例，整个配置文件十分清晰明了，一看就知道构建工具是如何一步步制作出一个包的。

将配置文件保存到目录里，以 `PKGBUILD` 命名，然后执行 `makepkg`。

makepkg 是用于执行 PKGBUILD 文件的工具，它根据文件中的描述，进行包的构建。

# 如何提交一个包到 aur

当你希望自己打的包可以被别人使用的时候，一般都会请求上传到官方仓库，但是进入官方仓库的流程异常繁琐，而且对贡献者的要求会比较高。Arch 提供了一个用户仓库，供用户自由分享配置文件，只需要下载配置文件，在本机打包就可以了。

根据 aur 的 [贡献指南](https://wiki.archlinux.org/index.php/AUR_submission_guidelines)，我们可以很轻松的上传自己的配置文件，并借用yay等aur工具自动下载和构建软件包。

> 用户可以通过 Arch User Repository 分享 PKGBUILD。AUR 中不包含任何二进制包，仅包含用户上传的 PKGBUILD，供其他用户下载使用。所有软件包都是非官方的，使用风险自担。

提交aur之前，需要先了解一下 aur 的一些要求，[Rules of submission](https://wiki.archlinux.org/index.php/AUR_submission_guidelines#Rules_of_submission) 提供了一些规则，只要我们按照规则来，就可以上传和维护我们自己的包。

首先需要去 [aur官网](https://aur.archlinux.org) 注册一个账号，并上传自己的 ssh key 我们通过克隆一个新的仓库来作为上传的方式。

```shell
git clone ssh://aur@aur.archlinux.org/<pkgname>.git
```

以dtkcore为例，克隆的时候把pkgname改成dtkcore。

```shell
git clone ssh://aur@aur.archlinux.org/dtkcore.git
```

**如果是第一次创建，会提示你克隆的一个空仓库。**

把 PKGBUILD 文件复制到克隆的目录，然后执行 `makepkg --printsrcinfo > .SRCINFO` 创建一个软件包信息，将 `.SRCINFO` 和 `PKGBUILD` 文件都添加到 git 中，然后提交 commit 信息，推送到服务器。

> 注意： 如果您忘记在首次提交中包含 .SRCINFO，您可以使用 rebasing with --root 或是 filtering the tree 使得 AUR 接受您的第一次推送

> 提示： 为了保持工作目录和提交尽可能的干净，可以创建 gitignore 文件来排除所有文件，然后再按需添加文件。

参考资料：
> https://wiki.archlinux.org/index.php/PKGBUILD
