---
title: Qt多线程
date: 2020-11-17 15:53:37
tags:
categories:
---

类型注册 Qt 有三种多线程的方式，分别是继承 QThread、使用 QObject 的 moveToThread 函数和 Qtconcurrent 协程。

在很多文章中，大家都推荐继承 QThread 类，并重写 run 方法，在 run 中使用耗时操作代码。这种方式让我们觉得 QThread 是线程的实体。创建一个 QThread 对象就认为是开辟了一个新的线程。这种讨巧的方法似乎能帮助我们快速入门，但是只要深度了解多线程编程就会发现，这样子做会使代码脱离我们的控制，代码越写越复杂。最典型的问题就是明明将代码放入了新线程，但是仍然在旧线程中运行。

<!-- more -->

**我们应该把耗时代码放在哪里？**

暂时不考虑多线程的情况，我们一般都会将耗时代码封装到一个类中。在考虑多线程的情况下，难道我们要将代码剥离出来放到某个地方吗？其实不用这么麻烦。在 qt4 时代，我们需要使用继承 QThread 的方法，这样会破坏我们原有的代码结构，并且 run 方法只能运行一段代码，如果我们有成千上万个函数，我们总不能封装如此多的 QThread。

所以在 Qt5 中，Qt 库完善了线程的亲和性以及信号槽机制，我们有了更为优雅的使用线程的方式，即 QObject::moveToThread()。这也是官方推荐的做法。

我们准备两个类来介绍和解释一下工作流程。

controller.hpp

```c++
#ifndef CONTROLLER_H
#define CONTROLLER_H
#include <QObject>
#include <QDebug>
class Controller : public QObject {
    Q_OBJECT
public:
    explicit Controller(QObject *parent = nullptr)
        : QObject(parent) {}
signals:
    void requestPing();
public slots:
    void pong() {
        qDebug() << Q_FUNC_INFO << this->thread();
        qDebug() << "pong";
    }
};
#endif // CONTROLLER_H
```

handler.hpp

```c++
#ifndef HANDLEER_H
#define HANDLEER_H
#include <QObject>
#include <QDebug>
#include <QThread>
class Handler : public QObject {
    Q_OBJECT
public:
    explicit Handler(QObject *parent = nullptr)
        : QObject(parent) {}
signals:
    void requestPong();
public slots:
    void ping() {
        qDebug() << Q_FUNC_INFO << this->thread();
        emit requestPong();
    }
};
#endif // HANDLEER_H
```

在 main.cpp 中初始化对象，并连接信号和槽。

```c++
#include <QCoreApplication>
#include <QThread>
#include "controller.h"
#include "handleer.h"
int main(int argc, char *argv[]) {
    QCoreApplication a(argc, argv);
    QThread* handleThread = new QThread;
    Controller* controller = new Controller;
    Handler* handler = new Handler;
    // 移动对象到新的线程
    handler->moveToThread(handleThread);
    handleThread->start();
    // 将对象的信号的槽绑定，注意必须在 moveToThread 之后链接。
    QObject::connect(controller, &Controller::requestPing, handler, &Handler::ping);
    QObject::connect(handler, &Handler::requestPong, controller, &Controller::pong);
    emit controller->requestPing();
    return a.exec();
}
```
看一下执行结果:

```plain
void Handler::ping() QThread(0x14ee080)
void Controller::pong() QThread(0x14e9e60)
pong
```
可以看出来两个函数获取到的QThread对象并不是同一个了。
使用 movetothread 将一个对象移动到新的线程，并通过信号调用目标函数，从而达到在新线程执行的目的。

使用这种方式，我们可以方便的通过操作QThread对象来控制线程的执行，例如设置线程的优先级别，暂停线程或者恢复线程。并且这种方式比继承QThread可以更加直观的感受到，QThread只是一个线程的管理类，而不是线程实体，如果采用继承的方式，则会认为QThread就是线程实体，从而造成一定的认知混乱。

还有一种多线程的方式，这种方案更加的灵活，不需要我们new新的QThread对象，是一个较高层次的API封装。QtConCurrent可根据计算机的 CPU 核数，自动调整运行的线程数目。

在使用Qtconcurrent之前需要添加对应的Qt模块concurrent。

在使用的时候，我们需要添加一个QFutureWatcher对象，用来控制和执行一个QFuture对象，并且通过finished信号接收QFuture对象的执行结果。

```c++
QFutureWatcher<bool>* watcher = new QFutureWatcher<bool>();
QObject::connect(watcher, &QFutureWatcher<bool>::finished, watcher, [=] {
    const bool result = watcher->result();
    qDebug() << result;
    watcher->deleteLater();
});
QFuture<bool> future = QtConcurrent::run([=]() -> bool {
    return true;
});
watcher->setFuture(future);
```

以上就是一个简单的例子，不难发现，Qt为我们提供了相当不错的解决方案，这种形式比较类似于QDbus对象使用QDbusPendingCallWatcher来异步获取结果的方式，使用起来非常容易上手。
在使用多线程的时候，我们需要注意一些事情：互斥与同步同步，类型注册，在线程中开辟线程。

在多线程开发中，我们需要注意的地方就有点多了，最重要的就是线程同步，我们需要使用一些手段，让不同线程中的函数可以正确的访问的数据。

* 互斥：一个公共资源同一时刻只能被一个进程或线程使用，多个进程或线程不能同时使用公共资源。
* 同步：两个或两个以上的进程或线程在运行过程中协同步调，按预定的先后次序运行。

解决方法：互斥锁，条件变量，读写锁，自旋锁，信号量（互斥与同步）。

在Qt编程中，我们可以利用Qt的信号与槽机制实现两个对象的通信，无论两个对象是否在同一个线程，但是我们传递参数需要注册给Qt的元对象系统，否则Qt将无法完成数据传递。

在Qt中注册自定义类型有两种方式，一种是qRegisterMetaType<T>()函数和Q_DECLARE_METATYPE(Type)宏。

这两种注册方式有不同的作用。使用qRegisterMetaType<T>()函数可以让自定义类型在Qt的信号槽中传递，而Q_DECLARE_METATYPE(Type)宏则是可以让注册的自定义类型使用QVariant进行包装。

在多线程开发中我们还需要注意，Qt存在半自动内存管理，这个内存管理方式会影响着我们使用多线程开发。我们在创建新的QObject对象时，如果制定了parent，则该对象将与父对象进行线程绑定。如果两个对象在不同的线程中，Qt会警告我们父对象的线程和当前对象的线程不是同一个，他们将无法使用Qt的connect函数进行消息传递。
