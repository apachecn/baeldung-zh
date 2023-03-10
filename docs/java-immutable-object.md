# Java 中的不可变对象

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-immutable-object>

## 1。概述

在本教程中，我们将学习什么使一个对象不可变，如何在 Java 中实现不变性，以及这样做的好处。

## 2。什么是不可变对象？

不可变对象是一个**对象，它的内部状态在完全创建**后保持不变。

这意味着一个不可变对象的公共 API 向我们保证，在它的整个生命周期中，它的行为是相同的。

如果我们看一下类`String`，我们可以看到，即使它的 API 似乎用它的`replace`方法为我们提供了一个可变的行为，最初的`String`并没有改变:

```java
String name = "baeldung";
String newName = name.replace("dung", "----");

assertEquals("baeldung", name);
assertEquals("bael----", newName);
```

API 给了我们只读的方法，它不应该包含改变对象内部状态的方法。

## 3。Java 中的`final`关键字

在尝试用 Java 实现不变性之前，我们应该先谈谈`final`关键字。

在 Java 中，**变量在默认情况下是可变的，这意味着我们可以改变它们保存的值**。

通过在声明变量时使用`final`关键字，Java 编译器不会让我们改变该变量的值。相反，它将报告一个编译时错误:

```java
final String name = "baeldung";
name = "bael...";
```

注意`final`只禁止我们改变变量持有的引用，它不保护我们通过使用它的公共 API 来改变它所引用的对象的内部状态:

```java
final List<String> strings = new ArrayList<>();
assertEquals(0, strings.size());
strings.add("baeldung");
assertEquals(0, strings.size());
```

第二个`assertEquals`将失败，因为向列表中添加一个元素会改变它的大小，因此，它不是一个不可变的对象。

## 4。Java 中的不变性

现在我们知道了如何避免改变变量的内容，我们可以用它来构建不可变对象的 API。

构建不可变对象的 API 要求我们保证无论我们如何使用它的 API，它的内部状态都不会改变。

向正确方向前进的一步是在声明其属性时使用`final`:

```java
class Money {
    private final double amount;
    private final Currency currency;

    // ...
}
```

注意，Java 向我们保证`amount`的值不会改变，这是所有原始类型变量的情况。

然而，在我们的例子中，我们只能保证`currency`不会改变，所以**我们必须依靠`Currency`** **API 来保护自己不受变化**的影响。

大多数时候，我们需要对象的属性来保存自定义值，初始化不可变对象内部状态的地方是它的构造函数:

```java
class Money {
    // ...
    public Money(double amount, Currency currency) {
        this.amount = amount;
        this.currency = currency;
    }

    public Currency getCurrency() {
        return currency;
    }

    public double getAmount() {
        return amount;
    }
}
```

正如我们之前说过的，为了满足不可变 API 的要求，我们的`Money`类只有只读方法。

Using the reflection API, we can break immutability and [change immutable objects](https://web.archive.org/web/20221105220145/https://stackoverflow.com/questions/20945049/is-a-java-string-really-immutable). However, reflection violates immutable object's public API, and usually, we should avoid doing this.

## 5。优势

由于不可变对象的内部状态在时间上保持不变，**我们可以在多个线程**之间安全地共享它。

我们也可以自由地使用它，没有一个引用它的对象会注意到任何差异，我们可以说**不可变对象是无副作用的**。

## 6。结论

不可变对象不会及时改变它们的内部状态，它们是线程安全的，没有副作用。由于这些属性，不可变对象在处理多线程环境时也特别有用。

你可以在 GitHub 上找到本文[中使用的例子。](https://web.archive.org/web/20221105220145/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-oop-patterns)