---
title: 添加Vue动画
date: 2019-12-08 08:03:40
tags: Vue
categories: Vue
---

以前一直搞不懂动画是怎么做的，它怎么这么神奇，写了一点看不懂的代码，就实现了非常丰富的效果，现在做了三年Qt开发，接触到了Qt的动画类，明白了动画是怎么一会儿事，现在来看当初的css动画代码，也明白了它是如何工作的了。本文会介绍一下Vue提供的组件过渡动画模块——transitions。

<!-- more -->

## 概述

Vue在插入、更新和移除DOM元素时，提供了多种不同方式的应用过渡效果。包含以下工具：

- 在css过渡和动画中自动应用class
- 可以配合第三方动画css类，例如Animae.css
- 提供钩子函数来使JS操作DOM元素
- 可以配合使用第三方JavaScript动画库，例如Velocity.js

## 单元素/组件过渡

Vue提供了 `transitions` 的封装组件，在下面的情况中，可以给任意元素或组件添加进入和离开的过渡效果。

- 条件渲染 (使用 `v-show`)
- 按需渲染 (使用 `v-if`)
- 动态节点
- 组件根元素

这是一个基本的例子：

```
<div id="app">
    <button v-on:click="show = !show">
    Toggle
    </button>
    <transitions name="fade">
        <p v-if="show"> hello! </p>
    <transitions>
<div>
```

```
new Vue({
    el: "#app",
    data: {
        show: false
    }
});
```

在head中添加style：

```
.fade-enter-active,
.fade-leave-active {
  transition: opacity .5s;
}

.fade-enter,
.fade-leave-to {
  opacity: 0;
}
```

这里有三点需要注意一下，需要动画的元素需要使用transitions节包裹起来，transitions需要一个name，css中需要使用固定的拼写来应用动画，入场动画和离场动画的状态是一致的，所以写在了一组里。

当插入和删除包含在 `transitions` 组件中的元素时，Vue会做以下的事情：

- 自动嗅探组件是否应用了css的过渡或动画，如果有，则在恰当的实际添加/删除css类名。
- 如果 `transitions` 组件提供了钩子函数，Vue会在恰当的时机调用钩子函数。
- 如果没有找到css过渡和动画，也没有找到钩子函数，则DOM的操作(插入和删除)在下一帧中立即执行。(注意是指浏览器的逐帧动画，而不是Vue的nextTick机制)

### 过渡的类名

Vue的过渡动画一共有6个状态：

1. `v-enter`: 定义进入过渡的开始状态，在元素被插入之前生效，待元素插入以后被移除。
2. `v-enter-active`: 定义进入过渡生效时的状态，在整个进入过渡的阶段中应用，在元素插入之前生效，在过渡/动画完成后被移除。这个类可以定义过渡时间、延迟和动画曲线。
3. `v-enter-to`: **在2.1.8版本及以上** 定义进入过渡的结束状态，在元素被插入的下一帧生效(与此同时 `v-enter` 被移除)，在过渡/动画完成后移除。
4. `v-leave`: 定义离开过渡的开始状态，在离开过渡被触发时立即生效，下一帧被移除。
5. `v-leave-active`: 定义离开过渡生效时的状态，在整个离开过渡的阶段中应用，在离开过渡触发时立即生效，在过渡/动画完成后立即被移除。这个类可以定义离开过渡的过程时间、延迟和动画曲线。
6. `v-leave-to`: **在2.1。8版本及以上** 定义离开过渡的结束状态，在离开过渡被触发之后的下一帧被移除(与此同时`v-leave`也被删除)
，在过渡/动画完成之后移除。

![transitions](https://cn.vuejs.org/images/transition.png)

可以看出一共两组动画，进入和离开的active。并且分别有两个状态，enter和enter-to，这6个状态控制了入场动画和离场动画。(吐槽一下Qt的动画系统，定义一个QAnimation只能做半场动画，想做到Vue这样的要定义两组，或者反向播放)

对于那些正在过渡中切换的类名来说，如果使用了没有`name`属性的`transition`，Vue会使用v-当做默认前缀。为了避免多组动画冲突，我个人建议每一个`transition`组件都提供name属性。

### JavaScript钩子函数

`transition`也提供了钩子函数，使我们可以通过JavaScript来控制DOM元素，一共也包含了8个函数：

1. beforeEnter
2. enter
3. afterEnter
4. enterCancelled
5. beforeLeave
6. leave
7. afterLeave
8. leaveCancelled

和css上要求的命名保持一致，只是增加了两个取消的接口，当动画被取消的时候被调用。

这些钩子函数可以结合CSS `transition/animations` 使用，也可以单独使用。

> 当只使用JavaScript过渡的时候，必须在 **enter** 和 **leave** 显式调用`done()`进行回调，否则他们将被同步调用，过渡会立即完成。

> 推荐对于仅使用JavaScript过渡的元素添加`v-bind:css="false"`，Vue会跳过CSS的检测，这也可以避免过渡过程中css的影响。

## 列表元素的过渡

以上我分享的都是单元素/组件的过渡，那么问题来了，列表这种通过v-for创建的元素该如何增加过渡效果呢？

Vue提供了`<transition-group>`组件，在深入了解之前，需要先介绍一下这个组件的一些特点：

- 不同于`<transition>`，`<transition-group>`会创建一个真实的DOM元素，默认是<span>，可以通过tag属性切换为其他元素。
- 过渡模式不再可用，因为我们不再相互切换特有的元素
- 内部元素总是需要提供唯一的key值来进行区分
- CSS过渡将会应用在内部的元素中，而不是这个组/容器本身

### 列表的进入/离开过渡

```
<div id="app">
<button v-on:click="add">add</button>
<button v-on:click="remove">remove</button>
<transition-group name="group" tag="ul">
<li v-for="item in items" v-bind:key="item">
  {{ item }}
</li>
</transition-group>
</div>
```

```
new Vue({
	el: "#app",
	data: {
			items: [1,2,3]
	  },
	  methods: {
	  	add() {
	  		this.items.push(0)
	  	}
	  }
})
```

```
.group-enter,
.group-leave-to {
  opacity: 0;
  transform: translateY(10px)
}

.group-enter-active,
.group-leave-active {
  transition: all 1s;
}
```

代码在[这里，点击访问](https://jsfiddle.net/justforlxz/9denwmor/20/)，只实现了添加元素的过渡效果。

希望本文可以帮助你理解Vue是如何处理过渡动画，本文是基于官网的知识和demo所编写的，本文只写了一部分我觉得需要掌握的基本功能，Vue的transition组件还有很多功能等待你的挖掘，[点击前往Vue官网文档](https://cn.vuejs.org/v2/guide/transitions.html)
