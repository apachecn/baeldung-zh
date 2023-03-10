# 在 Java 中将 boolean 转换为 int

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-boolean-to-int>

## 1.概观

在本教程中，我们将学习如何将一个`boolean`值转换成一个`int`值。首先，我们将看看 Java 如何处理这两种[原始数据类型](/web/20220524050815/https://www.baeldung.com/java-primitives)；然后，我们将探索将布尔值转换为整数的多种方法。

## 2.数据类型

在 Java 中，整数可以由`int `原始数据类型或整数[包装类](/web/20220524050815/https://www.baeldung.com/java-wrapper-classes)来表示。原始数据类型是由[二进制补码](/web/20220524050815/https://www.baeldung.com/cs/two-complement)编码方法表示的 32 位有符号整数。Integer 类充当一个包装器，允许您执行无符号整数运算，以及将整数(原始)值作为对象来处理[泛型](/web/20220524050815/https://www.baeldung.com/java-generics)。

另一方面，`boolean `值在内存中没有特定的大小，但是它默认为操作系统和 [Java 虚拟机(JVM)](/web/20220524050815/https://www.baeldung.com/jvm-vs-jre-vs-jdk) 。类似地，像 Java 中的所有原始数据类型一样，`boolean `有一个布尔包装类，它允许布尔值像对象一样工作。

**我们可以利用两种数据类型(`boolean` 和`int`)的原始值和包装类来执行数据转换。** **假设`true `和`false` 布尔值分别代表 1 和 0，我们有多种方法进行转换。**

## 3.原语`boolean`到`int`

为了将一个原始的 `boolean` 值 转换成一个 `int` ，我们对表达式的条件进行求值，以确定我们要返回的整数:

```java
public int booleanPrimitiveToInt(boolean foo) {
    int bar = 0;
    if (foo) {
        bar = 1;
    }
    return bar;
}
```

我们可以通过使用三元运算符来简化这个函数:

```java
public int booleanPrimitiveToIntTernary(boolean foo) {
    return (foo) ? 1 : 0;
}
```

这种方法使用原始数据类型(`boolean` 和`int)` 来进行转换。因此，当布尔表达式为`true.` 时，我们得到 1，否则，该方法返回 0。

## 4.包装类

使用布尔包装类，我们有两种方法来完成转换:

*   我们可以利用布尔类中的`static `方法。
*   我们可以直接从布尔对象中调用这些方法。

### 4.1.静态方法

布尔类有一个`compare `方法，我们可以如下使用:

```java
public static int booleanObjectToInt(boolean foo) {
    return Boolean.compare(foo, false);
}
```

**回想一下，如果两个参数具有相同的值，静态`compare`方法返回`0 `** **。** **换句话说，当** **`foo`为假时，比较会产生`0`。否则，当第一个参数为`true`且第二个参数为 false 时，函数返回`1 `。**

类似地，我们可以使用相同的静态方法，将第二个参数改为`true`:

```java
public static int booleanObjectToIntInverse(boolean foo) { 
    return Boolean.compare(foo, true) + 1;
}
```

这一次，如果`foo`为真，`compare`方法计算两个相同值的参数，结果是`0`。但是，将结果加 1 将从真布尔变量返回预期的整数值。

### 4.2.布尔类对象

布尔类对象具有我们可以使用的函数，例如`compareTo` :

```java
public static int booleanObjectMethodToInt(Boolean foo) {
    return foo.compareTo(false);
}
```

使用方法`booleanObjectMethodToInt`，我们可以像使用静态方法一样将布尔值转换为整数。类似地，您可以通过将参数更改为`true`并将`1`添加到结果中来应用相反的版本。

## 5.Apache common(Apache 公共)

[Apache Commons](https://web.archive.org/web/20220524050815/https://search.maven.org/search?q=g:org.apache.commons%20AND%20a:commons-lang3) 是一个流行的 Java 开源库，它提供了实用程序类，比如`BooleanUtils`。我们可以在 Maven 中添加这个库作为依赖项，如下所示:

```java
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-lang3</artifactId>
    <version>3.12.0</version>
</dependency> 
```

一旦库在我们的 pom.xml 文件中，我们可以使用`BooleanUtils `类将布尔值转换成整数:

```java
public static int booleanUtilsToInt(Boolean foo) {
    return BooleanUtils.toInteger(foo);
}
```

像示例方法`booleanPrimitiveToIntTernary`一样，在内部，`toInteger `方法执行相同的三元运算符来进行转换。

## 6.结论

在本教程中，我们学习了如何将布尔值转换为整数值。假设`true `转化为`1 `，而`false `转化为`0`，我们探索了不同的实现来实现这种转换。

和往常一样，本文的完整代码示例可以在 GitHub 的[中找到。](https://web.archive.org/web/20220524050815/https://github.com/eugenp/tutorials/tree/master/java-numbers-4)