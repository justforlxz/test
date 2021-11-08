---
title: 使用github actions自动部署hexo文章到html仓库
date: 2019-12-09 13:19:11
tags:
categories:
---

请先允许我大喊一声：微软牛逼！

本文没有啥含金量，就是简单的说一下如何部署github-actions来自动生成hexo的public，并且再推送到html仓库的。

我的博客仓库一共分为两个，blog仓库是私有的，需要通过我的私钥才能访问，html仓库是公开的，hexo生成的静态内容会被上传到这里。

首先在package.json中添加一些命令，方便我们一键编译和提交:

```
  "scripts": {
    "build": "hexo clean && hexo g",
    "deploy": "yarn run build && hexo d",
    "backup": "hexo b",
  }
```

因为CI环境需要提交代码到仓库，所以申请一个个人用的token，访问[https://github.com/settings/tokens](https://github.com/settings/tokens)创建一个新的，勾选上`repo`，生成完token以后，修改一下`_config.yml`中对deploy仓库的url，格式固定为`https://x-access-token:你的token@github.com/你的名字/仓库名.git`,例如我这里是`https://x-access-token:xxxxxxxxxx@github.com/justforlxz/html.git`。

然后新家一个github actions，选择nodejs环境，我们只需要修改最后一个步骤，执行我们自己的命令即可。

- 设置git的用户名和邮箱地址
- npm install -g yarn
- yarn run deploy

如果你还有一些其他步骤，可以自行扩展，比如我就有主题相关的操作，具体的内容如下:

```
    - name: npm install, build, and deploy
      run: |
        git config --global user.email "justforlxz@gmail.com"
        git config --global user.name "justforlxz"
        git submodule update --init
        cd themes/next
        git checkout dev
        cd ../../
        npm install -g yarn
        yarn
        yarn run deploy
```

然后就可以愉快的自动部署了。
