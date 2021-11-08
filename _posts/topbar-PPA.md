---
title: topbar PPA
date: 2017-07-20 13:37:51
tags:
---

自己搭了一个仓库，提供deepin-topbar及相关依赖的包。

I created a repository,provide deepin-topbar and dependencies.

<!-- more -->

也许需要安装dirmngr

maybe you need install dirmngr

# 追加内容到/etc/apt/sources.list

Append content to /etc/apt/sources.list

```
deb [arch=amd64] https://packages.mkacg.com panda main 

```

# 导入key 

import key

```
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 3BBF73EE77F2FB2A
```

# 刷新列表，进行安装

then, refresh list and install

``` 
sudo apt update && sudo apt install deepin-topbar 
```