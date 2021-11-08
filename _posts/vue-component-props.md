---
title: Vue父子组件传值 —— props & $emit
date: 2019-12-08 21:08:00
tags: Vue
categories: Vue
---

Vue的父子组件传值比较有意思，父组件通过属性绑定，把自身的值和子组件的一个属性绑定起来，子组件通过props属性接收，该属性类型为数组，是Vue对象中比较少有的类型，data、computer、methods等方法都是对象的形式，props则是数组的形式。父组件通过v-on来监听子组件发出的事件来接收子组件的调用。在这里我是理解成子组件发送信号来通知上层，毕竟调用的是this.$emit来做到的。

<!-- more -->

我们假设子组件名为<hello>，我们通过v-bind来绑定一个值给它。

```
<template>
    <div id="#app">
        // 通过v-bind绑定父子组件的属性
        <hello v-bind:messageFromParent="message"/>
    </div>
<template>

<script lang="ts">
import Vue from 'vue';
import Hello from './Hello.vue';
export default Vue.extend({
    data: {
        message: "this message from parent"
    },
    components: {
        "hello": Hello
    }
});
</script>
```

子组件hello.vue通过props属性接收，内容是这样的：

```
<template>
    <div>
        {{ messageFromParent }}
    </div>
</template>

<script lang="ts">
import Vue from 'vue';
export default Vue.extend({
    // 通过props数组接收
    props: [ "messageFromParent" ]
});
</script>
```

这里有个需要注意的地方，父组件给子组件的数据是单向的，虽然子组件也可以修改父组件传入的数据，但是会产生一个错误，并打印在终端里。

那么我们怎么才能修改父组件的值呢？答案是`this.$emit`。

我们给子组件绑定上v-on，来监听子组件的事件。

```
<template>
    <div id="#app">
        // 通过v-bind绑定父子组件的属性，通过v-on监听子组件的属性变化
        <hello v-bind:messageFromParent="message" v-on:changeParentData="changeData"/>
    </div>
<template>

<script lang="ts">
import Vue from 'vue';
import Hello from './Hello.vue';
export default Vue.extend({
    data: {
        message: "this message from parent"
    },
    components: {
        "hello": Hello
    },
    methods: {
        changeData: function(data: string) {
            message = data;
        }
    }
});
</script>
```

子组件只需要发送出修改即可：

```
<template>
    <div>
        <button v-on:click="change">修改父组件数据</button>
        {{ messageFromParent }}
    </div>
</template>

<script lang="ts">
import Vue from 'vue';
export default Vue.extend({
    // 通过props数组接收
    props: [ "messageFromParent" ],
    methods: {
        change: function() {
            // 调用this.$emit方法第一个参数是事件的名称，后面全部都是参数。
            // this.$emit方法其实是自定义了一个事件，通过这种方式来完成子组件向父组件传递消息。
            this.$emit("changeParentData", "change data by child");
        }
    }
});
</script>
```

以上就是Vue父子组件传值的一种常用方法，适用于相邻组件的，如果隔代了，那么这种方式就不好用了，中间路过的组件都需要转发这个事件，处理这种情况就需要使用`provide/ inject`了，不过那就是另一篇文章啦。
