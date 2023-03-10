# Java 8 中的新特性

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-8-new-features>

[This article is part of a series:](javascript:void(0);)• New Features in Java 8 (current article)[• New Features in Java 9](/web/20221129010510/https://www.baeldung.com/new-java-9)
[• New Features in Java 10](/web/20221129010510/https://www.baeldung.com/java-10-overview)
[• New Features in Java 11](/web/20221129010510/https://www.baeldung.com/java-11-new-features)
[• New Features in Java 12](/web/20221129010510/https://www.baeldung.com/java-12-new-features)
[• New Features in Java 13](/web/20221129010510/https://www.baeldung.com/java-13-new-features)
[• New Features in Java 14](/web/20221129010510/https://www.baeldung.com/java-14-new-features)
[• What's New in Java 15](/web/20221129010510/https://www.baeldung.com/java-15-new)
[• New Features in Java 16](/web/20221129010510/https://www.baeldung.com/java-16-new-features)
[• New Features in Java 17](/web/20221129010510/https://www.baeldung.com/java-17-new-features)

## 1。概述

在本教程中，我们将快速浏览 Java 8 中一些最有趣的新特性。

我们将讨论接口默认和静态方法、方法引用和可选方法。

我们已经介绍了 Java 8 版本的一些特性——流 API 、 [lambda 表达式和函数接口](/web/20221129010510/https://www.baeldung.com/java-8-lambda-expressions-tips)——因为它们是值得单独研究的综合主题。

## 2。接口默认和静态方法

在 Java 8 之前，接口只能有公共抽象方法。如果不强制所有实现类创建新方法的实现，就不可能向现有接口添加新功能，也不可能用实现来创建接口方法。

从 Java 8 开始，接口可以有`**static**`和 **`default`** 方法，尽管这些方法是在接口中声明的，但都有定义的行为。

### 2.1。静态方法

考虑这个接口的方法(让我们称这个接口为`Vehicle`):

```java
static String producer() {
    return "N&F; Vehicles";
}
```

静态的`producer()`方法只能在接口内部使用。它不能被实现类重写。

要在接口外部调用它，应该使用静态方法调用的标准方法:

```java
String producer = Vehicle.producer();
```

### 2.2。默认方法

默认方法是使用新的 **`default` 关键字声明的。**这些可以通过实现类的实例来访问，并且可以被覆盖。

让我们给我们的`Vehicle` 接口添加一个`default`方法，它也将调用这个接口的`static`方法:

```java
default String getOverview() {
    return "ATV made by " + producer();
}
```

假设这个接口是由类`VehicleImpl`实现的。

为了执行`default`方法，应该创建这个类的一个实例:

```java
Vehicle vehicle = new VehicleImpl();
String overview = vehicle.getOverview();
```

## 3。方法引用

对于只调用现有方法的 lambda 表达式，方法引用可以用作更短、可读性更好的替代方法。方法引用有四种变体。

### 3.1。对静态方法的引用

对静态方法的引用包含语法 **`ContainingClass::methodName`。**

我们将尝试借助 Stream API 对`List<String>`中的所有空字符串进行计数:

```java
boolean isReal = list.stream().anyMatch(u -> User.isRealUser(u));
```

让我们仔细看看`anyMatch()`方法中的 lambda 表达式。它只是调用了`User` 类的静态方法`isRealUser(User user)`。

因此，可以用对静态方法的引用来替换它:

```java
boolean isReal = list.stream().anyMatch(User::isRealUser);
```

这种类型的代码看起来信息量更大。

### 3.2。引用一个实例方法

对实例方法的引用包含语法 **`containingInstance::methodName`。**

以下代码调用类型为`User`的方法`isLegalName(String string)` ，该方法验证输入参数:

```java
User user = new User();
boolean isLegalName = list.stream().anyMatch(user::isLegalName); 
```

### 3.3。对特定类型对象的实例方法的引用

这个引用方法采用语法 **`ContainingType::methodName`。**

让我们看一个例子:

```java
long count = list.stream().filter(String::isEmpty).count();
```

### 3.4。对构造函数的引用

对构造函数的引用采用语法 **`ClassName::new`。**

由于 Java 中的构造函数是一个特殊的方法，方法引用也可以应用于它，借助 **`new`** 作为方法名:

```java
Stream<User> stream = list.stream().map(User::new);
```

## 4。 `**Optional<T>**`

在 Java 8 之前，开发人员必须仔细验证他们引用的值，因为有可能抛出`NullPointerException (NPE)`。所有这些检查都需要一个非常烦人且容易出错的样板代码。

Java 8 `Optional<T>`类可以帮助处理有可能获得`NPE`的情况。它作为类型为`T`的对象的容器。如果这个值不是一个`null`，它可以返回这个对象的值。当这个容器中的值是`null`时，它允许做一些预定义的动作，而不是抛出`NPE`。

### 4.1。`Optional<T>` 【T2 的创造】

可以在静态方法的帮助下创建`Optional`类的实例。

让我们看看如何返回一个空的`Optional`:

```java
Optional<String> optional = Optional.empty();
```

接下来，我们返回一个包含非空值的`Optional`:

```java
String str = "value";
Optional<String> optional = Optional.of(str);
```

最后，下面是如何返回一个带有特定值的`Optional`或者一个空的`Optional`(如果参数是`null`):

```java
Optional<String> optional = Optional.ofNullable(getString());
```

### 4.2。 `Optional<T> Usage`

假设我们期望得到一个`List<String>`，对于`null`，我们希望用一个`ArrayList<String>`的新实例来替换它。

对于 Java 8 之前的代码，我们需要这样做:

```java
List<String> list = getList();
List<String> listOpt = list != null ? list : new ArrayList<>();
```

使用 Java 8，可以用更短的代码实现相同的功能:

```java
List<String> listOpt = getList().orElseGet(() -> new ArrayList<>());
```

当我们需要以旧的方式访问某个对象的字段时，甚至会有更多的样板代码。

假设我们有一个类型为`User`的对象，它有一个类型为*的地址字段*和一个类型为`String`的字段 s *treet* ，我们需要返回一个`street` 字段的值(如果有的话)或者一个默认值(如果`street` 是`null`:

```java
User user = getUser();
if (user != null) {
    Address address = user.getAddress();
    if (address != null) {
        String street = address.getStreet();
        if (street != null) {
            return street;
        }
    }
}
return "not specified";
```

这可以用`Optional`来简化:

```java
Optional<User> user = Optional.ofNullable(getUser());
String result = user
  .map(User::getAddress)
  .map(Address::getStreet)
  .orElse("not specified");
```

在这个例子中，我们使用了`map()`方法来将调用`getAdress()` 的结果转换为`Optional<Address>` ，将`getStreet()`的结果转换为`Optional<String>`。如果这些方法中的任何一个返回了`null`，那么`map()`方法将返回一个空的`Optional`。

现在想象我们的 getters 返回`Optional<T>`。

在这种情况下，我们应该使用`flatMap()` 方法，而不是`map()`:

```java
Optional<OptionalUser> optionalUser = Optional.ofNullable(getOptionalUser());
String result = optionalUser
  .flatMap(OptionalUser::getAddress)
  .flatMap(OptionalAddress::getStreet)
  .orElse("not specified");
```

`Optional`的另一个用例是用另一个异常改变`NPE` 。

因此，正如我们之前所做的那样，让我们尝试用 Java 8 之前的风格来做这件事:

```java
String value = null;
String result = "";
try {
    result = value.toUpperCase();
} catch (NullPointerException exception) {
    throw new CustomException();
}
```

如果我们使用`Optional<String>`，答案会更易读、更简单:

```java
String value = null;
Optional<String> valueOpt = Optional.ofNullable(value);
String result = valueOpt.orElseThrow(CustomException::new).toUpperCase();
```

请注意，如何在我们的应用程序中使用`Optional`以及出于什么目的是一个严肃且有争议的设计决策，对其利弊的解释超出了本文的范围。但是有很多有趣的文章专门讨论这个问题。[这一个](https://web.archive.org/web/20221129010510/http://blog.joda.org/2014/11/optional-in-java-se-8.html)和[这一个](https://web.archive.org/web/20221129010510/http://blog.joda.org/2015/08/java-se-8-optional-pragmatic-approach.html)可能非常有助于深入挖掘。

## 5。结论

在本文中，我们简要讨论了 Java 8 中一些有趣的新特性。

当然，在许多 Java 8 JDK 包和类中还有许多其他的添加和改进。

但是本文中展示的信息是探索和学习这些新特性的良好起点。

最后，本文的所有源代码都可以在 GitHub 上找到。

Next **»**[New Features in Java 9](/web/20221129010510/https://www.baeldung.com/new-java-9)