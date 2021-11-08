---
title: 解决IntelliJ IDEA界面瞎眼
date: 2017-12-25 14:22:25
tags: Linux
---

今天在逛深度论坛的时候，无意间看到了有个回复，是处理IEDA字体很挫的，试了一下，效果非常棒。

我之前也试了些网上的办法，都没有解决，字体挫的根本看不了，被逼无奈跑到windows下写MOD了。

<!--more-->

[原文链接](https://bbs.deepin.org/forum.php?mod=redirect&goto=findpost&ptid=150634&pid=418410&fromuid=13250)

在/etc/profile.d/新建一个文件，用来设置java的环境变量:

```
sudo vim /etc/profile.d/z999__java_options.sh
```

```
#!/bin/bash
opts=" -Dswing.aatext=true  -Dawt.useSystemAAFontSettings=lcd -Djava.net.useSystemProxies=true "
export _JAVA_OPTIONS="`echo ${_JAVA_OPTIONS} |sed -Ee 's/-Dawt.useSystemAAFontSettings=\w+//ig'` $opts"
```

然后注销再登录，就可以看到效果了。

其实这个解决办法在arch的wiki上有，只不过似乎是我写错了吧，反正是没生效，按照这种方法是可以的，就这么用吧。非常感觉@ihipop。