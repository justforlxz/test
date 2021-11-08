---
title: 智能指针
s: cpp-smart-pointer
date: 2018-08-29 09:43:17
tags: C++
categories: C++
---


其实一直都对智能指针的应用场景不清楚，项目中也很少用到，今天在 [@zccrs](https://zccrs.com) 大佬的帮助下，大概理解了智能指针的作用和应用场景。

<!-- more -->

# 设计思想

智能指针依赖一种叫引用计数的手段来协助管理对象指针，通过引用计数为0时删除对象指针来完成内存的释放，本质上是通过栈对象来管理堆对象的一种方法。

## 传统做法

```
void test() {
    Test* t = new Test;
    ...
    if (...) {
        throw exception();
    }

    delete t;
}
```

当出现异常时，delete将不会被执行到，t也就泄露了。虽然我们可以在异常那里把delete给加上，但是在较为大型的项目中，如果对代码进行review来排查这种错误，将会是非常麻烦的一件事，所以为了避免内存泄漏，发明了基于引用技术的智能指针。

## 智能指针做法

```
void test() {
    std::unique_ptr<Test> t(new Test);
    ...
    if (...) {
        throw exception();
    }
}

```

如果不关心std::unique_ptr是什么，这段代码无意是糟糕的，new出来的Test对象根本没有地方被删除，内存泄露了。

但是不必担心，指针已经由std::unique_ptr来管理了，根本不会发生内存泄漏，对象将在离开函数作用域以后被删除。

这就是智能指针的方便之处。

# 智能指针的基本实现

智能指针都通过模板编程来实现，模板是C++的另一大功能，可以使我们更关心实现而不需要关心具体的对象，通过更加抽象的方式来编写程序。

智能指针有两层，里层用来保存对象的指针和引用计数，外层用来调用里层来控制引用计数。

里层的辅助类
```
template<typename T>
class P_ptr {
    private:
        friend class Pointer<T>;
        P_ptr(T t)
        : pointer(t)
        , count(1)
        {

        }

        uint count;
        T pointer;
}
```

外层的控制类

```
template<typename T>
class Pointer {
    public:
        Pointer(T t)
        : m_ptr(new P_ptr(t))
        {

        }

        Pointer(const Pointer &pointer)
        : m_ptr(pointer.m_ptr)
        {
            ++m_ptr->count;
        }

        Pointer& operator=(const Pointer &pointer) {
            ++pointer->count;
            if (--m_ptr->count == 0) { // 应对自赋值
                delete m_ptr;
            }
            m_ptr = pointer->m_ptr;
            return *this;
        }

        ~Pointer() {
            if (--m_ptr->count == 0) {
                delete m_ptr;
            }
        }

    private:
        P_ptr m_ptr;
}
```

通过重写控制类的拷贝构造函数和赋值运算符重载来更新引用计数。

使用实例

```
void test() {
    Pointer<Test> t(new Test); // 引用计数目前是1
    Pointer<Test> t1 = t; // t的引用计数是2，t1的引用计数也是2
}

// 离开作用域，t被删除，引用计数是1. t1被删除，引用计数为0，Test被删除，内存没有泄露。
```

这样我们就有一个简单的智能指针了，不过他还存在一些问题，比如循环引用导致内存泄漏，没有->和*的操作运算符等。所以我们需要更强大的智能指针来帮助我们。

# 几种智能指针的介绍

标准库提供了几个针对不同方面使用的智能指针，以满足我们的需求。

- unique_ptr
  > 只允许一个所有者，除非确信你需要共享该指针，则应该使用```shared_ptr```。可以转移到新的所有者，但是不会复制和共享。
- shared_ptr
  > 采用引用计数的智能指针，如果你想将一个原始指针分配给多个所有者，请使用该智能指针，直到```shared_ptr```所有者超出了范围或放弃所有权，才会删除原始指针，大小为两个指针，一个用于对象，一个用于引用计数。
- weak_ptr
  > 结合```shared_ptr```使用的特殊智能指针，提供一个或多个```shared_ptr```实例所拥有的对象的访问，但是不会增加引用计数。如果你想观察某个对象，但是不需要保持活动状态，则可以使用该智能指针。在某些情况下，需要断开```shared_ptr```实例间的循环引用。

# 如何正确的选择智能指针

智能指针只需要区分需不需要共享使用，如果外部需要使用这个对象，使用```shared_ptr```，否则就使用unique_ptr进行独占使用。

# 陷阱和坑

- 不要使用相同的内置指针来初始化多个智能指针
- 不要主动回收智能指针内原始指针的内存
- 不要使用智能指针的get来初始化或者reset另一个智能指针
- 智能指针管理的资源只会默认删除new分配的内存，如果不是new分配的，则需要使用删除器