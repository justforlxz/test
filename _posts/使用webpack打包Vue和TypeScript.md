---
title: 使用webpack打包Vue和TypeScript
date: 2019-10-22 15:20:08
tags: [
    Vue,
    Webpack,
    TypeScript
]
categories: Web
---

本文将会介绍如何通过Webpack将基于TypeScript的Vue项目进行打包。

<!-- more -->

## webpack基础配置

首先创建一个基本的webpack.config.js文件:

```
const path = require( 'path' );

module.exports = {
    entry: {
        index: "./src/index.ts",
    },
    output: {
        path: path.resolve( __dirname, 'dist' ),
        publicPath: '/dist/',
        filename: '[name].js'
    },
    devtool: 'inline-source-map',
    mode: 'development',
    module: {
        rules: [
        ]
    },
    resolve: {
    }
};
```

此时webpack只能将src/index.ts文件直接输出为index.js，我们需要添加typescript的loader，进行typescript的转换。

将以下代码加入rules节:

```
{
    test: /\.ts?$/,
    loader: 'ts-loader',
    exclude: /node_modules/,
},
```

通过ts-loader进行ts文件的转换，我们还需要创建typescript的一个配置文件。

## 添加typescript支持

创建tsconfig.json

```
{
    "compilerOptions": {
        "outDir": "./dist/",
        "sourceMap": true,
        "strict": true,
        "module": "commonjs",
        "moduleResolution": "node",
        "target": "es5",
        "skipLibCheck": true,
        "esModuleInterop": true,
        "experimentalDecorators": true
    },
    "include": [
        "./src/**/*"
    ]
}
```

还需要在webpack的配置中添加ts文件，在resolve节中添加:

```
extensions: [ '.ts', '.js' ],
```

我们指定ts转换出的js代码是es5的。

这个时候我们运行webpack，将会看到正常的转换输出。

```
Hash: c3a0ae2c47032de12eec
Version: webpack 4.41.0
Time: 1880ms
Built at: 10/22/2019 3:40:59 PM
   Asset      Size  Chunks             Chunk Names
index.js  11.8 KiB   index  [emitted]  index
Entrypoint index = index.js
[./src/index.ts] 269 bytes {index} [built]
```

入口文件就是index.ts了，之后我们就正常的在index.ts中写我们的代码，webpack就会查找所有的依赖，并打包输出到index.js中。

## 添加Vue单文件的支持

Vue单文件组件(SFC)规范是指在一个文件中，提供html、css和script代码，三者包含在顶级语言块 `<template>`、`<script>` 和 `<style>`
 中，还允许添加可选的自定义块。

这是一个简单的vue单文件例子:

```
<template>
  <div class="example">{{ msg }}</div>
</template>

<script>
export default {
  data () {
    return {
      msg: 'Hello world!'
    }
  }
}
</script>

<style>
.example {
  color: red;
}
</style>

<custom1>
  This could be e.g. documentation for the component.
</custom1>
```

我们通过vue-loader来解析该文件，提取每一个语言块，如有需要，会传递给其他loader进行处理，最后组装为一个ES Module。

我们在webpack的rules节中添加vue-loader:

```
{
  test: /\.vue$/,
  loader: 'vue-loader',
  options: {
    loaders: {
      // Since sass-loader (weirdly) has SCSS as its default parse mode, we map
      // the "scss" and "sass" values for the lang attribute to the right configs here.
      // other preprocessors should work out of the box, no loader config like this necessary.
      'scss': 'vue-style-loader!css-loader!sass-loader',
      'sass': 'vue-style-loader!css-loader!sass-loader?indentedSyntax',
    }
    // other vue-loader options go here
  }
},
```

## 如果vue是typescript代码？

其实这很简单，ts-loader有一个appendTsSuffixTo的功能，可以给某个文件增加.ts的后缀，从而识别这个文件为ts文件。

```
{
  test: /\.tsx?$/,
  loader: 'ts-loader',
  exclude: /node_modules/,
  options: {
    appendTsSuffixTo: [/\.vue$/],
  }
},
```

我们还需要在项目中添加一个vue-shim.d.ts来让ts正确的识别vue。

```
declare module '*.vue' {
  import Vue from 'vue'
  export default Vue
}
```

还需要在webpack的resolve节追加vue的后缀:

```
resolve: {
  extensions: [ '.tsx', '.ts', '.js' , '.vue'],
  alias: {
    'vue': 'vue/dist/vue.js'
  }
},
```

vue-loader现在需要手动处理一下插件，在webpack.config.js的头部导入vue-loader，并在plugins节创建对象。

```
const { VueLoaderPlugin } = require('vue-loader')

.......

plugins: [
  new VueLoaderPlugin()
],
```

否则将不能正确工作。

此时已经完成了webpack+vue+typescript的全部工作。

```
Hash: 320d4ed3f55f52872694
Version: webpack 4.41.0
Time: 2494ms
Built at: 10/22/2019 4:00:50 PM
      Asset       Size    Chunks             Chunk Names
  bundle.js   1.12 MiB    bundle  [emitted]  bundle
electron.js   12.2 KiB  electron  [emitted]  electron
 index.html  194 bytes            [emitted]
Entrypoint bundle = bundle.js
Entrypoint electron = electron.js
[./node_modules/css-loader/dist/cjs.js!./node_modules/vue-loader/lib/loaders/stylePostLoader.js!./node_modules/sass-loader/dist/cjs.js!./node_modules/vue-loader/lib/index.js?!./src/app.vue?vue&type=style&index=0&id=5ef48958&rel=stylesheet%2Fscss&lang=scss&scoped=true&] ./node_modules/css-loader/dist/cjs.js!./node_modules/vue-loader/lib/loaders/stylePostLoader.js!./node_modules/sass-loader/dist/cjs.js!./node_modules/vue-loader/lib??vue-loader-options!./src/app.vue?vue&type=style&index=0&id=5ef48958&rel=stylesheet%2Fscss&lang=scss&scoped=true& 542 bytes {bundle} [built]
[./node_modules/ts-loader/index.js?!./node_modules/vue-loader/lib/index.js?!./src/Components/About.vue?vue&type=script&lang=ts&] ./node_modules/ts-loader??ref--1!./node_modules/vue-loader/lib??vue-loader-options!./src/Components/About.vue?vue&type=script&lang=ts& 305 bytes {bundle} [built]
[./node_modules/vue-loader/lib/loaders/templateLoader.js?!./node_modules/vue-loader/lib/index.js?!./src/Components/About.vue?vue&type=template&id=aa9c95a6&] ./node_modules/vue-loader/lib/loaders/templateLoader.js??vue-loader-options!./node_modules/vue-loader/lib??vue-loader-options!./src/Components/About.vue?vue&type=template&id=aa9c95a6& 235 bytes {bundle} [built]
[./node_modules/vue-loader/lib/loaders/templateLoader.js?!./node_modules/vue-loader/lib/index.js?!./src/app.vue?vue&type=template&id=5ef48958&scoped=true&] ./node_modules/vue-loader/lib/loaders/templateLoader.js??vue-loader-options!./node_modules/vue-loader/lib??vue-loader-options!./src/app.vue?vue&type=template&id=5ef48958&scoped=true& 589 bytes {bundle} [built]
[./node_modules/vue-style-loader/index.js!./node_modules/css-loader/dist/cjs.js!./node_modules/vue-loader/lib/loaders/stylePostLoader.js!./node_modules/sass-loader/dist/cjs.js!./node_modules/vue-loader/lib/index.js?!./src/app.vue?vue&type=style&index=0&id=5ef48958&rel=stylesheet%2Fscss&lang=scss&scoped=true&] ./node_modules/vue-style-loader!./node_modules/css-loader/dist/cjs.js!./node_modules/vue-loader/lib/loaders/stylePostLoader.js!./node_modules/sass-loader/dist/cjs.js!./node_modules/vue-loader/lib??vue-loader-options!./src/app.vue?vue&type=style&index=0&id=5ef48958&rel=stylesheet%2Fscss&lang=scss&scoped=true& 1.64 KiB {bundle} [built]
[./src/Components/About.vue] 1.06 KiB {bundle} [built]
[./src/Components/About.vue?vue&type=script&lang=ts&] 350 bytes {bundle} [built]
[./src/Components/About.vue?vue&type=template&id=aa9c95a6&] 203 bytes {bundle} [built]
[./src/app.vue] 1.08 KiB {bundle} [built]
[./src/app.vue?vue&type=style&index=0&id=5ef48958&rel=stylesheet%2Fscss&lang=scss&scoped=true&] 716 bytes {bundle} [built]
[./src/app.vue?vue&type=template&id=5ef48958&scoped=true&] 207 bytes {bundle} [built]
[./src/entry.ts] 538 bytes {bundle} [built]
[./src/main.ts] 1.11 KiB {electron} [built]
[./src/route.ts] 1.35 KiB {bundle} [built]
```
