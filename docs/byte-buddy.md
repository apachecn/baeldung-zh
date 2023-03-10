# 字节伙伴指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/byte-buddy>

## 1。概述

简单来说， [ByteBuddy](https://web.archive.org/web/20220820050723/http://bytebuddy.net/#/) 是一个在运行时动态生成 Java 类的库。

在这篇重点文章中，我们将使用框架来操作现有的类，按需创建新的类，甚至拦截方法调用。

## 2。依赖性

让我们首先将依赖项添加到项目中。对于基于 Maven 的项目，我们需要将这种依赖性添加到我们的`pom.xml`:

```java
<dependency>
    <groupId>net.bytebuddy</groupId>
    <artifactId>byte-buddy</artifactId>
    <version>1.12.13</version>
</dependency>
```

对于基于 Gradle 的项目，我们需要将相同的工件添加到我们的`build.gradle`文件中:

```java
compile net.bytebuddy:byte-buddy:1.12.13
```

最新版本可以在 [Maven Central](https://web.archive.org/web/20220820050723/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22net.bytebuddy%22%20AND%20a%3A%22byte-buddy%22) 上找到。

## 3。运行时创建 Java 类

让我们从通过子类化一个现有的类来创建一个动态类开始。我们来看看经典的`Hello World`项目。

在这个例子中，我们创建了一个类型(`Class`)，它是`Object.class` 的子类，并覆盖了`toString()`方法:

```java
DynamicType.Unloaded unloadedType = new ByteBuddy()
  .subclass(Object.class)
  .method(ElementMatchers.isToString())
  .intercept(FixedValue.value("Hello World ByteBuddy!"))
  .make();
```

我们刚刚做的是创建一个`ByteBuddy.` 的实例，然后，我们使用`subclass() API` 来扩展`Object.class`，并且我们使用`ElementMatchers`来选择超类`Object.class`的`toString()`。

最后，使用`intercept()`方法，我们提供了`toString()` 的实现并返回一个固定值。

`make()` 方法触发新类的生成。

此时，我们的类已经创建好了，但是还没有加载到 JVM 中。它由一个`DynamicType.Unloaded`的实例来表示，这个实例是生成类型的二进制形式。

因此，在使用之前，我们需要将生成的类加载到 JVM 中:

```java
Class<?> dynamicType = unloadedType.load(getClass()
  .getClassLoader())
  .getLoaded();
```

现在，我们可以实例化`dynamicType`并对其调用`toString()`方法:

```java
assertEquals(
  dynamicType.newInstance().toString(), "Hello World ByteBuddy!");
```

请注意，调用`dynamicType.toString()` 将不起作用，因为这只会调用`ByteBuddy.class`的`toString()`实现。

`newInstance()`是一个 Java 反射方法，它创建一个由这个`ByteBuddy`对象表示的类型的新实例；类似于在无参数构造函数中使用`new`关键字。

到目前为止，我们只能在动态类型的超类中覆盖一个方法，并返回我们自己的固定值。在接下来的部分中，我们将看看如何用定制逻辑定义我们的方法。

## 4。方法委托和自定义逻辑

在前面的例子中，我们从`toString()`方法返回一个固定值。

实际上，应用程序需要比这更复杂的逻辑。为动态类型提供定制逻辑的一种有效方法是委托方法调用。

让我们创建一个动态类型，它继承了具有`sayHelloFoo()`方法的`Foo.class` :

```java
public String sayHelloFoo() { 
    return "Hello in Foo!"; 
}
```

此外，让我们用与`sayHelloFoo()`相同的签名和返回类型的静态`sayHelloBar()`创建另一个类`Bar` :

```java
public static String sayHelloBar() { 
    return "Holla in Bar!"; 
}
```

现在，让我们使用`ByteBuddy`的 DSL 将所有对`sayHelloFoo()`的调用委托给`sayHelloBar()`。这允许我们在运行时为新创建的类提供用纯 Java 编写的定制逻辑:

```java
String r = new ByteBuddy()
  .subclass(Foo.class)
  .method(named("sayHelloFoo")
    .and(isDeclaredBy(Foo.class)
    .and(returns(String.class))))        
  .intercept(MethodDelegation.to(Bar.class))
  .make()
  .load(getClass().getClassLoader())
  .getLoaded()
  .newInstance()
  .sayHelloFoo();

assertEquals(r, Bar.sayHelloBar());
```

调用`sayHelloFoo()`将相应地调用`sayHelloBar()`。

**怎么知道要调用`Bar.class`中的哪个方法？**它根据方法签名、返回类型、方法名和注释选择一个匹配的方法。

`sayHelloFoo()`和`sayHelloBar()`方法没有相同的名称，但是它们有相同的方法签名和返回类型。

如果在`Bar.class`中有多个可调用的方法具有匹配的签名和返回类型，我们可以使用`@BindingPriority`注释来解决歧义。

`@BindingPriority`采用整数参数——整数值越高，调用特定实现的优先级越高。因此，在下面的代码片段中，`sayHelloBar()`将优先于`sayBar()`:

```java
@BindingPriority(3)
public static String sayHelloBar() { 
    return "Holla in Bar!"; 
}

@BindingPriority(2)
public static String sayBar() { 
    return "bar"; 
}
```

## 5。方法和字段定义

我们已经能够覆盖在动态类型的超类中声明的方法。让我们进一步向我们的类添加一个新方法(和一个字段)。

我们将使用 Java 反射来调用动态创建的方法:

```java
Class<?> type = new ByteBuddy()
  .subclass(Object.class)
  .name("MyClassName")
  .defineMethod("custom", String.class, Modifier.PUBLIC)
  .intercept(MethodDelegation.to(Bar.class))
  .defineField("x", String.class, Modifier.PUBLIC)
  .make()
  .load(
    getClass().getClassLoader(), ClassLoadingStrategy.Default.WRAPPER)
  .getLoaded();

Method m = type.getDeclaredMethod("custom", null);
assertEquals(m.invoke(type.newInstance()), Bar.sayHelloBar());
assertNotNull(type.getDeclaredField("x"));
```

我们创建了一个名为`MyClassName`的类，它是`Object.class`的子类。然后我们定义一个方法，`custom,`，它返回一个`String`，并有一个`public`访问修饰符。

就像我们在前面的例子中所做的一样，我们通过拦截对它的调用并将它们委托给我们在本教程前面创建的`Bar.class`来实现我们的方法。

## 6。重新定义现有的类

虽然我们一直在使用动态创建的类，但是我们也可以使用已经加载的类。这可以通过重新定义(或重新定义)现有的类并使用`ByteBuddyAgent`将它们重新加载到 JVM 中来实现。

首先，让我们将`ByteBuddyAgent`添加到我们的`pom.xml`中:

```java
<dependency>
    <groupId>net.bytebuddy</groupId>
    <artifactId>byte-buddy-agent</artifactId>
    <version>1.7.1</version>
</dependency>
```

最新版本可以在这里找到[。](https://web.archive.org/web/20220820050723/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22net.bytebuddy%22%20AND%20a%3A%22byte-buddy-agent%22)

现在，让我们重新定义之前在`Foo.class` 中创建的`sayHelloFoo()`方法:

```java
ByteBuddyAgent.install();
new ByteBuddy()
  .redefine(Foo.class)
  .method(named("sayHelloFoo"))
  .intercept(FixedValue.value("Hello Foo Redefined"))
  .make()
  .load(
    Foo.class.getClassLoader(), 
    ClassReloadingStrategy.fromInstalledAgent());

Foo f = new Foo();

assertEquals(f.sayHelloFoo(), "Hello Foo Redefined");
```

## 7。结论

在这篇精心制作的指南中，我们广泛地研究了`ByteBuddy`库的功能以及如何使用它来高效地创建动态类。

它的[文档](https://web.archive.org/web/20220820050723/http://bytebuddy.net/#/tutorial)对该库的内部运作和其他方面提供了深入的解释。

和往常一样，本教程的完整代码片段可以在 Github 上找到[。](https://web.archive.org/web/20220820050723/https://github.com/eugenp/tutorials/tree/master/libraries-5)