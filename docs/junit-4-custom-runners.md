# 自定义 JUnit 4 测试运行程序

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/junit-4-custom-runners>

## 1。概述

在这篇简短的文章中，我们将关注如何使用定制的测试运行程序来运行 JUnit 测试。

简单地说，为了指定定制的转轮，我们需要使用`@RunWith` 注释。

## 2。准备工作

让我们首先将标准的 [`JUnit`](https://web.archive.org/web/20221128040443/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22junit%22) 依赖项添加到我们的`pom.xml`中:

```java
<dependency>
  <groupId>junit</groupId>
  <artifactId>junit</artifactId>
  <version>4.12</version>  
</dependency>
```

## 3。实现自定义运行器

在下面的例子中，我们将展示如何编写我们自己的定制`Runner`并使用@ `RunWith`运行它。

**JUnit Runner 是一个扩展 JUnit 抽象`Runner` 类的类，它负责运行 JUnit 测试**，通常使用反射。

这里，我们实现了`Runner`类的抽象方法:

```java
public class TestRunner extends Runner {

    private Class testClass;
    public TestRunner(Class testClass) {
        super();
        this.testClass = testClass;
    }

    @Override
    public Description getDescription() {
        return Description
          .createTestDescription(testClass, "My runner description");
    }

    @Override
    public void run(RunNotifier notifier) {
        System.out.println("running the tests from MyRunner: " + testClass);
        try {
            Object testObject = testClass.newInstance();
            for (Method method : testClass.getMethods()) {
                if (method.isAnnotationPresent(Test.class)) {
                    notifier.fireTestStarted(Description
                      .createTestDescription(testClass, method.getName()));
                    method.invoke(testObject);
                    notifier.fireTestFinished(Description
                      .createTestDescription(testClass, method.getName()));
                }
            }
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }
}
```

`getDescription` 方法是从`Describable`继承而来的，它返回一个`Description`，其中包含了以后要导出的信息，可以被各种工具使用。

在`run`实现中，我们使用反射调用目标测试方法。

我们已经定义了一个接受`Class`参数的构造函数；这是 JUnit 的要求。在运行时，JUnit 会将目标测试类传递给这个构造函数。

`RunNotifier`用于激发包含测试进度信息的事件。

让我们在测试类中使用 runner:

```java
public class Calculator {
    public int add(int a, int b) {
        return a + b;
    }
}

@RunWith(TestRunner.class)
public class CalculatorTest {
    Calculator calculator = new Calculator();

    @Test
    public void testAddition() {
        Syste.out.println("in testAddition");
        assertEquals("addition", 8, calculator.add(5, 3));
    }
}
```

我们得到的结果是:

```java
-------------------------------------------------------
 T E S T S
-------------------------------------------------------
Running com.baeldung.junit.CalculatorTest
running the tests from MyRunner: class com.baeldung.junit.CalculatorTest
in testAddition
Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.002 sec

Results :

Tests run: 1, Failures: 0, Errors: 0, Skipped: 0
```

## 4。专业跑步者

我们可以扩展`Runner` : `ParentRunner`或`BlockJUnit4Runner`的一个专门化子类，而不是像上一个例子**那样扩展低级别的`Runner`类。**

抽象的`ParentRunner`类以分层的方式运行测试。

是一个具体的类，如果我们喜欢定制某些方法，我们可能会扩展这个类。

让我们看一个例子:

```java
public class BlockingTestRunner extends BlockJUnit4ClassRunner {
    public BlockingTestRunner(Class<?> klass) throws InitializationError {
        super(klass);
    }

    @Override
    protected Statement methodInvoker(FrameworkMethod method, Object test) {
        System.out.println("invoking: " + method.getName());
        return super.methodInvoker(method, test);
    }
}
```

用`@RunWith(JUnit4.class)` 注释一个类将总是调用当前版本 JUnit 中默认的 JUnit 4 runner 该类是当前默认 JUnit 4 类运行器的别名:

```java
@RunWith(JUnit4.class)
public class CalculatorTest {
    Calculator calculator = new Calculator();

    @Test
    public void testAddition() {
        assertEquals("addition", 8, calculator.add(5, 3));
    }
}
```

## 5。结论

JUnit 运行程序适应性很强，允许开发人员改变测试执行过程和整个测试过程。

如果我们只想做一些小的改动，看看`BlockJUnit4Class` runner 的保护方法是个好主意。

一些流行的第三方 runners 实现包括`SpringJUnit4ClassRunner, MockitoJUnitRunner, HierarchicalContextRunner,` `Cucumber Runner` 等等。

所有这些示例和代码片段的实现都可以在 [GitHub 项目](https://web.archive.org/web/20221128040443/https://github.com/eugenp/tutorials/tree/master/testing-modules/junit-4)中找到——这是一个 Maven 项目，因此它应该很容易导入和运行。