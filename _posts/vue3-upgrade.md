---
title: vue3升级遇到的坑
date: 2020-05-31 21:11:43
tags: [ 'Web' ]
categories:
---

最近一直忙工作上的事，对提高自身能力的事有点落下了，趁着今天把之前思考的一些问题都给解决了，也顺手给自己的VueBlog把vue和webpack都升级到最新的beta版本，然后遇到了很多坑，今天就把坑都记录一下，免的以后忘了。

VueBlog目前使用的是webpack5 + vue3 + vue-router-next + typescript构建，目的在于替换当前的hexo站点，同样也是一个静态博客生成器，不过和hexo的定位不同，我使用的是单页面设计，而不是给每个页面生成对应的html文件，所以对SEO不友好，以后再想办法吧。

<!-- more -->

## 升级Vue3

首先使用`vue add vue-next`来升级vue到beta版本，执行以后vue会对代码进行一次转换，将旧版本的一些api转换为新版本。

### App
例如将main.ts中创建App对象的代码转换为新的，在vue2中，我们通过new Vue()来创建app对象，并调用$mount函数挂在元素。

```
import App from "./App.vue";
const app = new Vue(App);
app.$mount('#app')
```

在vue3中，主体思想都尽量通过函数来进行了，因为可以通过函数的参数和返回值进行类型推导。在vue3中，创建app对象通过createApp函数来进行，再通过mount函数挂载dom元素。

```
import App from "./App.vue";
const app = createApp(APp);
app.mount("#app");
```

### Vur Router

如果使用的有vue-router之类的插件，使用方法也有一些变化，router也需要通过对应的create函数创建。首先需要先升级vue-router，vue-router的下一个版本叫[vue-router-next](https://github.com/vuejs/vue-router-next)。在vue-router中，创建router对象的函数从VueRouter函数改为createRouter。

```
const router = new VueRouter({
    ...
});

export default router;
```

在vue3中则需要使用新的函数返回：

```
const router = createRouter({
    ...
});

export default router;
```

内容也改了一部分，可以访问[github](https://github.com/vuejs/vue-router-next)仓库来看文档。

### composition API

composition api是vue3提出的核心功能，其核心目的是通过将分散在各处的数据都整合到一个setup函数中进行初始化，并依赖vue的响应式数据改变来完成功能实现。

在RFC中就有composition api的动机。

> #### 更好的逻辑复用与代码组织
> 1. 随着功能的增长，复杂组件的代码变得越来越难以阅读和理解。这种情况在开发人员阅读他人编写的代码时尤为常见。根本原因是 Vue 现有的 API 迫使我们通过选项组织代码，但是有的时候通过逻辑关系组织代码更有意义。
> 2. 目前缺少一种简洁且低成本的机制来提取和重用多个组件之间的逻辑。
> #### 更好的类型推导
> 另一个来自大型项目开发者的常见需求是更好的 TypeScript 支持。Vue 当前的 API 在集成 TypeScript 时遇到了不小的麻烦，其主要原因是 Vue 依靠一个简单的 this 上下文来暴露 property，我们现在使用 this 的方式是比较微妙的。（比如 methods 选项下的函数的 this 是指向组件实例的，而不是这个 methods 对象）。
>
>换句话说，Vue 现有的 API 在设计之初没有照顾到类型推导，这使适配 TypeScript 变得复杂。
> 相比较过后，本 RFC 中提出的方案更多地利用了天然对类型友好的普通变量与函数。用该提案中的 API 撰写的代码会完美享用类型推导，并且也不用做太多额外的类型标注。
>
> 这也同样意味着你写出的 JavaScript 代码几乎就是 TypeScript 的代码。即使是非 TypeScript 开发者也会因此得到更好的 IDE 类型支持而获益。
>
> [composition api](https://composition-api.vuejs.org/zh/api.html) 文档官方
>
> [vue3 rfc](https://composition-api.vuejs.org/zh) rfc网站

setup函数用起来确实舒服，所有有用的东西都可以放在一块，代码整理也方便，不像以前一样需要分散到各种hook和计算属性、data函数中。但是也有我用起来不舒服的地方，基本类型和对象都需要使用ref函数和reactive函数进行包装，有的时候用起来就各种麻烦，需要多注意一些。不过这个问题倒不是什么大问题，和写c++的时候所有的对象用智能指针包裹一层一样，用多了就习惯了。

这是一个vue2的经典例子，通过data函数和计算属性来返回不同的数据。

```
export default new Vue({
    data: function() {
        return {
            message: "hello"
        }
    },
    computed: {
        reversedMessage: function () {
            return this.message.split('').reverse().join('');
        }
    }
});
```

在vue3中就可以全部集中到setup函数，并且一并返回，模板可以直接使用。

```
import { ref, computed } from "vue";

export default {
    setup() {
        const message = ref("hello");
        const reversedMessage = computed(() => {
            return message.value.split('').reverse().join('');
        });

        return { message, reversedMessage };
    }
};
```

可以看出使用setup函数可以将模板所需的内容一块返回，结构更为清晰，vue2的模式也是可以的，只不过侧重点不一样，vue2的目的是一种动作的数据应该被放在一块，而vue3的setup函数则是将数据处理都放在一块，这样对数据的的整理比较方便和集中。

### Props

props是在组件上注册的自定义属性，当一个值传递给props的时候，它就会成为那个组件的一个property。

```
<hello v-bind:message="message" />
```

hello组件可以通过定义props函数来接收自定义属性。

```
export default new Vue({
    props: {
        message: String
    }
});
```

这样就可以在helle.vue中使用message这个属性，不过需要注意的是，hello组件不要修改传递进来的message,否则会破坏数据的流向。

在vue3中使用会更加方便，因为类型推导更加方便。

```
interface Props {
    message?: String
}

export default {
    props: {
        message: {
            type: String,
            require: false,
            default: "",
        }
    },
    setup(props: Props) {
        const reversedMessage = computed(() => {
            if (props.message === undefined) {
                return String;
            }

            const innerMessage = props.message;
            return innerMessage.split('').reverse().join('');
        });

        return { reversedMessage };
    }
};
```

在vue3和typescript中使用props需要有一些注意的地方，首先Props里需要设置值可能为空，否则setup函数的签名将无法匹配。其次访问props数据需要开启setup函数的props参数，还有一个context参数，可以访问上下文的内容。
