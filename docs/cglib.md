# cglib 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/cglib>

## 1。概述

在本文中，我们将关注`[cglib](https://web.archive.org/web/20221011084927/https://github.com/cglib/cglib)`(代码生成库)库。它是一个字节插装库，用于许多 Java 框架，如`Hibernate`或`Spring`。字节码插装允许在程序的编译阶段之后操作或创建类。

## 2。Maven 依赖关系

要在您的项目中使用`cglib`，只需添加一个 Maven 依赖项(最新版本可以在[这里找到](https://web.archive.org/web/20221011084927/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22cglib%22%20AND%20a%3A%22cglib%22)):

```java
<dependency>
    <groupId>cglib</groupId>
    <artifactId>cglib</artifactId>
    <version>3.2.4</version>
</dependency>
```

## 3。Cglib

Java 中的类是在运行时动态加载的。利用 Java 语言的这一特性，可以向已经运行的 Java 程序中添加新的类。

`Hibernate`使用 cglib 生成动态代理。例如，它不会返回存储在数据库中的完整对象，但它会返回存储类的已检测版本，该版本根据需要从数据库中缓慢加载值。

流行的嘲讽框架，如`Mockito,`使用`cglib`作为嘲讽方法。mock 是一个插装类，其中的方法被空的实现所取代。

我们将会看到`cglib.`中最有用的构造

## 4。使用`cglib`实现代理

假设我们有一个`PersonService` 类，它有两个方法:

```java
public class PersonService {
    public String sayHello(String name) {
        return "Hello " + name;
    }

    public Integer lengthOfName(String name) {
        return name.length();
    }
}
```

注意，第一个方法返回`String` ，第二个方法返回`Integer.`

### 4.1。返回相同的值

我们想要创建一个简单的代理类来拦截对`sayHello()` 方法的调用。 [`Enhancer`](https://web.archive.org/web/20221011084927/http://cglib.sourceforge.net/apidocs/net/sf/cglib/Enhancer.html) 类允许我们通过使用来自`Enhancer` 类的`setSuperclass()` 方法动态扩展`PersonService` 类来创建代理:

```java
Enhancer enhancer = new Enhancer();
enhancer.setSuperclass(PersonService.class);
enhancer.setCallback((FixedValue) () -> "Hello Tom!");
PersonService proxy = (PersonService) enhancer.create();

String res = proxy.sayHello(null);

assertEquals("Hello Tom!", res);
```

`[FixedValue](https://web.archive.org/web/20221011084927/http://cglib.sourceforge.net/apidocs/net/sf/cglib/proxy/FixedValue.html)` 是一个回调接口，它简单地从代理方法返回值。在代理上执行`sayHello()` 方法返回了代理方法中指定的值。

### 4.2。根据方法签名返回值

我们的代理的第一个版本有一些缺点，因为我们不能决定代理应该拦截哪个方法，以及应该从超类调用哪个方法。我们可以使用一个`[MethodInterceptor](https://web.archive.org/web/20221011084927/http://cglib.sourceforge.net/apidocs/net/sf/cglib/proxy/MethodInterceptor.html)` 接口来拦截对代理的所有调用，并决定是否要进行特定的调用或执行超类中的方法:

```java
Enhancer enhancer = new Enhancer();
enhancer.setSuperclass(PersonService.class);
enhancer.setCallback((MethodInterceptor) (obj, method, args, proxy) -> {
    if (method.getDeclaringClass() != Object.class && method.getReturnType() == String.class) {
        return "Hello Tom!";
    } else {
        return proxy.invokeSuper(obj, args);
    }
});

PersonService proxy = (PersonService) enhancer.create();

assertEquals("Hello Tom!", proxy.sayHello(null));
int lengthOfName = proxy.lengthOfName("Mary");

assertEquals(4, lengthOfName);
```

在这个例子中，当方法签名不是来自于`Object` 类时，我们拦截所有的调用，也就是说`toString()` 或`hashCode()` 方法不会被拦截。除此之外，我们只拦截来自返回`String`的`PersonService` 的方法。对一个`lengthOfName()`方法的调用不会被拦截，因为它的返回类型是一个`Integer.`

## 5。Bean 创建者

来自`cglib` 的另一个有用的构造是一个 [`BeanGenerator`](https://web.archive.org/web/20221011084927/http://cglib.sourceforge.net/apidocs/net/sf/cglib/beans/BeanGenerator.html) 类。它允许我们动态地创建 beans，并用 setter 和 getter 方法添加字段。代码生成工具可以使用它来生成简单的 POJO 对象:

```java
BeanGenerator beanGenerator = new BeanGenerator();

beanGenerator.addProperty("name", String.class);
Object myBean = beanGenerator.create();
Method setter = myBean.getClass().getMethod("setName", String.class);
setter.invoke(myBean, "some string value set by a cglib");

Method getter = myBean.getClass().getMethod("getName");
assertEquals("some string value set by a cglib", getter.invoke(myBean));
```

## 6。创建混音

`mixin`是一个允许将多个对象组合成一个的构造。我们可以包含几个类的行为，并将该行为作为单个类或接口公开。`cglib`混合允许将几个对象组合成一个对象。然而，为了做到这一点，mixin 中包含的所有对象都必须有接口支持。

假设我们想要创建两个接口的混合。我们需要定义接口及其实现:

```java
public interface Interface1 {
    String first();
}

public interface Interface2 {
    String second();
}

public class Class1 implements Interface1 {
    @Override
    public String first() {
        return "first behaviour";
    }
}

public class Class2 implements Interface2 {
    @Override
    public String second() {
        return "second behaviour";
    }
} 
```

为了组合`Interface1`和`Interface2` 的实现，我们需要创建一个扩展它们的接口:

```java
public interface MixinInterface extends Interface1, Interface2 { }
```

通过使用 [`Mixin`](https://web.archive.org/web/20221011084927/http://cglib.sourceforge.net/apidocs/net/sf/cglib/proxy/Mixin.html) 类中的`create()`方法，我们可以将`Class1`和`Class2`的行为包含到`MixinInterface:`中

```java
Mixin mixin = Mixin.create(
  new Class[]{ Interface1.class, Interface2.class, MixinInterface.class },
  new Object[]{ new Class1(), new Class2() }
);
MixinInterface mixinDelegate = (MixinInterface) mixin;

assertEquals("first behaviour", mixinDelegate.first());
assertEquals("second behaviour", mixinDelegate.second());
```

调用`mixinDelegate` 上的方法将调用`Class1`和`Class2.`的实现

## 7。结论

在本文中，我们正在研究`cglib` 及其最有用的构造。我们使用一个`Enhancer` 类创建了一个代理。我们使用了一个`BeanCreator` ，最后，我们创建了一个包含其他类行为的`Mixin` 。

Spring 框架广泛使用 Cglib。Spring 使用 cglib 代理的一个例子是向方法调用添加安全约束。Spring security 不是直接调用一个方法，而是首先检查(通过代理)一个指定的安全检查是否通过，并且只有在验证成功时才委托给实际的方法。在本文中，我们看到了如何为我们自己的目的创建这样的代理。

所有这些示例和代码片段的实现都可以在 [GitHub 项目](https://web.archive.org/web/20221011084927/https://github.com/eugenp/tutorials/tree/master/libraries/src/test/java/com/baeldung/cglib/proxy)中找到——这是一个 Maven 项目，因此它应该很容易导入和运行。