---
title: Qt Meta Object System
date: 2019-05-25 15:22:01
tags: [
    Qt
]
categories: Qt
---
C++支持运行时类型信息(Runtime type information，RTTI),可以通过typeid以及dynamic_cast关键字实现运行时类型判断，但是这个功能太弱了，C++还不支持反射机制，无法做到运行时获取对象的全部信息。

Qt通过扩展C++的内省机制，但是并没有使用C++的RTTI，而是采用了更强大的元对象系统(Meta Object System)来实现内省。基于内省机制，可以列出对象的方法和属性列表，并且能获取对象的全部信息。

<!-- more -->

## 元对象简介

Qt中的元对象系统是一个基于标准C++的扩展，为Qt提供了信号与槽机制、实时类型信息、动态属性系统。元对象系统基于QObject类、Q_OBJECT宏、元对象编译器MOC实现。

- QObject类

    每一个需要用元对象系统的类的基类

- Q_OBJECT宏

    定义每一个类的私有段，用来启用动态属性、信号和槽。

    在每个QObject类或者派生类中，如果没有启用Q_OBJECT宏，类的metaobject对象就不会被生成，调用对象的metaObject()返回的就是父类的metaobject对象，导致从类实例中获取的都是父对象的信息，从而类中定义的信号和槽都无法使用，所以任何基于QObject类，都需要启用Q_OBJECT宏。

- MOC实现

    MOC会分析C++源文件，如果发现头文件中定义了Q_OBJECT宏，就会动态的生成一个moc文件，源文件中包含Q_OBJECT的实现代码。

## 元对象系统的功能

元对象系统除了提供信号和槽的功能外，还提供了运行时的方法：

- QObject::metaObject()

  获取与一个类相关的metaObject。

- QMetaObject::className()

  运行时获取对象的类名，不需要编译器的RTTI支持。

-　QObject::inherits()

    用来判断一个对象是不是从一个特定的类继承的，必须是在QObject类直 接或间接的派生类当中。

- QObject::tr()和QObject::trUtf8()

  为软件提供国际化支持。

- QObject::setProperty()和QObject::property()

  根据属性名动态的设置和获取属性值

- qobject_cast()

  方法可以在QObject类之间提供动态转换，qobject_cast类似于C++中的dynamic_cast，但是qobject_cast不需要RTTI的支持。

Qt Meta Object的所有数据和方法都封装在QMetaObject类中，它包含一个Qt类的meta信息，并且提供查询功能。

1. 信号表: 与对应Qt类相关的定义和自定义signal的名字
2. 槽表: 与对应Qt类相关的定义和自定义slot的名字
3. 类信息表: Qt类的类型信息
4. 属性表: 与对应类中所有的属性名字
5. 指向parent meta object的指针

Q_OBJECT宏定义如下:

```
/* qmake ignore Q_OBJECT */
#define Q_OBJECT \
public: \
    QT_WARNING_PUSH \
    Q_OBJECT_NO_OVERRIDE_WARNING \
    static const QMetaObject staticMetaObject; \
    virtual const QMetaObject *metaObject() const; \
    virtual void *qt_metacast(const char *); \
    virtual int qt_metacall(QMetaObject::Call, int, void **); \
    QT_TR_FUNCTIONS \
private: \
    Q_OBJECT_NO_ATTRIBUTES_WARNING \
    Q_DECL_HIDDEN_STATIC_METACALL static void qt_static_metacall(QObject *, QMetaObject::Call, int, void **); \
    QT_WARNING_POP \
    struct QPrivateSignal {}; \
    QT_ANNOTATE_CLASS(qt_qobject, "")
```

staticMetaObject是静态常量，metaObject()是获取该对象指针的方法，所有QObject对象都会共享staticMetaObject变量，来完成信号和槽的功能。

```
struct { // private data
    const QMetaObject *superdata;
    const QByteArrayData *stringdata;
    const uint *data;
    typedef void (*StaticMetacallFunction)(QObject *,QMetaObject::Call, int, void **);
    StaticMetacallFunction static_metacall;
    const QMetaObject * const *relatedMetaObjects;
    void *extradata; //reserved for future use
} d;
```

以上是QMetaObject所定义的全部成员，记录了所有的signal、slot、property和class等信息。

const QMetaObject *superdata，这个变量指向与之对应的QObject类的父类对象，或者是祖先类的QMetaObject对象，每一个QObject类或者派生类都有可能有一个父类或者父类的父类，superdata指向的是最接近祖先类中的QMetaObject对象，如果没有父类，该值为nullptr。

const QByteArrayData *stringdata，这是一个指向string data的指针，在连续的内存中保存了若干字符串，里面保存的是MetaSystem的核心功能——属性。

const uint *data，这个指针指向一个无符号整形数组，该数组是个预定义的复合数据结构，由QMetaObjectPrivate类提供管理，保存了类的基本信息，和一些索引值。在不同的qobject中的数据长度不一样长，这取决于定义了多少的signal、slot和property。有一部分指出了前一个变量stringdata中不同字符串的索引值，但是需要注意的是，这里面的数值并不是直接标明了每一个字符串的索引值，这个数组还需要通过一个相应的算法计算后，才能获得正确的字符串的索引值。

const void *extradata，这是一个指向QMetaObjectExtraData数据结构的指针。
