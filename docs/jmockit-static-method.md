# 使用 JMockit 模拟静态方法

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jmockit-static-method>

## 1。概述

一些流行的模仿库如 [Mockito 和 Easymock](/web/20221128042548/https://www.baeldung.com/mockito-vs-easymock-vs-jmockit) 通过利用 Java 的基于继承的类模型来生成模型。 **`EasyMock`在运行时实现一个接口，而`Mockito`从目标类继承来创建一个模仿存根。**

这两种方法都不适用于静态方法，因为静态方法与类相关联，并且不能被重写。然而， **JMockit 确实提供了静态方法模仿特性。**

在本教程中，我们将探索其中的一些功能。

关于 JMockit 的介绍，请参见我们的[上一篇文章](/web/20221128042548/https://www.baeldung.com/jmockit-101)。

## 2。Maven 依赖关系

让我们从 Maven 依赖项开始:

```java
<dependency>
    <groupId>org.jmockit</groupId>
    <artifactId>jmockit</artifactId>
    <version>1.24</version>
    <scope>test</scope>
</dependency>
```

你可以在 [Maven Central](https://web.archive.org/web/20221128042548/https://search.maven.org/classic/#search%7Cgav%7C1%7Ca%3A%22jmockit%22) 上找到这些库的最新版本。

## 3。从非静态方法调用静态方法

首先，让我们考虑这样一种情况，当我们有一个非静态方法的类**在内部依赖于静态方法**时:

```java
public class AppManager {

    public boolean managerResponse(String question) {
        return AppManager.isResponsePositive(question);
    }

    public static boolean isResponsePositive(String value) {
        if (value == null) {
            return false;
        }
        int length = value.length();
        int randomNumber = randomNumber();
        return length == randomNumber ? true : false;
    }

    private static int randomNumber() {
        return new Random().nextInt(7);
    }
}
```

现在，我们想要测试方法`managerResponse().`，因为它的返回值依赖于另一个方法，我们需要模拟`isResponsePositive()`方法。

我们可以使用`JMockit's`匿名类`mockit.MockUp.MockUp<T>` ( `where T will be the class name`)和`@Mock `注释来模拟这个静态方法:

```java
@Test
public void givenAppManager_whenStaticMethodCalled_thenValidateExpectedResponse() {
    new MockUp<AppManager>() {
        @Mock
        public boolean isResponsePositive(String value) {
            return false;
        }
    };

    assertFalse(appManager.managerResponse("Some string..."));
}
```

在这里，我们用一个我们想用于测试的返回值来模拟`isResponsePositive()`。因此，使用 Junit-5 中可用的 [`Assertions`](/web/20221128042548/https://www.baeldung.com/junit-5-preview#new) 实用程序来验证预期的结果。

## 4。测试私有静态方法

在少数情况下，其他方法使用类的私有静态方法:

```java
private static Integer stringToInteger(String num) {
    return Integer.parseInt(num);
}
```

为了测试这样的方法，**，我们需要模仿私有静态方法**。我们可以使用`JMockit`提供的`Deencapsulation.invoke() `实用方法:

```java
@Test
public void givenAppManager_whenPrivateStaticMethod_thenValidateExpectedResponse() {
    int response = Deencapsulation.invoke(AppManager.class, "stringToInteger", "110");
    assertEquals(110, response);
}
```

顾名思义，它的用途是`de-encapsulate`一个对象的状态。通过这种方式，JMockit 简化了用其他方法无法测试的测试方法。

## 5。结论

在本文中，我们已经看到了如何使用`JMockit`来模仿静态方法。要更深入地了解 JMockit 的一些高级特性，请看我们的 JMockit 高级用法[文章](/web/20221128042548/https://www.baeldung.com/jmockit-advanced-usage)。

像往常一样，本教程的完整源代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20221128042548/https://github.com/eugenp/tutorials/tree/master/testing-modules/mocks)