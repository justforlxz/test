---
title: CMake CTests for dde-control-center
date: 2019-05-23 17:16:15
tags: [
    CMake
    Linux
]
categories: Linux
---

什么是单元测试?

>在计算机编程中，单元测试又称为模块测试，是针对程序模块（软件设计的最小单位）来进行正确性检验的测试工作。

单元测试存在的意义在于，如果程序发生了异常情况，比如接收了错误的值，从而导致结果不正确，当修正程序中的错误后，为了避免再次遇到这个问题，需要对出问题的值和函数/功能进行一次测试，确保结果符合预期。

单元测试很重要，如果是新项目，请一定要刚开始就规划好单元测试。

为什么说单元测试很重要呢？因为单元测试的目的是隔离其他单元，并证明当前单元是正确的。这需要开发者在设计程序的时候就要考虑很多，合理的设计和规划项目。当未来重构项目的时候，可以局部重构来优化项目，而不是从零重写。

本文没有详细说明Qt的单元测试是如何编写的，编写Qt的单元测试放在以后再写(~~咕咕咕~~)。

<!-- more -->

写这篇文章是因为最近在给控制中心写单元测试，控制中心的模块都是MVC的，本身就做好了大方向的隔离，每个函数也基本是拆分出来的最小功能，可以单独拿出来测试。控制中心目前存在一个问题，Worker类是从DBus上接收数据，处理完成后放入Model中，如果测试Worker类，需要做很多和DBus相关的处理，比较麻烦，所以最开始我先把重心放在了创建Tests和测试一个基本的转换函数的功能，验证单元测试的流程。

>[控制中心单元测试PR](https://github.com/linuxdeepin/dde-control-center/pull/171)

控制中心项目使用的CMake作为项目构建工具，所以用到了CTests，控制中心使用的Qt进行的开发，Qt也提供了自己的单元测试，我两个都做了支持。

在顶层的CMakeLists.txt中添加CTests的支持：
```
# 启用CTest检查
include(Dart)

# 启用CTest
include(CTest)
```
这两行内容需要在顶层CMakeLists.txt中添加，不然不会生效。

在子项目中创建一个dcc_test.h，用来写单元测试的类。

```
#ifndef DCC_TEST_H
#define DCC_TEST_H

#include <QMap>
#include <QString>
#include <QTest>

#include "modules/display/displaywidget.h"

namespace Tests {

class Tests : public QObject {
    Q_OBJECT

private Q_SLOTS:
    void testSliderValue_data()
    {
        QTest::addColumn<float>("value");
        QTest::addColumn<int>("result");

        QMap<float, int> testMap{ { 1.0, 1 },  { 1.25, 2 }, { 1.5, 3 },
                                  { 1.75, 4 }, { 2.0, 5 },  { 2.25, 6 },
                                  { 2.5, 7 },  { 2.75, 8 }, { 3.0, 9 } };

        for (auto it = testMap.constBegin(); it != testMap.constEnd(); ++it) {
            QTest::newRow("converToSlider") << it.key() << it.value();
        }
    }
    void testSliderValue()
    {
        QFETCH(float, value);
        QFETCH(int, result);

        using namespace dcc::display;

        QCOMPARE(DisplayWidget::convertToSlider(value), result);
        QCOMPARE(DisplayWidget::convertToScale(result), value);
    }
};
}  // namespace Tests

QTEST_MAIN(Tests::Tests)
#endif  // !DCC_TEST_H
```

在子项目的CMakeLists.txt中添加一个二进制，用来当作单元测试程序。

```
# 这个宏是Dart提供的，用来判断是否开启CTest
if(BUILD_TESTING)
find_package(Qt5 COMPONENTS
    Test
REQUIRED)

set(Qt_LIBS
    ${Qt_LIBS}
    Qt5::Test
)

set(TEST_SRCS
tests/dcc_test.h
${DISPLAY_FILES}
${WIDGETS_FILES}
${MODULE_FILES}
)

# 添加一个叫unit-test的二进制
add_executable(unit-test
${TEST_SRCS}
${PROJECT_BINARY_DIR}
)

target_include_directories(unit-test PUBLIC
${TEST_SRCS}
${PROJECT_BINARY_DIR}
${DFrameworkDBus_INCLUDE_DIRS}
${QGSettings_INCLUDE_DIRS}
${Qt5Gui_PRIVATE_INCLUDE_DIRS}
)

target_link_libraries(unit-test PRIVATE
${Qt_LIBS}
${DFrameworkDBus_LIBRARIES}
${QGSettings_LIBRARIES}
${DtkWidget_LIBRARIES}
${XCB_EWMH_LIBRARIES}
)
```

到这里，直接编译启动unit-test就可以使用Qt的单元测试了，但是加上CTest的支持只需要一行：

```
add_test(ctest unit-test)
endif()
```

使用ctest -j6 -C Debug -T test --output-on-failure跑CTest，得到执行结果：

```
[ctest]    Site: xiaomi-air
[ctest]    Build name: Linux-c++
[ctest] Test project /home/justforlxz/Projects/Deepin/dde-control-center/build
[ctest]     Start 1: ctest
[ctest] 1/1 Test #1: ctest ............................   Passed    0.05 sec
[ctest]
[ctest] 100% tests passed, 0 tests failed out of 1
[ctest]
[ctest] Total Test time (real) =   0.06 sec
[ctest] CTest finished with return code 0
```

如果是跑unit-test二进制，则会得到Qt打印的相关信息：

```
********* Start testing of Tests::Tests *********
Config: Using QtTest library 5.12.3, Qt 5.12.3 (x86_64-little_endian-lp64 shared (dynamic) release build; by GCC 8.3.0)
PASS   : Tests::Tests::initTestCase()
PASS   : Tests::Tests::testSliderValue(converToSlider)
PASS   : Tests::Tests::testSliderValue(converToSlider)
PASS   : Tests::Tests::testSliderValue(converToSlider)
PASS   : Tests::Tests::testSliderValue(converToSlider)
PASS   : Tests::Tests::testSliderValue(converToSlider)
PASS   : Tests::Tests::testSliderValue(converToSlider)
PASS   : Tests::Tests::testSliderValue(converToSlider)
PASS   : Tests::Tests::testSliderValue(converToSlider)
PASS   : Tests::Tests::testSliderValue(converToSlider)
PASS   : Tests::Tests::cleanupTestCase()
Totals: 11 passed, 0 failed, 0 skipped, 0 blacklisted, 0ms
********* Finished testing of Tests::Tests *********
```

对比CTest和Qt的单元测试，Qt会告诉你详细的函数调用和执行过程，CTest更注重结果，不过在Qtcreator的单元测试面板中，会看到更好的输出。

说到底，CTest支持启动了一个带有单元测试的程序，而程序自己使用了Qt提供的单元测试类进行测试。
