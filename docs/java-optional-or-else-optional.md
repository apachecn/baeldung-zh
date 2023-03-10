# 选购的不纯功能

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-optional-or-else-optional>

## 1。简介

在某些情况下，如果另一个实例是空的，我们可能想要回退到另一个`Optional`实例。

在本教程中，我们将简要介绍如何做到这一点——这比看起来要难。

关于 Java 可选类的介绍，请看我们的前一篇文章。

## 2。Java 8

**在 Java 8 中，如果第一个为空，没有直接的方法来实现返回一个不同的`Optional`。**

因此，我们可以实现我们自己的自定义方法:

```java
public static <T> Optional<T> or(Optional<T> optional, Optional<T> fallback) {
    return optional.isPresent() ? optional : fallback;
}
```

实际上:

```java
@Test
public void givenOptional_whenValue_thenOptionalGeneralMethod() {
    String name = "Filan Fisteku";
    String missingOptional = "Name not provided";
    Optional<String> optionalString = Optional.ofNullable(name);
    Optional<String> fallbackOptionalString = Optional.ofNullable(missingOptional);

    assertEquals(
      optionalString, 
      Optionals.or(optionalString, fallbackOptionalString));
}

@Test
public void givenEmptyOptional_whenValue_thenOptionalGeneralMethod() {
    Optional<String> optionalString = Optional.empty();
    Optional<String> fallbackOptionalString = Optional.ofNullable("Name not provided");

    assertEquals(
      fallbackOptionalString, 
      Optionals.or(optionalString, fallbackOptionalString));
}
```

### 2.1.懒惰评估

上面的解决方案有一个**严重的缺点——在使用我们定制的`or()`方法**之前，我们需要评估两个`Optional`变量。

想象一下，我们有两个返回`Optional`的方法，都是在后台查询数据库。从性能的角度来看，如果第一个方法已经返回了我们需要的值，那么调用这两个方法是不可接受的。

让我们创建一个简单的`ItemsProvider`类:

```java
public class ItemsProvider {
    public Optional<String> getNail(){
        System.out.println("Returning a nail");
        return Optional.of("nail");
    }

    public Optional<String> getHammer(){
        System.out.println("Returning a hammer");
        return Optional.of("hammer");
    }
}
```

下面是我们如何**链接这些方法并利用惰性评估**:

```java
@Test
public void givenTwoOptionalMethods_whenFirstNonEmpty_thenSecondNotEvaluated() {
    ItemsProvider itemsProvider = new ItemsProvider();

    Optional<String> item = itemsProvider.getNail()
            .map(Optional::of)
            .orElseGet(itemsProvider::getHammer);

    assertEquals(Optional.of("nail"), item);
}
```

上面的测试用例只打印了`“Returning a nail”`。这清楚地表明只执行了`getNail()`方法。

## 3.Java 9

**Java 9 增加了一个`or()`方法，我们可以用它来获得一个`Optional`，或者另一个值，如果那个`Optional`不存在的话**。

让我们通过一个简单的例子来看看这一点:

```java
public static Optional<String> getName(Optional<String> name) {
    return name.or(() -> getCustomMessage());
}
```

我们使用了一个辅助方法来帮助我们的例子:

```java
private static Optional<String> getCustomMessage() {
    return Optional.of("Name not provided");
}
```

我们可以测试它，并进一步了解它是如何工作的。以下测试案例演示了`Optional` 有值时的情况:

```java
@Test
public void givenOptional_whenValue_thenOptional() {
    String name = "Filan Fisteku";
    Optional<String> optionalString = Optional.ofNullable(name);
    assertEquals(optionalString, Optionals.getName(optionalString));
}
```

## 4。使用番石榴

另一种方法是使用 guava 的`Optional` 类的*或()*方法。首先，我们需要在我们的项目中添加`guava` (最新版本可以在[这里找到](https://web.archive.org/web/20221206135926/https://mvnrepository.com/artifact/com.google.guava/guava/19.0)):

```java
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>31.0.1-jre</version>
</dependency> 
```

现在，我们可以继续前面的例子:

```java
public static com.google.common.base.Optional<String> 
  getOptionalGuavaName(com.google.common.base.Optional<String> name) {
    return name.or(getCustomMessageGuava());
}
```

```java
private static com.google.common.base.Optional<String> getCustomMessageGuava() {
    return com.google.common.base.Optional.of("Name not provided");
}
```

正如我们所看到的，它与上面显示的非常相似。但是，它在方法的命名上略有不同，与 JDK 9 中的类`Optional` 的`or()` 方法完全相同。

我们现在可以测试它，类似于上面的例子:

```java
@Test
public void givenGuavaOptional_whenInvoke_thenOptional() {
    String name = "Filan Fisteku";
    Optional<String> stringOptional = Optional.of(name);

    assertEquals(name, Optionals.getOptionalGuavaName(stringOptional));
}
```

```java
@Test
public void givenGuavaOptional_whenNull_thenDefaultText() {
    assertEquals(
      com.google.common.base.Optional.of("Name not provided"), 
      Optionals.getOptionalGuavaName(com.google.common.base.Optional.fromNullable(null)));
}
```

## 5。结论

这是一篇说明如何实现`Optional orElse Optional`功能的快速文章。

这里解释的所有例子的代码，以及更多可以在 GitHub 上找到的[。](https://web.archive.org/web/20221206135926/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-optional)