---
title: 把博客转移到coding
p: hexo page move to coding
date: 2018-11-09 20:17:32
tags:
categories:
---

上周末折腾黑果子的时候，不小心被果子坑爹的磁盘管理坑了，整个home被直接改成HFS+了，本来是打算分配一个空闲分区出来的，当我新建分区以后，从空间分区开始到home，分区全部都变成HFS+了，但是… 空闲分区新建失败，提示我磁盘空间不足，我就重启进deepin打算直接新建一个算了，然后就GG几率了。在windows下看到home已经成果子的文件系统了，然后我用arch的安装盘看了一下，已经无法重新挂载了(成功GG)，然后数据就都没了。

还好我的数据在公司还有一份，私钥也都在，经过一星期的努力复制，大部分数据都恢复了，不过topbar的新功能代码是彻底没了，周五晚上太自信了，没有提交到gayhub上(猛叹气)。

我们现在正在尝试把日常工作转向github的project和看板，每天早上开一下晨会，简单分配一下任务，开完会以后我会把自己的任务写在谷歌日历和task上，然后安排一下任务的先后顺序，我准备把自己的一些做法写到博客上，但是home已经不在了，所以我要先恢复我的博客，刚好国内有人说我博客访问的很慢，我打算国内解析到coding，国外解析到github。

<!-- more -->

首先，创建新的博客目录，用来拉取旧的数据。

```
mkdir blog && cd blog
```

初始化git目录。

```
git init
```

添加远程仓库。

```
git remote add origin 你的博客git地址
```

取回origin的backup分支，和本地master合并。因为hexo-git-backup插件只支持master，但是coding只支持master部署page服务，所以需要使用其他分支。

```
git pull origin backup:master
```

拉取了代码以后，我们需要做点其他设置，首先设置上游分支。

```
git branch --set-upstream-to=origin/backup master
```

设置git的默认push策略，可以参考[thekaiway](http://thekaiway.com/2013/07/30/config-your-git-push-strategy/)的文章。

```
git config push.default upstream
```

然后添加coding的git地址。

```
git remote add coding 你的git地址
```

之后就正常使用了，通过npm安装hexo，再安装需要的插件，最后完成了在一台新电脑上恢复hexo博客。
