---
title: 浅谈Javascript构造器模式
date: 2020-01-31 23:15:23
tags: JavaScript
categories: 设计模式
---

为了简化操作，JavaScript提供了new关键字。new关键字用于创建一个用户定义类型的实例，或者具有构造函数的内置对象的实例。

<!-- more -->

每当我们在一个函数调用前使用new关键字，该函数便会以一种特殊模式——构造模式来运行，在此模式中，JavaScript可以自动完成一些操作。基本上它是指解释器在你的代码中嵌入几行操作代码。

在JavaScript中，构造函数通常是认为用来实现实例的，但是JavaScript中没有类的概念，但是有特殊的构造函数，通过new关键字来调用定义的构造函数，你可以告诉JavaScript你需要创建一个新对象，并且新对象的成员声明都是构造函数里定义的。在构造函数内部，this引用的是新创建的对象。

```
function People(name: String, age: Number) {
    this.name = name;
    this.age = age;
    this.output = function() {
        return this.name + "已经" + this.age + "岁了";
    }
}

let people = new People("justforlxz", 24);

console.log(people.output());
```

上面是个很简单的构造函数模式，我们从字面上this是people对象，但是其实并不是这样的，new运算符帮助我们生成了this的初始化代码。

new运算符一共做了三件事：

1. 创建一个空对象
2. 将空对象的原型赋值为构造器函数的原型
3. 更改构造器函数内部的this，将其指向新创建的对象

```
let tmp = new Object();
tmp.__proto__ = People.prototype;
People.call(tmp);
```

最后会经过一个判断，如果构造器函数设置了返回值，并且返回值是一个Object类型的话，就直接返回该Object，否则就会返回新创建的空对象。

总结一下： JavaScript没有类的概念，但是为了实现OOP，就通过new关键字实现对函数进行插入代码来实现对象实例的初始化。构造器模式就是通过一个方法来new出一个对象，这个操作就叫构造器模式。
