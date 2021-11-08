---
title: webhook
date: 2017-07-14 02:52:20
tags: linux
---

blog现在是用hexo，放在自己的[code](https://code.mkacg.com)网站上。

code是用无闻大大的gogs搭建的，跑在台式机的docker中，本机跑了很多docker服务，有hexo，有aria2c，有gogs，还有个webserver caddy。

caddy这东西还是基友 [不爱写博客的mioto](https://mioto.me/)推荐给我的，之前我一直是用nginx的，那配置文件太复杂了，根本玩不来。

写一篇文章，会先提交到code，然后触发webhook，caddy会拉取code中的文章，由于是静态的，所以不需要处理其他的，只需要拉取最新的就可以了。

caddy的配置

```
blog.mkacg.com {
    gzip
    redir 301 {
    if {>X-Forwarded-Proto} is http
        /  https://{host}{uri}
    }
    tls kirigaya@mkacg.com
    root /srv/www/blog
    git code.mkacg.com/kirigayakazushin/blog {
        path /srv/www/blog
        branch gh-pages
        hook /_webhook xxx
        hook_type gogs
        then git pull
    }
}

```

gogs上只需要创建一个webhook，地址填写成caddy中的hook地址，加密填写hook后的xxx即可，加密自己设置。

然后就可以提交了。

提交会触发push操作，gogs会根据设置的webhook中的规则，执行和push相关的webhook，webhook会向指定的url发送POST操作，发送的内容中包含了相关信息，caddy会根据相关信息，来处理webhook，执行你规定的操作。