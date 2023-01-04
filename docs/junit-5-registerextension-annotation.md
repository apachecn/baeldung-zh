# 通过@RegisterExtension 注册 JUnit5 编程扩展

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/junit-5-registerextension-annotation>

## 1。概述

JUnit 5 提供了多种注册扩展的方法。关于这些方法的概述，请参考我们的 JUnit 5 扩展指南。

在这个快速教程中，我们将使用`[@RegisterExtension](https://web.archive.org/web/20220627091020/https://junit.org/junit5/docs/current/api/org.junit.jupiter.api/org/junit/jupiter/api/extension/RegisterExtension.html)`注释重点关注 JUnit 5 扩展的编程注册。

## 2。`@RegisterExtension`

我们可以将这种注释应用到测试类的字段中。这种方法的一个优点是我们可以在测试内容中直接访问作为对象的扩展。

JUnit 将在适当的阶段调用扩展方法。

例如，如果一个扩展实现了`BeforeEachCallback,`，JUnit 将在执行一个测试方法之前调用它相应的接口方法。

## 3。使用静态字段`@RegisterExtension`

当与静态字段一起使用时，JUnit 将在应用了基于类级`@ExtendWith`的扩展之后应用该扩展的方法。

此外，JUnit 将调用扩展的类级和方法级回调。

例如，下面的扩展包含了一个`beforeAll`和一个`beforeEach` 实现:

```java
public class LoggingExtension implements 
  BeforeAllCallback, BeforeEachCallback {

    // logger, constructor etc

    @Override
    public void beforeAll(ExtensionContext extensionContext) 
      throws Exception {
        logger.info("Type {} In beforeAll : {}", 
          type, extensionContext.getDisplayName());
    }

    @Override
    public void beforeEach(ExtensionContext extensionContext) throws Exception {
        logger.info("Type {} In beforeEach : {}",
          type, extensionContext.getDisplayName());
    }

    public String getType() {
        return type;
    }
}
```

让我们将这个扩展应用到测试的静态字段:

```java
public class RegisterExtensionTest {

    @RegisterExtension
    static LoggingExtension staticExtension = new LoggingExtension("static version");

    @Test
    public void demoTest() {
        // assertions
    }
}
```

输出显示了来自`beforeAll`和`beforeEach`方法的消息:

```java
Type static version In beforeAll : RegisterExtensionTest
Type static version In beforeEach : demoTest()
```

## 4。将`@RegisterExtension`用于实例字段

如果我们对非静态字段使用`RegisterExtension`，JUnit 将只在处理完所有`TestInstancePostProcessor`回调后应用扩展。

在这种情况下，JUnit 将忽略类似于`beforeAll`的类级回调。

在上面的例子中，让我们从`LoggingExtension`中移除`static`修饰符:

```java
@RegisterExtension
LoggingExtension instanceLevelExtension = new LoggingExtension("instance version");
```

现在 JUnit 将只调用`beforeEach`方法，如输出所示:

```java
Type instance version In beforeEach : demoTest()
```

## 5。结论

在本文中，我们概述了用`@RegisterExtension`以编程方式注册 JUnit 5 扩展。

我们还讨论了将扩展应用于静态字段和实例字段之间的区别。

像往常一样，代码示例可以在我们的 [Github 库](https://web.archive.org/web/20220627091020/https://github.com/eugenp/tutorials/tree/master/testing-modules/junit5-annotations)中找到。