# Spring 断言语句

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-assert>

## 1。概述

在本教程中，我们将关注并描述 [Spring `Assert`](https://web.archive.org/web/20220524033151/https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/util/Assert.html) 类的用途，并演示如何使用它。

## 2。`Assert`班的目的

Spring `Assert`类帮助我们验证参数。**通过使用`Assert`类的方法，我们可以写出我们期望为真的假设。如果不满足，就会抛出运行时异常。**

每个`Assert`的方法都可以与 Java [`assert`](https://web.archive.org/web/20220524033151/https://docs.oracle.com/javase/8/docs/technotes/guides/language/assert.html) 语句进行比较。如果条件失败，Java `assert`语句在运行时抛出一个`Error`。有趣的是，这些断言可以被禁用。

以下是 Spring `Assert`方法的一些特点:

*   `Assert`的方法是静态的
*   他们投掷`IllegalArgumentException`或`IllegalStateException`
*   第一个参数通常是用于验证的参数或要检查的逻辑条件
*   最后一个参数通常是在验证失败时显示的异常消息
*   该消息可以作为`String`参数或`Supplier<String> `参数传递

还要注意，尽管名称相似，Spring 断言与 [JUnit](https://web.archive.org/web/20220524033151/https://junit.org/) 和其他测试框架的断言没有任何共同之处。Spring 断言不是为了测试，而是为了调试。

## 3。使用示例

让我们用公共方法`drive()`定义一个`Car`类:

```java
public class Car {
    private String state = "stop";

    public void drive(int speed) {
        Assert.isTrue(speed > 0, "speed must be positive");
        this.state = "drive";
        // ...
    }
}
```

我们可以看到速度为什么一定是正数。上面的行是检查条件并在条件失败时抛出异常的一种快捷方式:

```java
if (!(speed > 0)) {
    throw new IllegalArgumentException("speed must be positive");
}
```

每个`Assert`的公共方法大致包含以下代码——一个带有运行时异常的条件块，应用程序不会从该异常中恢复。

如果我们试图用一个负参数调用`drive()`方法，将会抛出一个`IllegalArgumentException`异常:

```java
Exception in thread "main" java.lang.IllegalArgumentException: speed must be positive
```

## 4。逻辑断言

### 4.1。T0

这个断言在上面已经讨论过了。它接受一个`boolean`条件，并在条件为假时抛出一个`IllegalArgumentException`。

### 4.2。 `**state()**`

**`state()`方法与`isTrue()`具有相同的签名，但是抛出了`IllegalStateException.`**

顾名思义，它应该在由于对象的非法状态而不能继续使用该方法时使用。

想象一下，如果汽车在运行，我们不能调用`fuel()`方法。让我们在这种情况下使用`state()`断言:

```java
public void fuel() {
    Assert.state(this.state.equals("stop"), "car must be stopped");
    // ...
}
```

当然，我们可以使用逻辑断言来验证一切。但是为了更好的可读性，我们可以使用额外的断言使我们的代码更有表现力。

## 5。对象和类型声明 ns

### 5.1。 `**notNull()**`

我们可以通过使用`notNull()`方法假设一个对象不是`null`:

```java
public void сhangeOil(String oil) {
    Assert.notNull(oil, "oil mustn't be null");
    // ...
}
```

### 5.2。`isNull()`

另一方面，我们可以使用`isNull()`方法检查对象是否为`null`:

```java
public void replaceBattery(CarBattery carBattery) {
    Assert.isNull(
      carBattery.getCharge(), 
      "to replace battery the charge must be null");
    // ...
}
```

### 5.3。`isInstanceOf()`

要检查一个对象是否是另一个特定类型对象的实例，我们可以使用`isInstanceOf()`方法:

```java
public void сhangeEngine(Engine engine) {
    Assert.isInstanceOf(ToyotaEngine.class, engine);
    // ...
}
```

在我们的例子中，检查成功通过，因为`ToyotaEngine`是`Engine.`的子类

### 5.4。`isAssignable()`

要检查类型，我们可以使用 `Assert.isAssignable()`:

```java
public void repairEngine(Engine engine) {
    Assert.isAssignable(Engine.class, ToyotaEngine.class);
    // ...
}
```

最近的两个断言代表了一种`is-a`关系。

## 6.文本断言

文本断言用于对`String`参数进行检查。

### 6.1。`hasLength()`

我们可以使用`hasLength()`方法检查`String`是否为空，这意味着它至少包含一个空格:

```java
public void startWithHasLength(String key) {
    Assert.hasLength(key, "key must not be null and must not the empty");
    // ...
}
```

### 6.2。`hasText()`

我们可以通过使用`hasText()`方法来强化条件并检查`String`是否包含至少一个非空白字符:

```java
public void startWithHasText(String key) {
    Assert.hasText(
      key, 
      "key must not be null and must contain at least one non-whitespace  character");
    // ...
}
```

### 6.3。`doesNotContain` `()`

我们可以通过使用`doesNotContain()`方法来确定`String`参数是否不包含特定的子串:

```java
public void startWithNotContain(String key) {
    Assert.doesNotContain(key, "123", "key mustn't contain 123");
    // ...
}
```

## 7.集合和映射断言

### 7.1。`notEmpty()`进行收藏

顾名思义，`notEmpty()`方法断言集合不是空的，这意味着它不是`null`并且包含至少一个元素:

```java
public void repair(Collection<String> repairParts) {
    Assert.notEmpty(
      repairParts, 
      "collection of repairParts mustn't be empty");
    // ...
}
```

### 7.2。`notEmpty()`供图

相同的方法被重载用于映射，我们可以检查映射是否不为空并且包含至少一个条目:

```java
public void repair(Map<String, String> repairParts) {
    Assert.notEmpty(
      repairParts, 
      "map of repairParts mustn't be empty");
    // ...
}
```

## 8.数组断言

### 8.1。`notEmpty()`对于数组

最后，我们可以通过使用`notEmpty()`方法来检查数组是否不为空并且包含至少一个元素:

```java
public void repair(String[] repairParts) {
    Assert.notEmpty(
      repairParts, 
      "array of repairParts mustn't be empty");
    // ...
}
```

### 8.2。`noNullElements()`

我们可以通过使用`noNullElements()` 方法来验证数组不包含`null`元素:

```java
public void repairWithNoNull(String[] repairParts) {
    Assert.noNullElements(
      repairParts, 
      "array of repairParts mustn't contain null elements");
    // ...
}
```

注意，如果数组是空的，只要数组中没有`null`元素，这个检查仍然通过。

## 9.结论

在本文中，我们探索了`Assert`类。这个类在 Spring 框架中被广泛使用，但是我们可以很容易地利用它编写更健壮、更有表现力的代码。

一如既往，本文的完整代码可以在 [GitHub 项目](https://web.archive.org/web/20220524033151/https://github.com/eugenp/tutorials/tree/master/spring-5)中找到。