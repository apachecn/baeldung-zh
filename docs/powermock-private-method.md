# 使用 PowerMock 模拟私有方法

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/powermock-private-method>

## 1。概述

单元测试的挑战之一是模仿私有方法。

在本教程中，我们将了解如何通过使用由 JUnit 和 TestNG 支持的 [PowerMock](https://web.archive.org/web/20220627174223/https://github.com/powermock/powermock) 库来实现这一点。

PowerMock 与 EasyMock 和 Mockito 等模仿框架集成，旨在为这些框架添加额外的功能，如模仿私有方法、最终类和最终方法等。

它依靠字节码操作和一个完全独立的类加载器来实现。

## 2。Maven 依赖关系

首先，让我们将使用 PowerMock 和 Mockito 以及 JUnit 所需的依赖项添加到我们的`pom.xml`:

```
<dependency>
    <groupId>org.powermock</groupId>
    <artifactId>powermock-module-junit4</artifactId>
    <version>1.7.3</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.powermock</groupId>
    <artifactId>powermock-api-mockito2</artifactId>
    <version>1.7.3</version>
    <scope>test</scope>
</dependency>
```

最新版本可以查看[这里](https://web.archive.org/web/20220627174223/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22powermock-module-junit4%22)和[这里](https://web.archive.org/web/20220627174223/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22powermock-api-mockito2%22)。

## 3。示例

让我们从一个`LuckyNumberGenerator.` 的例子开始。这个类有一个生成幸运数字的公共方法:

```
public int getLuckyNumber(String name) {
    saveIntoDatabase(name);
    if (name == null) {
        return getDefaultLuckyNumber();
    }
    return getComputedLuckyNumber(name.length());
}
```

## 4。私有方法模仿的变体

对于方法的详尽单元测试，我们需要模仿私有方法。

### 4.1。没有参数但有返回值的方法

作为一个简单的例子，让我们模拟一个没有参数的私有方法的行为，并强制它返回所需的值:

```
LuckyNumberGenerator mock = spy(new LuckyNumberGenerator());

when(mock, "getDefaultLuckyNumber").thenReturn(300); 
```

在这种情况下，我们模仿私有方法`getDefaultLuckyNumber`并使其返回值 300。

### 4.2。带参数和返回值的方法

接下来，让我们用一个参数模拟一个私有方法的行为，并强制它返回所需的值:

```
LuckyNumberGenerator mock = spy(new LuckyNumberGenerator());

doReturn(1).when(mock, "getComputedLuckyNumber", ArgumentMatchers.anyInt()); 
```

在这种情况下，我们模仿私有方法并使它返回 1。

注意，我们并不关心输入参数，而是使用`ArgumentMatchers.anyInt()`作为通配符。

### 4.3。方法调用的验证

我们最后的策略是使用 PowerMock 来验证私有方法的调用:

```
LuckyNumberGenerator mock = spy(new LuckyNumberGenerator());
int result = mock.getLuckyNumber("Tyranosorous");

verifyPrivate(mock).invoke("saveIntoDatabase", ArgumentMatchers.anyString()); 
```

## 5。提醒一句

最后，尽管私有方法可以使用 PowerMock 进行测试，但是在使用这种技术时，我们必须格外小心。

鉴于我们测试的目的是验证一个类的行为，我们应该避免在单元测试期间改变类的内部行为。

嘲讽技术应该应用于类的外部依赖，而不是类本身。

如果模仿私有方法对于测试我们的类是必不可少的，那么这通常意味着一个糟糕的设计。

## 6。结论

在这篇简短的文章中，我们展示了如何使用 PowerMock 来扩展 Mockito 的功能，以模拟和验证被测类中的私有方法。

本教程的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220627174223/https://github.com/eugenp/tutorials/tree/master/testing-modules/powermock)