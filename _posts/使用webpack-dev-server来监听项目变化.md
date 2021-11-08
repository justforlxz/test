---
title: 使用webpack-dev-server来监听项目变化
date: 2019-11-25 17:54:36
tags: Vue
categories: Vue
---

webpack的出现方便了前端开发者，使开发和部署分成了两部分，开发者可以正常根据工程化的要求进行开发，部署时通过webpack实现代码的裁剪和优化。

本次就介绍一个webpack的功能 `webpack-dev-server`

将webpack与提供实时重载的开发服务器一起使用。这仅应用于开发。
它在后台使用了webpack-dev-middleware，它提供了对Webpack资产的快速内存访问。

<!-- more -->

webpack-dev-server提供了一个小型的express的http服务器，这个http服务器和client使用了websocket通讯协议，原始文件作出改动后，webpack-dev-server会实时的编译，但是最后的编译的文件并没有输出到目标文件夹。

**注意：启动webpack-dev-server后，在目标文件夹中是看不到编译后的文件的,编译后的文件都保存到了内存当中来加速访问。**

## 启用webpack-dev-server

```
npm install -D webpack-dev-server
```

在webpack.config.js中添加devServer对象：

```
var path = require('path');

module.exports = {
  //...
  devServer: {
    contentBase: path.join(__dirname, 'dist'),
    compress: true, // 开启压缩
    port: 9000 // 指定运行的端口
  }
};
```

然后通过`npx webpack-dev-server`启动，终端上会输出一些信息，一般我们会增加一些参数来使输出更加好看:

```
webpack-dev-server --devtool eval-source-map --progress --colors --hot --inline
```

上面的命令增加一个开发工具 `eval-source-map`，开启了progress进度显示，开启了colors颜色，hot热更新和inline更新模式。上面的参数也可以添加到devServer的属性中。

终端输出的内容如下:

```
10% building 1/1 modules 0 activeℹ ｢wds｣: Project is running at http://localhost:9000/
ℹ ｢wds｣: webpack output is served from /dist/
ℹ ｢wds｣: Content not from webpack is served from /home/justforlxz/Projects/VueBlog/dist
ℹ ｢wdm｣: Hash: ff9005d9f6ffafd11cd4
Version: webpack 4.41.0
Time: 2938ms
Built at: 11/25/2019 6:03:50 PM
  Asset      Size  Chunks             Chunk Names
main.js  2.09 MiB    main  [emitted]  main
Entrypoint main = main.js
[0] multi (webpack)-dev-server/client?http://localhost:9000 (webpack)/hot/dev-server.js ./src/main.ts 52 bytes {main} [built]
```

我们就可以通过localhost:9000来访问我们的应用了。

需要注意的是，由于我们经常把内容输出到dist目录，但是webpack运行是在项目目录的，访问webpack生成在dist目录的main.js时，需要写上相对于webpack的目录，例如dist/main.js。否则会找不到文件。

如果遇到问题，导航到 /webpack-dev-server 路径，可以显示出文件的服务位置。 例如，http://localhost:9000/webpack-dev-server。

## 配置webpack

webpack-dev-server支持在服务内部调用中间件对数据进行处理。

### devServer.before

`function (app, server)`

在服务内部的所有其他中间件之前， 提供执行自定义中间件的功能。 这可以用来配置自定义处理程序，例如：

```
module.exports = {
  //...
  devServer: {
    before: function(app, server) {
      app.get('/some/path', function(req, res) {
        res.json({ custom: 'response' });
      });
    }
  }
};
```

### devServer.after

同devServer.before，在服务内部的所有中间件之后，提供执行自定义中间件的功能。

### devServer.allowedHosts

允许添加白名单服务，允许一些开发服务器访问。

```
module.exports = {
  //...
  devServer: {
    allowedHosts: [
      'host.com',
      'subdomain.host.com',
      'subdomain2.host.com',
      'host2.com'
    ]
  }
};
```

模仿 django 的 ALLOWED_HOSTS，以 . 开头的值可以用作子域通配符。.host.com 将会匹配 host.com, www.host.com 和 host.com 的任何其他子域名。

```
module.exports = {
  //...
  devServer: {
    // 这实现了与第一个示例相同的效果，
    // 如果新的子域名需要访问 dev server，
    // 则无需更新您的配置
    allowedHosts: [
      '.host.com',
      'host2.com'
    ]
  }
};
```

### devServer.clientLogLevel

`string: 'none' | 'info' | 'error' | 'warning'`

当使用内联模式(inline mode)时，会在开发工具(DevTools)的控制台(console)显示消息，例如：在重新加载之前，在一个错误之前，或者 模块热替换(Hot Module Replacement) 启用时。默认值是 info。

devServer.clientLogLevel 可能会显得很繁琐，你可以通过将其设置为 'none' 来关闭 log。

```
module.exports = {
  //...
  devServer: {
    clientLogLevel: 'none'
  }
};
```

### devServer.color  - 只用于命令行工具(CLI)

只在终端下启用，启用/禁用控制台的彩色输出。

```
webpack-dev-server --color
```

### devServer.compress

`boolean`

一切服务都开启gzip压缩。

```
module.exports = {
  //...
  devServer: {
    compress: true
  }
};
```

### devServer.contentBase

`boolean: false string [string] number`

告诉服务器从哪个目录中提供内容。只有在你想要提供静态文件时才需要。devServer.publicPath 将用于确定应该从哪里提供 bundle，并且此选项优先。

默认情况下，将使用当前工作目录作为提供内容的目录。将其设置为 false 以禁用 contentBase。

```
module.exports = {
  //...
  devServer: {
    contentBase: path.join(__dirname, 'public')
  }
};
```

也可以从多个目录提供内容：

```
module.exports = {
  //...
  devServer: {
    contentBase: [path.join(__dirname, 'public'), path.join(__dirname, 'assets')]
  }
};
```

### devServer.disableHostCheck

`boolean`

设置为 true 时，此选项绕过主机检查。不建议这样做，因为不检查主机的应用程序容易受到 DNS 重新连接攻击。

```
module.exports = {
  //...
  devServer: {
    disableHostCheck: true
  }
};
```

### devServer.filename

`string`

在 lazy mode(惰性模式) 中，此选项可减少编译。 默认在 lazy mode(惰性模式)，每个请求结果都会产生全新的编译。使用 filename，可以只在某个文件被请求时编译。

如果 output.filename 设置为 'bundle.js' ，devServer.filename 用法如下：

```
module.exports = {
  //...
  output: {
    filename: 'bundle.js'
  },
  devServer: {
    lazy: true,
    filename: 'bundle.js'
  }
};
```

现在只有在请求了bundle.js时，才会去编译bundle。

## 总结

webpack的功能确实很强大，可以针对代码进行各种操作，最终生成出可以适应各种场景的代码，使开发和部署彻底分离开来，开发者可以更加专注项目。
