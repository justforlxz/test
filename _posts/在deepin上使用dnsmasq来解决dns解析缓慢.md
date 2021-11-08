---
title: 在deepin上使用dnsmasq来解决dns解析缓慢
date: 2017-08-11 14:07:26
tags:
---

其实这个问题影响并不是很大，只是稍微的增加一点点访问速度，缓存这东西有利有弊。

<!-- more -->

在写完这篇文章以后，我就不用dnsmasq了，现在用的是github上的[Pcap_DNSProxy](https://github.com/chengr28/Pcap_DNSProxy)。用来防止dns污染的。

根据[https://stackoverflow.com/questions/11020027/dns-caching-in-linux](https://stackoverflow.com/questions/11020027/dns-caching-in-linux) 中回答者提供的信息来看，linux发行版是不提供dns解析缓存的，上面提到的nscd也不在deepin的预装列表中，所以我们只能自己动手丰衣足食了。

首先安装口碑较好的dnsmasq，来为我们提供dns缓存。

```
sudo apt install dnsmasq
```

如果是deepin最新的2015.4.1版本中安装，安装结束会提醒一个错误，这个错误的解决办法来自[https://stackoverflow.com/questions/16939306/i-cant-restart-my-dnsmasq-service-so-my-fog-server-wont-work](https://stackoverflow.com/questions/16939306/i-cant-restart-my-dnsmasq-service-so-my-fog-server-wont-work).
这个错误似乎是因为/usr/share/dns/root.ds文件更新后结构导致的错误。

编辑/etc/init.d/dnsmasq，并找到

```
ROOT_DS="/usr/share/dns/root.ds"

if [ -f $ROOT_DS ]; then
   DNSMASQ_OPTS="$DNSMASQ_OPTS `sed -e s/". IN DS "/--trust-anchor=.,/ -e s/" "/,/g $ROOT_DS | tr '\n' ' '`" 
fi
```

修改为

```
ROOT_DS="/usr/share/dns/root.ds"

if [ -f $ROOT_DS ]; then
   DNSMASQ_OPTS="$DNSMASQ_OPTS `sed -e s/".*IN[[:space:]]DS[[:space:]]"/--trust-anchor=.,/ -e s/"[[:space:]]"/,/g $ROOT_DS | tr '\n' ' '`" 
fi
```

当错误解决以后，我们手动重启一下dnsmasq的systemd服务。

```
sudo systemctl restart dnsmasq
```

deepin的/etc/resolv.conf来自/etc/NetworkManager/resolv.conf.是一个软连接。我采取的行为是删除这个文件，重新创建。

```
sudo rm /etc/resolv.conf

sudo vim /etc/resolv.conf
```

然后写入本地地址当做dns地址。

```
nameserver 127.0.0.1
```

dnsmasq是一个本地的dns和dhcp服务器，当我们在上面成功启动dnsmasq以后，个人系统中就已经在提供dns服务了，所以本机使用回环地址来表明dns服务器就是本机，所有的dns查询都会发送到本机的dnsmasq中。

如果需要额外添加dns服务器，做法来自[https://wiki.archlinux.org/index.php/Dnsmasq#More_than_three_nameservers](https://wiki.archlinux.org/index.php/Dnsmasq#More_than_three_nameservers).

创建一个 /etc/resolv.dnsmasq.conf，写入其他dns服务器的地址。

```
nameserver 114.114.114.114
```

然后编辑 /etc/dnsmasq.conf,找到resolv-file字段，改为

```
resolv-file=/etc/resolv.dnsmasq.conf
```

然后重启dnsmasq。

验证的话通过dig命令。

```
dig blog.mkacg.com
```

通过执行两次来判断，Query time在第二次查询是为0 msec。