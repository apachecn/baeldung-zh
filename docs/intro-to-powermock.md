# PowerMock 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/intro-to-powermock>

## 1。概述

在 mocking 框架的帮助下进行单元测试很久以来就被认为是一种有用的实践，特别是最近几年来, [Mockito](https://web.archive.org/web/20220704161927/http://mockito.org/) 框架已经统治了这个市场。

为了促进体面的代码设计并使公共 API 简单，一些期望的特性被有意地省略了。然而，在某些情况下，这些缺点迫使测试人员编写繁琐的代码，只是为了让模拟的创建变得可行。

这就是 PowerMock 框架发挥作用的地方。

[`PowerMockito`](https://web.archive.org/web/20220704161927/https://github.com/jayway/powermock/wiki/MockitoUsage) 是 PowerMock 支持 Mockito 的扩展 API。它提供了以简单的方式使用 Java 反射 API 的能力，以克服 Mockito 的问题，例如缺乏模仿 final、static 或 private 方法的能力。

本教程将介绍 PowerMockito API，并看看它是如何应用于测试的。

## 2。准备使用 PowerMockito 进行测试

为 Mockito 集成 PowerMock 支持的第一步是在 Maven POM 文件中包含以下两个依赖项:

```java
<dependency>
    <groupId>org.powermock</groupId>
    <artifactId>powermock-module-junit4</artifactId>
    <version>1.6.4</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.powermock</groupId>
    <artifactId>powermock-api-mockito</artifactId>
    <version>1.6.4</version>
    <scope>test</scope>
</dependency>
```

接下来，我们需要通过应用以下两个注释来准备使用`PowerMockito`的测试用例:

```java
@RunWith(PowerMockRunner.class)
@PrepareForTest(fullyQualifiedNames = "com.baeldung.powermockito.introduction.*")
```

注释`@PrepareForTest`中的 *fullyQualifiedNames* 元素表示我们想要模仿的类型的完全限定名的数组。在这种情况下，我们使用一个带有通配符的包名来告诉`PowerMockito`准备好`com.baeldung.powermockito.introduction`包中的所有类型进行模拟。

现在我们准备利用`PowerMockito`的力量。

## 3。模仿构造函数和最终方法

在这一节中，我们将演示在用`new`操作符实例化一个类时，如何获得一个模拟实例而不是真实实例，然后使用该对象模拟一个最终方法。

下面是我们将如何定义协作类，它的构造函数和最终方法将被模仿:

```java
public class CollaboratorWithFinalMethods {
    public final String helloMethod() {
        return "Hello World!";
    }
}
```

首先，我们使用`PowerMockito` API 创建一个模拟对象:

```java
CollaboratorWithFinalMethods mock = mock(CollaboratorWithFinalMethods.class);
```

接下来，我们设置一个期望，即无论何时调用该类的无参数构造函数，都应该返回一个模拟实例，而不是一个真实的实例:

```java
whenNew(CollaboratorWithFinalMethods.class).withNoArguments().thenReturn(mock);
```

让我们通过使用默认构造函数实例化`CollaboratorWithFinalMethods`类来看看这个构造模拟是如何工作的，然后我们将验证 PowerMock 的行为:

```java
CollaboratorWithFinalMethods collaborator = new CollaboratorWithFinalMethods();
verifyNew(CollaboratorWithFinalMethods.class).withNoArguments();
```

在下一步中，为最终方法设置一个期望值:

```java
when(collaborator.helloMethod()).thenReturn("Hello Baeldung!");
```

然后执行该方法:

```java
String welcome = collaborator.helloMethod();
```

下面的断言确认了`helloMethod`方法已经在*协作者*对象上被调用，并返回模仿期望设置的值:

```java
Mockito.verify(collaborator).helloMethod();
assertEquals("Hello Baeldung!", welcome);
```

如果我们想模仿一个特定的 final 方法，而不是一个对象中的所有 final 方法，`Mockito.spy(T object)`方法可能会派上用场。这在第 5 节中有说明。

## 4。模仿静态方法

假设我们想要模仿一个名为`CollaboratorWithStaticMethods`的类的静态方法。

下面是我们声明这个类的方法:

```java
public class CollaboratorWithStaticMethods {
    public static String firstMethod(String name) {
        return "Hello " + name + " !";
    }

    public static String secondMethod() {
        return "Hello no one!";
    }

    public static String thirdMethod() {
        return "Hello no one again!";
    }
}
```

为了模仿这些静态方法，我们需要用`PowerMockito` API 注册封闭类:

```java
mockStatic(CollaboratorWithStaticMethods.class);
```

或者，我们可以使用`Mockito.spy(Class<T> class)`方法模拟一个特定的方法，如下一节所示。

接下来，可以设置期望值来定义方法在被调用时应该返回的值:

```java
when(CollaboratorWithStaticMethods.firstMethod(Mockito.anyString()))
  .thenReturn("Hello Baeldung!");
when(CollaboratorWithStaticMethods.secondMethod()).thenReturn("Nothing special");
```

或者可以设置在调用`thirdMethod`方法时抛出异常:

```java
doThrow(new RuntimeException()).when(CollaboratorWithStaticMethods.class);
CollaboratorWithStaticMethods.thirdMethod();
```

现在是运行前两个方法的时候了:

```java
String firstWelcome = CollaboratorWithStaticMethods.firstMethod("Whoever");
String secondWelcome = CollaboratorWithStaticMethods.firstMethod("Whatever");
```

上面的调用不是调用真实类的成员，而是委托给 mock 的方法。

这些断言证明嘲弄已经生效:

```java
assertEquals("Hello Baeldung!", firstWelcome);
assertEquals("Hello Baeldung!", secondWelcome);
```

我们还能够验证 mock 方法的行为，包括一个方法被调用了多少次。

在这种情况下，`firstMethod`被调用了两次，而`secondMethod`从未被调用过:

```java
verifyStatic(Mockito.times(2));
CollaboratorWithStaticMethods.firstMethod(Mockito.anyString());

verifyStatic(Mockito.never());
CollaboratorWithStaticMethods.secondMethod();
```

**注意:**必须在任何静态方法验证之前调用`verifyStatic`方法，以便`PowerMockito`知道需要验证的是后续的方法调用。

最后，静态的`thirdMethod`方法应该抛出一个`RuntimeException`，就像之前在 mock 中声明的那样。

它由`@Test`注释的`expected`柠檬验证:

```java
@Test(expected = RuntimeException.class)
public void givenStaticMethods_whenUsingPowerMockito_thenCorrect() {
    // other methods   

    CollaboratorWithStaticMethods.thirdMethod();
}
```

## 5。部分嘲讽

不是模仿整个类，`PowerMockito` API 允许使用`spy`方法模仿它的一部分。

这个类将作为合作者来说明 PowerMock 对部分模仿的支持:

```java
public class CollaboratorForPartialMocking {
    public static String staticMethod() {
        return "Hello Baeldung!";
    }

    public final String finalMethod() {
        return "Hello Baeldung!";
    }

    private String privateMethod() {
        return "Hello Baeldung!";
    }

    public String privateMethodCaller() {
        return privateMethod() + " Welcome to the Java world.";
    }
}
```

让我们从模仿一个静态方法开始，它在上面的类定义中被命名为`staticMethod`。

首先，我们使用`PowerMockito` API 部分模拟`CollaboratorForPartialMocking`类，并为其静态方法设置一个期望值:

```java
spy(CollaboratorForPartialMocking.class);
when(CollaboratorForPartialMocking.staticMethod()).thenReturn("I am a static mock method.");
```

然后执行静态方法:

```java
returnValue = CollaboratorForPartialMocking.staticMethod();
```

嘲讽的行为得到了验证:

```java
verifyStatic();
CollaboratorForPartialMocking.staticMethod();
```

下面的断言通过将返回值与期望值进行比较来确认 mock 方法实际上已被调用:

```java
assertEquals("I am a static mock method.", returnValue);
```

现在是时候转到最终的私有方法了。

为了说明这些方法的部分嘲讽，我们需要实例化这个类，并告诉`PowerMockito` API 去`spy`它:

```java
CollaboratorForPartialMocking collaborator = new CollaboratorForPartialMocking();
CollaboratorForPartialMocking mock = spy(collaborator);
```

上面创建的对象用于演示 final 和 private 方法的模拟。

我们现在将通过设置期望值和调用方法来处理最后一个方法:

```java
when(mock.finalMethod()).thenReturn("I am a final mock method.");
returnValue = mock.finalMethod();
```

部分嘲笑该方法的行为被证明:

```java
Mockito.verify(mock).finalMethod();
```

一个测试验证了调用`finalMethod`方法将返回一个符合预期的值:

```java
assertEquals("I am a final mock method.", returnValue);
```

类似的过程也适用于私有方法。主要的区别是我们不能从测试用例中直接调用这个方法。

基本上，私有方法被同一个类中的其他方法调用。在`CollaboratorForPartialMocking`类中，`privateMethod`方法被`privateMethodCaller`方法调用，我们将使用后者作为委托。

让我们从期望和调用开始:

```java
when(mock, "privateMethod").thenReturn("I am a private mock method.");
returnValue = mock.privateMethodCaller();
```

对私有方法的嘲讽得到了证实:

```java
verifyPrivate(mock).invoke("privateMethod");
```

以下测试确保调用私有方法的返回值与预期值相同:

```java
assertEquals("I am a private mock method. Welcome to the Java world.", returnValue);
```

## 6。结论

本文介绍了`PowerMockito` API，展示了它在解决开发人员在使用 Mockito 框架时遇到的一些问题中的用途。

这些例子和代码片段的实现可以在[链接的 GitHub 项目](https://web.archive.org/web/20220704161927/https://github.com/eugenp/tutorials/tree/master/testing-modules/powermock)中找到。