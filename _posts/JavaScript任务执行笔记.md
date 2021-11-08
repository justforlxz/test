---
title: JavaScript任务执行笔记
date: 2021-08-07 10:37:18
tags: JavaScript
categories: Web
---

浏览器中的 JavaScript 的执行流程和 Node.js 中的执行流程都是基于 **事件循环** 的

## 事件循环

**事件循环** 的概念非常简单，事件就是一个动作，界面点击会触发一个动作，生成点击事件，get 请求会触发一个动作，也会产生事件。

事件循环就是提供一个先入先出的队列，事件产生后加入到事件队列中，当队列头的事件被处理完以后，弹出头部的事件，执行下一个事件，不断循环。

在上述模型中，可以看出事件循环是在单一线程中执行的，那么如果此时有 IO 任务，线程岂不是会阻塞，在界面上的体现就是卡死、无法响应。

亦或者有些代码执行了大量计算，比方说在前端暴力破解密码之类的鬼操作，这就会导致后续代码一直在等待，页面处于假死状态，因为前边的代码并没有执行完。

所以如果全部代码都是同步执行的，这会引发很严重的问题，比方说我们要从远端获取一些数据，难道要一直循环代码去判断是否拿到了返回结果么？

于是有了异步事件的概念，注册一个回调函数，比如说发一个网络请求，我们告诉主程序等到接收到数据后通知我，然后我们就可以去做其他的事情了。

然后在异步完成后，会通知到我们，但是此时可能程序正在做其他的事情，所以即使异步完成了也需要在一旁等待，等到程序空闲下来才有时间去看哪些异步已经完成了，可以去执行。

## 宏任务和微任务

为什么要有宏任务和微任务的区别？

这里举一个经常看到的例子，假如 A 去银行办理业务，排了半天队伍，终于到 A了，在 A 的业务办理的过程中，A 想要除了存钱以外，还想要修改银行卡密码，可是总不能让 A 重新排队吧，这样太浪费时间了，所以 A 可以在当前业务办理完成后，立即执行修改密码的操作，而不用重新到队伍里排队。

A 办理业务的过程就是宏任务，而在办理业务中产生的新的业务，就是微任务。

在 JavaScript 中，宏任务是指普通的任务，例如渲染、请求、脚本、延时、io等。而微任务是指 promise 回调、proxy、监听DOM、nextTick等。

执行流程是宏任务队列在执行一个任务时，任务可能会产生新的任务，新的任务会根据类型来决定是放置在宏任务队列，还是开辟一个新的任务队列，在当前任务执行完毕后执行这个队列。

这是一个执行过程：

```javascript
<script>
setTimeout(function () {
        console.log('setTimeout')
    }, 0)
    new Promise(function (resolve) {
        console.log('promise1')
        for( let i = 0; i < 1000; i++ ) {
            i === 999 && resolve()
        }
        console.log('promise2')
    }).then(function ()  {
        console.log('promise3')
    })
    console.log('script')
</script>
```

输出结果： `promise1` → `promise2` → `script` → `promise3` → `setTimeout`

## async / await

`async` / `await` 是 ES7 引入的重大改进，它本质上是语法糖，可以以同步的方式写异步代码，不会产生回调地狱。

`async` ：异步执行和返回 Promise

`await` ：返回 Promise 对象

在没有 Promise 之前，我们写的异步代码必须用 setTimeout 来执行一个异步函数，很容易就会产生回调地狱，setTimeout 里嵌套 setTimeout。当 ES6 引入了 Promise，我们终于有了专门执行异步代码的结构，但是 Promise 仍然会产生回调地狱，所以在 ES7，引入了 async / await 语法糖，可以很方便的写出同步方式的异步代码。

**看代码**

```javascript
async function do2() {
    return 2;
}

async function do1() {
  console.log(1);
  let a = await do2();
  console.log(a);
  console.log(3);
}

console.log(4);
do1();
console.log(5);
```

输出结果： `4 1 5 2 3`

在代码中可以看到，函数增加一个 async 关键字，async 的意思是该函数会以异步的方式执行，async 函数是 AsyncFunction 构造函数的实例，并且其中允许使用 await 关键字。async 和 await 关键字让我们可以用一种更简洁的方式写出基于 Promise 的异步行为，而无需刻意地链式调用 Promise

async 函数一定会返回一个 Promise 对象。如果一个 async 函数的返回值看起来不是 Promise，那么它将会被隐式地包装在一个 Promise 中。
