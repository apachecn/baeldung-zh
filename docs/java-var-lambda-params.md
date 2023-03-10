# Lambda 参数的 Java 11 局部变量语法

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-var-lambda-params>

## 1。简介

lambda 参数的局部变量语法是 Java 11 中唯一引入的语言特性。在本教程中，我们将探索和使用这一新功能。

## 2。Lambda 参数的局部变量语法

Java 10 中引入的关键特性之一是[局部变量类型推断](/web/20221206174558/https://www.baeldung.com/java-10-local-variable-type-inference)。它允许使用`var`作为局部变量的类型，而不是实际类型。编译器根据分配给变量的值来推断类型。

然而，我们不能将这个特性与 lambda 参数一起使用。例如，考虑下面的λ。这里我们明确指定了参数的类型:

```java
(String s1, String s2) -> s1 + s2
```

我们可以跳过参数类型，将 lambda 重写为:

```java
(s1, s2) -> s1 + s2
```

甚至 Java 8 也支持这一点。在 Java 10 中对此的逻辑扩展是:

```java
(var s1, var s2) -> s1 + s2
```

但是，Java 10 不支持这一点。

Java 11 通过支持上述语法解决了这个问题。**这使得`var`在局部变量和 lambda 参数中的用法一致。
**

## 3。利益

当我们可以简单地跳过类型时，为什么我们要对 lambda 参数使用`var`?

一致性的一个好处是修饰符可以应用于局部变量和 lambda 形式而不失简洁。例如，常见的修饰符是类型注释:

```java
(@Nonnull var s1, @Nullable var s2) -> s1 + s2
```

如果不指定类型，我们就不能使用这样的注释。

## 4。局限性

在 lambda 中使用`var`有一些限制。

例如，我们不能对某些参数使用`var`而对其他参数跳过:

```java
(var s1, s2) -> s1 + s2
```

类似地，我们不能将`var`与显式类型混合使用:

```java
(var s1, String s2) -> s1 + s2
```

最后，尽管我们可以跳过单参数 lambda 中的括号:

```java
s1 -> s1.toUpperCase()
```

我们不能在使用`var`时跳过它们:

```java
var s1 -> s1.toUpperCase()
```

以上三种用法都会导致编译错误。

## 5。结论

在这篇简短的文章中，我们探索了 Java 11 中这个很酷的新特性，并了解了如何为 lambda 参数使用局部变量语法。

像往常一样，这些例子可以在 GitHub 的[上找到。](https://web.archive.org/web/20221206174558/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-11)