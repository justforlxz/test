---
title: JavaScript建造者模式
date: 2020-02-01 20:52:58
tags: JavaScript
categories: 设计模式
---

建造者模式就是指将类的构造和其表示分离开来，调用者可以通过不同的构建过程创造出不同表示的对象。主要解决在软件系统中，有时候面临着"一个复杂对象"的创建工作，由于需求的变化，这个复杂对象的某些部分经常面临着剧烈的变化，一些基本部件不会变。所以需要将变与不变分离。与抽象工厂的区别：在建造者模式里，有个指导者(Director)，由指导者来管理建造者，用户是与指导者联系的，指导者联系建造者最后得到产品。即建造者模式可以强制实行一种分步骤进行的建造过程。

<!-- more -->

##  建造者模式四要素

1. 产品类Product: 一般是一个较为复杂的对象，也就是说创建对象的过程比较复杂，一般会有较多的代码。
2. 抽象建造者类Builder: 将建造的具体过程交予它的子类来实现。
3. 建造者类ConcreateBuilder: 组件产品，返回组件好的产品
4. 指导类Director: 负责调用适当的建造者来组件产品，指导类一般不与产品类发生依赖关系，与指导类直接交互的是建造者类。

##  建造者模式的优点

建造者模式的封装很好，使用建造者模式可以进行有效的封装变化，在使用建造者模式的场景中，产品类和建造者类是比较稳定的，因此，将主要的业务逻辑封装在指导者类中对整体可以取得比较好的稳定性。

建造者类也很方便扩展，如果有新的需求，只需要实现一个新的建造者类即可。


产品类 product.ts
```
class Product {
    private _name: String;
    public name(): String {
        return this._name;
    }
    public setName(name: String) {
        this._name = name;
    }
}
```

抽象建造类 builder.ts
```
interface Builder {
    _product: Product;
    setName(name: String): Product;
    build(): Product;
}
```

建造类 concreatebuilder.ts
```
class ConcreateBuilder implements Builder {
    _product: Product = new Product;
    public setName(name: String): Product {
        this._product.setName(name);
        return this._product;
    }
    public build(): Product {
        return this._product;
    }
}

class HelloworldBuilder extends ConcreateBuilder {
    public build(): Product {
        this._product.setName("hello world!");
        return this._product;
    }
}
```

指导类 director.ts
```
class Director {
    private _defaultBuilder: ConcreateBuilder = new ConcreateBuilder;
    private _helloworldBuilder: HelloworldBuilder = new HelloworldBuilder;
    public buildForDefault(): Product  {
        return this._defaultBuilder.build();
    }
    public buildForHelloworld(): Product {
        return this._helloworldBuilder.build();
    }
}
```

测试运行:
```
let director = new Director();
console.log(director.buildForDefault().name());
console.log(director.buildForHelloworld().name());
```

执行结果
```
undefined
hello world!
```

通过不同的builder就可以构建不同的对象出来，当需求变动的时候，我们只需要扩展出不同的Builder和Director就可以满足。
