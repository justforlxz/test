---
title: Vuex基本使用
date: 2021-08-10 13:56:30
tags: Web
categories:
  - Web
  - Vue
  - Vuex
---

最近在写 deepincn 的首页，看着 elementary os 的首页样式挺好看的，就打算山寨一下。

本来打算用 Angular 开发的，但是看到 Vue3 已经十分稳定了，就打算重拾 Vue3，用 Vue3 来写新的首页。

为了方便以后更新数据，需要使用集中的数据管理，Vue 下比较出名和好用的就是 Vuex 了，而 Vuex 也已经适配了 Vue3，可以放心使用了。

本篇文章会见简单介绍一下 Vuex，然后讲一下基本语法，我会再开( ~水~ )一篇文章来专门山寨一个 Vuex，通过实现一个简易的 Vuex 来深入了解核心。

## 1. 简介

Vue 最重要的一个功能点就是 **数据驱动**。

所谓数据驱动，是指视图是由数据驱动生成的，我们对视图的修改，不会直接操作 DOM，而是通过修改数据。它相比我们传统的前端开发，如使用 jQuery 等前端库直接修改 DOM，大大简化了代码量。

特别是当交互复杂的时候，只关心数据的修改会让代码的逻辑变的非常清晰，因为 DOM 变成了数据的映射，我们所有的逻辑都是对数据的修改，而不用碰触 DOM，这样的代码非常利于维护。

Vue 还有一个重要的功能点是 **组件化**，每个组件就是最小的功能单位，每个组件都有自己的 `data`，`template` 和 `methods`。我们通过修改 `data` 中的数据来更新 `template` 中视图。在单个组件中我们更新视图是非常方便的，但是实际开发中，我们的项目不止一个组件，非常多的组件整合在一起的时候，维护同一个状态就会非常困难，特别是嵌入非常深的时候，传递参数就会变得非常麻烦（这里我想起来了 Deepin 的控制中心，组件里面有非常深的嵌套，最里面的组件想要获取数据，需要沿着嵌套层层传递，非常麻烦）。

为了解决这个问题，我们需要引入 Vuex 来进行状态管理，负责组件中的通信。

> Vuex 解决的问题

- 多个视图依赖于同一状态。

- 来自不同视图的行为需要变更同一状态。

## 2. 基础使用

首先先在项目中安装 Vuex，Vuex4 是为 Vue3 开发的版本，我们通过以下命令安装：

**npm**

```shell
npm install vuex@next
```

**yarn**

```shell
yarn add vuex@next
```

每一个 Vuex 应用的核心就是 store（仓库）。“store”基本上就是一个容器，它包含着你的应用中大部分的状态 (state)。Vuex 和单纯的全局对象有以下两点不同：

1. Vuex 的状态存储是响应式的。当 Vue 组件从 store 中读取状态的时候，若 store 中的状态发生变化，那么相应的组件也会相应地得到高效更新。

2. 你不能直接改变 store 中的状态。改变 store 中的状态的唯一途径就是显式地提交 (commit) mutation。这样使得我们可以方便地跟踪每一个状态的变化，从而让我们能够实现一些工具帮助我们更好地了解我们的应用。

现在已经安装好 Vuex 了，我们可以开始写一个简单的 store。创建一个 `store.ts` 文件。

```typescript
import { createStore } from 'vuex'

const store = createStore({
    state: {
        count: 0
    },
    mutations: {
        increment (state) {
            state.count++
        }
    },
});

export { store };
```

在 main.ts 中，将 store 注册到 Vue 的 app 实例中。

```typescript
import { store } from './store'

...

app.use(store);

app.mount('#app'); // 添加在挂载之前
```

以上就是一个最简单的 store 的例子了，是不是有些摸不着头脑？

store.ts 中我们导出了使用 createStore 函数返回的对象，在 store 中，我们定义了两个属性：`state` 和 `mutations`。

我们在 state 中定义状态，在 mutations 中定义操作方法，在 mutations 中定义的方法，必须通过 `store.commit()` 方法调用，而不能直接访问。

现在我们在组件中使用 store 吧。

```typescript
<template>
    <div>
        <p>{{ count }}</p>
    </div>
</template>

<script lang="ts">
import { defineComponent } from 'vue';

export default defineComponent({

    setup() {
        return { count: this.$store.state.count };
    },
    methods: {
        increment() {
            this.$store.commit('increment')
        }
    }
});
</script>
```

在组件的例子中，setup 函数是 Vue3 引入的 组合式 API（composition API），这里不详细介绍 setup 的使用，setup 的出现，我们可以将相同功能的代码放在接近的位置，只需要在函数最后返回视图(template)所需的变量即可。

在例子中，setup 向视图提供了 count 变量，这里使用了解构赋值，模板使用 count 变量时，实际上访问的还是 store 中的 count,为什么呢？

这是因为 Vuex 的状态存储是响应式的，具体请看 ~~[《Vue 响应式原理》](/)~~。

在例子中，组件提供了一个方法，用于修改 store 中的 count，更改 Vuex 的 store 中的状态的唯一方法是提交 mutation。Vuex 中的 mutation 非常类似于事件：每个 mutation 都有一个字符串的事件类型 (type)和一个回调函数 (handler)。这个回调函数就是我们实际进行状态更改的地方，并且它会接受 state 作为第一个参数。

你不能直接调用一个 mutation 处理函数。这个选项更像是事件注册：“当触发一个类型为 increment 的 mutation 时，调用此函数。”。

## 3. 核心概念

Vuex 一共有五个核心概念： State、Getter、Mutation、Action 和 Module。

### State

Vuex 使用单一状态树，用一个对象就包含了全部的应用层级状态。至此它便作为一个“唯一数据源 (SSOT)”而存在。这也意味着，每个应用将仅仅包含一个 store 实例。单一状态树让我们能够直接地定位任一特定的状态片段，在调试的过程中也能轻易地取得整个当前应用状态的快照。

在组件中，我们可以通过 `this.$store.state` 来获取注册到的 state。每当 store.state.count 变化的时候, 都会重新求取计算属性，并且触发更新相关联的 DOM。

Vuex 通过 Vue 的插件系统将 store 实例从根组件中“注入”到所有的子组件里。且子组件能通过 this.$store 访问到。

template 可以直接使用 this.$store.state.[属性] ，this 可以省略。

```typescript
<template>
  <div id="app">
    {{ this.$store.state.name }}
    {{ this.$store.state.age }}
  </div>
</template>
```

### Getter

有时候我们需要从 store 中的 state 中派生出一些状态，例如对列表进行过滤并计数：

```typescript
doneTodosCount () {
    return this.$store.state.todos.filter(todo => todo.done).length
}
```

如果有多个组件需要用到此属性，我们要么复制这个函数，或者抽取到一个共享函数然后在多处导入它——无论哪种方式都不是很理想。

Vuex 允许我们在 store 中定义“getter”（可以认为是 store 的计算属性）。

Getter 接受 state 作为其第一个参数：

```typescript
const store = createStore({
  state: {
    todos: [
      { id: 1, text: '...', done: true },
      { id: 2, text: '...', done: false }
    ]
  },
  getters: {
    doneTodos (state) => {
      return state.todos.filter(todo => todo.done)
    }
  }
})
```

Getter 会暴露为 store.getters 对象，你可以以属性的形式访问这些值：

```typescript
store.getters.doneTodos // -> [{ id: 1, text: '...', done: true }]
```

### Mutation

在上面已经说过了，更改 Vuex 的 store 中的状态的唯一方法是提交 mutation。

```typescript
const store = createStore({
  state: {
    count: 1
  },
  mutations: {
    increment (state) {
      // 变更状态
      state.count++
    }
  }
})
```

需要使用 commit 方法来提交一个调用：

```typescript
store.commit('increment')
```

也可以提交多个参数，commit 函数会展开所有参数：

```typescript
// ...
mutations: {
  increment (state, n) {
    state.count += n
  }
}
```

```typescript
store.commit('increment', 10)
```

### Action

Action 类似于 mutation，不同在于：

1. Action 提交的是 mutation，而不是直接变更状态。
2. Action 可以包含任意异步操作。

```typescript
const store = createStore({
  state: {
    count: 0
  },
  mutations: {
    increment (state) {
      state.count++
    }
  },
  actions: {
    increment (context) {
      context.commit('increment')
    }
  }
})
```

Action 函数接受一个与 store 实例具有相同方法和属性的 context 对象，因此你可以调用 context.commit 提交一个 mutation，或者通过 context.state 和 context.getters 来获取 state 和 getters。

Action 通过 store.dispatch 方法触发：

```typescript
store.dispatch('increment')
```

乍一眼看上去感觉多此一举，我们直接分发 mutation 岂不更方便？实际上并非如此，还记得 mutation 必须同步执行这个限制么？Action 就不受约束！我们可以在 action 内部执行异步操作：

```typescript
actions: {
  incrementAsync ({ commit }) {
    setTimeout(() => {
      commit('increment')
    }, 1000)
  }
}
```

### Module

由于使用单一状态树，应用的所有状态会集中到一个比较大的对象。当应用变得非常复杂时，store 对象就有可能变得相当臃肿。

为了解决以上问题，Vuex 允许我们将 store 分割成模块（module）。每个模块拥有自己的 state、mutation、action、getter、甚至是嵌套子模块——从上至下进行同样方式的分割：

```typescript
const moduleA = {
  state: () => ({ ... }),
  mutations: { ... },
  actions: { ... },
  getters: { ... }
}

const moduleB = {
  state: () => ({ ... }),
  mutations: { ... },
  actions: { ... }
}

const store = createStore({
  modules: {
    a: moduleA,
    b: moduleB
  }
})

store.state.a // -> moduleA 的状态
store.state.b // -> moduleB 的状态
```

对于模块内部的 mutation 和 getter，接收的第一个参数是模块的局部状态对象。

```typescript
const moduleA = {
  state: () => ({
    count: 0
  }),
  mutations: {
    increment (state) {
      // 这里的 `state` 对象是模块的局部状态
      state.count++
    }
  },

  getters: {
    doubleCount (state) {
      return state.count * 2
    }
  }
}
```

同样，对于模块内部的 action，局部状态通过 context.state 暴露出来，根节点状态则为 context.rootState：

```typescript
const moduleA = {
  // ...
  actions: {
    incrementIfOddOnRootSum ({ state, commit, rootState }) {
      if ((state.count + rootState.count) % 2 === 1) {
        commit('increment')
      }
    }
  }
}
```

默认情况下，模块内部的 action 和 mutation 仍然是注册在全局命名空间的——这样使得多个模块能够对同一个 action 或 mutation 作出响应。Getter 同样也默认注册在全局命名空间，但是目前这并非出于功能上的目的（仅仅是维持现状来避免非兼容性变更）。必须注意，不要在不同的、无命名空间的模块中定义两个不同的 getter 从而导致错误。

如果希望你的模块具有更高的封装度和复用性，你可以通过添加 namespaced: true 的方式使其成为带命名空间的模块。当模块被注册后，它的所有 getter、action 及 mutation 都会自动根据模块注册的路径调整命名。例如：

```typescript
const store = createStore({
  modules: {
    account: {
      namespaced: true,

      // 模块内容（module assets）
      state: () => ({ ... }), // 模块内的状态已经是嵌套的了，使用 `namespaced` 属性不会对其产生影响
      getters: {
        isAdmin () { ... } // -> getters['account/isAdmin']
      },
      actions: {
        login () { ... } // -> dispatch('account/login')
      },
      mutations: {
        login () { ... } // -> commit('account/login')
      },

      // 嵌套模块
      modules: {
        // 继承父模块的命名空间
        myPage: {
          state: () => ({ ... }),
          getters: {
            profile () { ... } // -> getters['account/profile']
          }
        },

        // 进一步嵌套命名空间
        posts: {
          namespaced: true,

          state: () => ({ ... }),
          getters: {
            popular () { ... } // -> getters['account/posts/popular']
          }
        }
      }
    }
  }
})
```

启用了命名空间的 getter 和 action 会收到局部化的 getter，dispatch 和 commit。换言之，你在使用模块内容（module assets）时不需要在同一模块内额外添加空间名前缀。更改 namespaced 属性后不需要修改模块内的代码。

## 4. 配合 TypeScript

Vuex4 使用 TypeScript 还是有一定的麻烦，Vue 提供了一个 InjectionKey 接口，该接口是扩展 Symbol 的泛型类型。它可用于在提供者和消费者之间同步 inject 值的类型。

```typescript
import { createStore, Store } from 'vuex'
import { InjectionKey } from 'vue'
import { Menu } from './settings';

export interface Menu {
    icon?: string;
    text: string;
    url: string;
    id: string;
}

interface MenuState {
  menus: Menu[];
}

const MenuKey: InjectionKey<Store<MenuState>> = Symbol()

const menuStore = createStore<MenuState>({
  state: {
    menus: []
  },
  mutations: {
    add(state: MenuState, menu: Menu) {
      state.menus.push(menu);
    }
  },
  actions: {
    add(context, menu: Menu) {
      context.commit('add', menu);
    }
  },
  getters: {
    menus(state) {
      return state.menus;
    }
  }
})

export { Menu, MenuKey, menuStore };
```

在 main.ts 中注入依赖：

```typescript
import { MenuKey, menuStore } from './store'
// ...

app.use(menuStore, MenuKey);

// ...
```
