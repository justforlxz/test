---
title: CTest & QTest/GTest
date: 2020-06-16 15:11:40
tags: [ 'C++', 'Qt', 'CMake', 'GTest', 'CTest' ]
categories: [ "unit test" ]
---

本文会介绍一下QTest和GTest的一些功能和区别。

<!-- more -->

# 单元测试

## ctest

ctest是CMake提供的运行单元测试的工具，在使用CMakeLists.txt文件编译工程的时候，CTest会自动configure、build、test和展现测试结果。

ctest有两个模式：

* 模式一：使用CMake configure和build工程，在CMakeLists.txt，使用特殊的命令创建tests。使用CTest来执行那些测试。
* 模式二：使用CTest来执行一个script，这个script的语法必须和CMakeLists.txt相同。

使用方法：

在CMakeLists.txt使用include(CTest)和include(Dart)来导入CTest模块和开启ctest。使用add_test()来添加一个测试程序，测试程序是一个普通的二进制，只不过内部运行的是qtest或者gtest编写的测试用例。

```plain
include(CTest)
include(Dart)
add_executable(tests tests/test.cpp)
add(NAME tests COMMAND $<TARGET_FILE:tests>)
```

## qt qtest

qtest是Qt提供的单元测试框架，Qt Test是用于对基于Qt的应用程序和库进行单元测试的框架。Qt Test提供了单元测试框架中常见的所有功能以及用于测试图形用户界面的扩展。

Qt测试旨在简化基于Qt的应用程序和库的单元测试的编写：

| 特征   | 描述   |
|:----|:----|
| 轻量   | Qt Test大约有6000行代码和60个导出符号组成   |
| 自成体系   | Qt Test仅需要Qt Core模块中的几个符号即可进行非GUI测试   |
| 快速测试   | Qt Test不需要特殊的测试运行程序，没有特殊的测试注册 |
| 数据驱动测试   | 可以使用不同的数据进行多次的测试   |
| 基本的GUI测试   | Qt Test提供了用于鼠标和键盘的模拟功能   |
| 标杆管理   | Qt Test支持基准测试，并提供多个测量后端   |
| IDE友好   | Qt Test输出可以由Qt Creator、Visual Studio等IDE解释的消息   |
| 线程安全   | 错误报告是线程安全和原子的   |
| 类型安全   | 模板的广泛使用可以防止隐式类型转换引起的错误   |
| 易于扩展   | 可以将自定义类型轻松添加到测试数据和测试输出中   |

### 断言

QVERIFY() 用于验证数据是否正确。

```c++
QVERIFY(1 == 1);
QVERIFY2(1 != 1, "1不等于1");
```

### 循环

QFETCH_GLOBAL() 该宏从全局数据表中的一行中获取类型类型为name的变量。 名称和类型必须与全局数据表中的列匹配。 这是断言，如果断言失败，则测试将中止。

QFETCH() 宏会在堆栈上创建一个类型为name的本地变量。 名称和类型必须与测试数据表中的列匹配。 这是断言，如果断言失败，则测试将中止。

```c++
QFETCH(QString, aString);
QFETCH_GLOBAL(QLocale, locale);
```

### 比较

QCOMPARE宏用于判断两个值是否相等，如果实际值和预期值匹配，将会继续运行，否则将失败记录在测试日至中，并且测试将被终止，不会尝试任何后续操作。

```c++
QCOMPARE(QString("hello").toUpper(), QString("HELLO"));
```

### 添加数据

通过在包含_data()的函数中调用QTest::addColumn和QTest::newRow向测试用例增加数据，并通过QFETCH宏在测试用例中访问数据。

```c++
void TestQString::toInt_data()
{
    QTest::addColumn<QString>("aString");
    QTest::addColumn<int>("expected");
    QTest::newRow("positive value") << "42" << 42;
    QTest::newRow("negative value") << "-42" << -42;
    QTest::newRow("zero") << "0" << 0;
}
void TestQString::toInt()
{
     QFETCH(QString, aString);
     QFETCH(int, expected);
     QCOMPARE(aString.toInt(), expected);
}
```

### 创建测试

要创建测试，需要派生自QObject，并添加一个或多个专用槽函数。每个专用槽函数都是测试中的一个测试功能且必须为private。函数命名方法以casen_函数名或者以test结尾的方式。

使用QTest::qExec()可用于执行测试对象中的所有测试功能。

此外，还可以定义不用于测试功能的专用槽函数。如果存在，它们将由测试框架执行，并用于初始化和清除整个测试或当前的测试功能。

* **initTestCase()** 将在第一个测试功能执行之前被调用
* **initTestCase_data()** 将被调用以创建全局测试数据表
* **cleanupTestCase()** 在最后一个测试函数执行后被调用
* **init()** 将在每个测试功能执行之前被调用
* **cleanup()** 将在每个测试函数之后调用

使用initTestCase()准备测试。每次测试都应使系统处于可用状态，因此可以重复运行。清理操作应在cleanupTestCase()中处理，因此即使测试失败也可以运行清理操作。

使用init()创建测试功能。每个测试功能都应使系统保持可用状态，以便可以重复运行。清理操作应在cleanup()中，即使测试功能失败并提前退出，清理动作也可以运行。

另外，可以使用RAII,并在析构函数中调用清除操作，以确保他们在测试函数返回且对象移出作用域时发生。

如果initTestCase()失败，将不执行任何测试功能。如果init()失败，则不执行以下测试功能，测试将继续进行下一个测试功能。

```c++
class Test : public QObject {
  Q_OBJECT
private:
  bool verify() {
    return true;
  }
private slots:
  void initTestCase() {
    qDebug() << "init test case";
  }
  void firstTest() {
    QVERIFY(true);
    QCOMPARE(1, 1);
  }
  void secondTest() {
    QVERIFY(verify());
    QVERIFY(1 != 2);
  }
  void cleanupTestCase() {
    qDebug() << "clean test case";
  }
};
```

最后，如果测试类具有静态且公共的void initMain()方法，则在实例化QApplication对象之前，由QTEST_MAIN宏调用该方法。例如，这允许设置应用程序的属性，例如Qt::AA_DisableHighDpiScaling。这是在Qt5.14添加的。

### 使用CMake和CTest构建测试项目

CMake还有其他优点。例如，几乎可以毫不费力地使用CDash将测试运行的结果发布到Web服务器上。

CTest可以扩展到非常不同的单元测试框架，并且可以与QTest一起使用。

```plain
project(mytest LANGUAGES CXX)
include(CTest)
include(Dart)
find_package(Qt5 COMPONENTS Test REQUIRED)
set(CMAKE_INCLUDE_CURRENT_DIR ON)
set(CMAKE_AUTOMOC ON)
add_executable(mytest tst_mytest.cpp)
add_test(NAME mytest COMMAND mytest)
target_link_libraries(mytest PRIVATE Qt5::Test)
```

## google test

google test(gtest)是google公司推出的c++单元测试框架，基于xUnit架构，并且支持Linux、Windows和mac，并且支持任何类型的测试和模拟，而不仅仅是单元测试。

### 基本概念

当使用gtest时，通过编写断言来检查条件是否为真。断言的结果可能是成功、非致命失败或者致命失败。如果发生致命故障，将终止当前功能，否则将继续运行。

一个测试套件包含一个或者多个测试。当测试套件中的多个测试需要共享通用对象和子例程时，可以将他们放入一个测试桶中。

一个测试程序可以包含多个测试套件。

### 断言

gtest断言类似于函数调用的宏，可以通过断言其行为来测试类或者函数。断言失败时，gtest会输出断言的源文件和行号位置以及失败消息。还可以提供自定义失败消息，该消息将会附加到gtest的消息之后。

断言成对出现，测试相同的事物，但是对当前函数有不同的影响。ASSERT_*版本失败时会产生致命错误，并终止当前功能。EXPECT_*会产生非致命错误，不会导致当前测试失败。通常EXPECT_*是首选，因为他们允许在测试中报告多个鼓掌，但是如果在断言失败时继续运行将没有意义时应当使用ASSERT_*。

由于ASSERT_*失败会从当前函数立即返回，可能会跳过其后的清理代码，导致内存泄漏。

### 基本断言

基本断言可以进行基本的真/假条件测试

| 致命断言   | 非致命断言   | 验证   |
|:----|:----|:----|
| ASSERT_TRUE(condition);   | EXPECT_TRUE(condition);   | condition是真的   |
| ASSERT_FALSE(condition);   | EXPECT_FLASE(condition);   | condition是假的   |

请记住，当它们失败时，将导致ASSERT_*致命故障并从当前函数返回，而当它们发生EXPECT_*非致命故障时，将允许该函数继续运行。无论哪种情况，断言失败都意味着其包含测试失败。

### 字符串比较

该组中的断言比较两个C字符串。如果要比较两个string对象，请改用EXPECT_EQ，EXPECT_NE等。

| 致命断言   | 非致命断言   | 验证   |
|:----:|:----:|:----:|
| ASSERT_STREQ(str1,str2); | EXPECT_STREQ(str1,str2); | 这两个C字符串的内容相同 |
| ASSERT_STRNE(str1,str2); | EXPECT_STRNE(str1,str2); | 两个C字符串的内容不同 |
| ASSERT_STRCASEEQ(str1,str2); | EXPECT_STRCASEEQ(str1,str2); | 两个C字符串的内容相同，忽略大小写 |
| ASSERT_STRCASENE(str1,str2); | EXPECT_STRCASENE(str1,str2); | 两个C字符串的内容不同，忽略大小写 |

注意，断言名称中的“ CASE”表示忽略大小写。一个NULL 指针和一个空字符串被认为是不同的。

*STREQ*并*STRNE*接受宽C字符串（wchar_t*）。如果两个宽字符串的比较失败，则它们的值将打印为UTF-8窄字符串。

### 简单测试

创建测试：

1. 使用TEST()宏定义和命名测试功能。这些是没有返回值的普通C++函数。
2. 在此函数，要与包含的所有有效C++语句一起使用各种gtest断言来检查。
3. 测试结果由断言确定，如果测试中的任何声明失败（致命或非致命），或者测试崩溃，整个测试都会失败，否则测试应当成功。

```plain
TEST(TestSuiteName, TestName) {
  ...测试代码...
}
```

TEST()函数第一个参数是测试套件的名称，第二个参数是测试套件内的测试名称。这两个名称都必须是有效的C++标识符，并且不应包含任何下划线。来自不同测试套件的测试可以具有相同的名称。
# 参考资料：

## qtest

[https://doc.qt.io/qt-5/qtest.html](https://doc.qt.io/qt-5/qtest.html)

## gtest

[https://github.com/google/googletest/blob/master/googletest/docs/primer.md](https://github.com/google/googletest/blob/master/googletest/docs/primer.md)
