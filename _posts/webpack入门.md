---
title: webpack入门
date: 2019-10-14 15:34:52
tags: 'Web'
categories: [
    'Web',
    'webpack'
]
---

现在前端开发不像以前一样，只需要写html、css和javascript文件就可以了。现代前端开发讲究工程化。

**什么是工程化？**

工程化即系统化、模块化、规范化的一个过程。

**为什么要工程化？**

工程化是让开发、测试和维护都变得更加可靠和提高效率的方式。

1. 制定规范
2. 版本管理
3. 单元测试
4. 自动化

通过制定流程的方式，规范了开发和测试的流程，让工作有章可循，方便团队协作。

<!--more -->

最初的网页开发，是写好几份的javascript代码和css文件，手动在html中引入的。这样不适合多人协作开发，一旦开发人员多了，不可避免的会造成文件和命名冲突。
为了避免这些事情的发生，javascript增加了模块的概念。

有好的事情出现，就会有坏的事情发生。

过多的模块导致js文件下载很慢，而且有冗余，为了避免这件事情影响用户体验，webpack横空出世了。

webpack是一个现代javascript的静态模块打包器。它会递归的构建出依赖图，并根据依赖图来输出应用程序需要的每个模块，然后将所有这些模块打包成一个或多个 bundle。

webpack有四个核心概念:

- 入口(entry)
- 输出(output)
- loader
- 插件

入口决定了webpack要从哪个文件开始构建依赖图。

看一个简单的例子:

```
module.exports = {
    entry: './src/index.js'
}
```

output则决定了webpack会在哪里输出生成的bundles，以及如何命名这些bundles。输出目录默认为 `./dist/` 。

```
const path = require('path');

module.exports = {
    entry: './src/index.js',
    output: {
        path: path.resolve(__dirname, 'dist'),
        filename: 'bundle.js'
    }
}
```

loader可以让webpack打包非javascript文件，loader可以将所有类型的文件转换为webpack可以识别的有效模块，然后利用webpack的打包能力，对他们进行处理。

```
const path = require('path');

module.exports = {
    entry: './src/index.js',
    output: {
        path: path.resolve(__dirname, 'dist'),
        filename: 'bundle.js'
    },
    module: {
        rules: {
            test: /\.css$/,
            use: 'css-loader'
        }
    }
}
```

rules中的意思是，当require()/impot中被解析为.css的路径时，先使用css-loader转换一下。

我们可以开发新的loader去加载不同的文件，最终都通过webpack来打包到一起。

loader用于转换某些类型的模块，插件则工作的更加广泛。插件的范围包括，从打包优化和压缩，一直到重新定义环境中的变量。插件接口功能极其强大，可以用来处理各种各样的任务。

想要使用一个插件，你只需要 require() 它，然后把它添加到 plugins 数组中。多数插件可以通过选项(option)自定义。你也可以在一个配置文件中因为不同目的而多次使用同一个插件，这时需要通过使用 new 操作符来创建它的一个实例。

```
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin'); // 通过 npm 安装
const webpack = require('webpack');

module.exports = {
    entry: './src/index.js',
    output: {
        path: path.resolve(__dirname, 'dist'),
        filename: 'bundle.js'
    },
    module: {
        rules: {
            test: /\.css$/,
            use: 'css-loader'
        }
    },
    plugins: [
        new HtmlWebpackPlugin({template: './src/index.html'})
    ]
}
```

## 总结

通过webpack，我们可以将整个项目都打包为一个文件进行分发，而且还可以进行优化。webpack的出现，将前端的开发和发布彻底的分离开，开发人员可以以各种方式进行开发，通过webpack打包以后输出部署需要的文件。
