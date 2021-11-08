---
title: CPP项目的一些坑
date: 2020-06-15 11:11:40
tags: [ 'C++' ]
categories:
---

本篇文章记录这几年项目中C++的一些问题和优化方法。需要注意的是，代码优化没有一本万利的方法，只能见招拆招，而且还要避免过早优化等问题，代码优化一定是要中后期才可以，而且不要为了优化而优化。

<!-- more -->

## const和const &

在接收一个返回值或者声明局部只读变量时没有使用const修饰。const的目的不仅仅是为了只读，更多的是编译器可以在此处提供优化。

```C++
QRect rect = m_displayInter->primaryRawRect();
qreal scale = qApp->primaryScreen()->devicePixelRatio();
```

在这两行例子中，react和scale都在当前函数内没有任何修改，而且不应该修改，需要添加const来修饰只读，并且QRect应该使用&来减少内存复制带来的额外影响。

## 类型强转

在部分代码中，经常能看到C风格的代码强转，应当根据具体情况使用static_cast、dynamic_cast和reinterpret_cast。

static_cast是使用的比较多的cast，经常用于派生类和基类之间转换。dynamic_cast也用于派生类和基类的转换，如果类型T是指针类型，若转换失败，则返回T类型的空指针，如果时T是引用类型，则会抛出异常，返回std::bad_cast。reinterpret_cast并不会做实际的转换，只会在编译时进行检查，如果不能进行cast转换，则编译报错。

## 过多的嵌套

过多的嵌套会严重影响代码阅读，经常出现只有if通过才会进入执行的情况，这种情况应该修改为不通过就不要继续执行，或者安排合理的if将条件限制在之前。

```C++
void BluetoothWorker::setAdapterPowered(const Adapter *adapter, const bool &powered)
{
    QDBusObjectPath path(adapter->id());
    //关闭蓝牙之前删除历史蓝牙设备列表，确保完全是删除后再设置开关
    if (!powered) {
        QDBusPendingCall call = m_bluetoothInter->ClearUnpairedDevice();
        QDBusPendingCallWatcher *watcher = new QDBusPendingCallWatcher(call, this);
        connect(watcher, &QDBusPendingCallWatcher::finished, [ = ] {
            if (!call.isError()) {
                QDBusPendingCall adapterPoweredOffCall  = m_bluetoothInter->SetAdapterPowered(path, false);
                QDBusPendingCallWatcher *watcher = new QDBusPendingCallWatcher(adapterPoweredOffCall, this);
                connect(watcher, &QDBusPendingCallWatcher::finished, [this, adapterPoweredOffCall, adapter] {
                    if (!adapterPoweredOffCall.isError()) {
                        setAdapterDiscoverable(adapter->id());
                    } else {
                        qWarning() << adapterPoweredOffCall.error().message();
                    }
                });
            } else {
                qWarning() << call.error().message();
            }
        });
    } else {
        QDBusPendingCall adapterPoweredOnCall  = m_bluetoothInter->SetAdapterPowered(path, true);
        QDBusPendingCallWatcher *watcher = new QDBusPendingCallWatcher(adapterPoweredOnCall, this);
        connect(watcher, &QDBusPendingCallWatcher::finished, [this, adapterPoweredOnCall, adapter] {
            if (!adapterPoweredOnCall.isError()) {
                setAdapterDiscoverable(adapter->id());
            } else {
                qWarning() << adapterPoweredOnCall.error().message();
            }
        });
    }
}
```

这里的代码其实是可以优化的，我们可以通过三元表达式获取某个QDBusPendingCall，这样就可以使用一个QDBusPendingCallWatcher对象，然后将原本的lambda内容提取到其他函数内，在新的lambda中同样使用三元表达式运行对应的函数，这样拆分代码的好处是，阅读代码时的顺序会和执行顺序一致，分支判断对机器和人类都不是太友好，特别是判断体内有很长的代码段，找到else段是一件不容易的事情，通过降低if else块来提高代码可读性。同时应提取相同动作的代码到公共区域，以免将来修改时发现没有将所有的地方都做修改。

## 循环

避免使用数组的方式来访问元素，使用迭代器的方式统一循环方式。

我注意到有些情况下，有人在for循环内直接定义静态变量，这种方式使用的时候需要注意，静态变量将会永远存在，但是大部分for循环内需要保存的数据都是成员变量，否则内存空间将永远不会释放，对内存有浪费。

而且经常遇到的问题就是foreach宏和for混用，在语法上就没有统一使用。

我推荐的方式是for+迭代器的方式，如果是简单遍历，使用原生的foreach语法即可。

```C++
std::list<int> list{ 1, 2, 3, 4};

// 原生foreach语法，推荐只读遍历使用
for (int item: list) {
    ...
}

// for+迭代器，只读遍历
for (auto it = list.cbegin(); it != list.cend(); ++it) {
    // it是迭代器对象，需要解引用使用。
    *it
}

// for+迭代器方式，推荐需要修改容器的长度使用
for (auto it = list.begin(); it != list.end();) {
    // 注意，如果要移除某个元素，需要手动下一步
    if (*it == 2) {
        it = list.erase(it);
    }
    else {
        ++it;
    }
}
```

## 内存泄漏

经常遇到使用容器将指针保存下来的场景，但是当对象被析构或者容器被清空的时候，有时候会忘记删除内部的对象，或者删除了不该删除的对象。对数据的处理应该保持RAII原则，避免直接使用裸指针，而是通过智能指针将指针保存起来，当最后一个对象不再持有智能指针对象时，智能指针会删除持有的对象，完成内存释放。

智能指针的类型

智能指针包含有三种：独占指针`unique_ptr`、共享指针`shared_ptr`和弱引用指针`week_ptr`。

### 独占指针

独占指针`std::unique_ptr`可以避免对象被转移到其他对象中，如果某个对象持有`unique_ptr`，则该ptr不允许转移给其他对象，但是可以使用`std::move`来转移控制权，注意这和普通的转移不一样，`unique_ptr`禁止的是拷贝，但是没有禁止移动，我们可以转移控制转，`unique_ptr`保证的是只有一个智能指针持有对象。

```C++
std::unique_ptr<T> p1 = std::move(ptr);
```

### 共享指针

共享指针`std::shared_ptr`顾名思义是用作共享的，和独占指针不同的是，它支持复制，内部通过引用计数来维持对象的生命周期，当没有任何一个对象持有共享指针时，也就意味着没有任何一个对象可以访问到内部对象了，就可以安全的删除对象，释放内存。

#### 弱引用指针

弱引用指针`std::week_ptr`是为了避免两个共享指针相互持有导致引用计数永远不会归零，导致内存永远不释放而提出的解决方案，具体就是弱引用指针不会导致引用计数增加，但是week_ptr同样不支持复制，必须转换为共享指针`std::shared_ptr`。

## 优化判断条件

对于常数的判断，尽量使用宏或者定义静态常量来避免直接使用数字或者字符判断。

## 排序

发现很多人在需要排序的时候总是使用冒泡算法，我介绍几个比较方便的排序方法。

### 使用std::sort

C++标准库提供了`std::sort`方法来方便的排序，它有三个参数，第一个参数是容器的begin迭代器，第二个参数是end迭代器，第三个参数接收一个返回值为bool类型的函数，该函数用于实现手动控制排序的判断。

我们可以提供一个lambda表达式来方便的控制排序，或者提供一个函数指针。

```C++
std::list<int> list{ 10, 4, 2, 5 };

std::sort(list.begin(), list.end(), [](int num1, int num2) {
    return num1 < num2;
});
```

这种排序方式是直接对原始容器进行操作的，如果不希望数据成为脏数据，应该先复制一份。

### 使用容器

使用容器的方式比较麻烦一些，我们需要对象自己支持大小比较，或者顺序是外部某个列表列表控制的。

我们可以使用map将内部数据和标记数据建立映射关系，再通过外部的list或者其他方式，从map中将数据读出来，添加到新的列表容器中，从而完成排序。

```C++
[
    "page1", "page2", "page3"
]

std::map<String, QWidget*> map;
…

const StringList & list = QJsonDocument::fromJson(readAll(“order.json”)).toStdList();

QList<QWidget*> pages;

for (const QString& key : list) {
    pages << map[key];
}
```
