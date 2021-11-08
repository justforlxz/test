---
title: vue-router路由复用后页面没有刷新
date: 2020-06-01 10:35:41
tags: [ 'Web' ]
categories:
---

vue-router提供了页面路由功能，可以用来构建单页面应用，在使用vue-router的动态路由匹配的时候，遇到了url变化了，但是页面却没有任何动静的问题，记录一下。

<!-- more -->

动态路由匹配url变化了，但是组件没有变化是因为vue进行了组件复用，因为两个路由都渲染同个组件，比起销毁再创建，复用则显得更加高效。不过，这也意味着组件的生命周期钩子不会再被调用。所以我们需要手动进行数据刷新。

我们可以简单的使用watch来监听当前的路由变化，从而实现数据刷新。

```ts
import { watch } from "vue";
import router from "./router";

export default {
    setup() {
        // vue2
        // watch: {
        //     $route(to, from) {
        //         // 对路由变化作出响应...
        //     }
        // }

        // vue3
        watch(router.currentRoute, () => {
            console.log("路由发生了变化");
        });
    }
};
```

也可以使用2.2中新加的beforeRouteUpdate路由守卫：

```ts
export default {
    setup() {

    },
    beforeRouteUpdate((to, from, next) => {
        // 不要在这里调用next
        // 通过to来判断是否重载数据
        console.log("路由发生了变化");
    }),
}
```

以上就是vue3中使用vue-router-next来处理动态路由变化导致页面不刷新的方法。
