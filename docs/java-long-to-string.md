# 在 Java 中将 Long 转换为 String

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-long-to-string>

## 1.概观

在这个简短的教程中，我们将学习如何在 Java 中将`Long`转换成`String`**。**

## 2.使用`Long.toString()`

例如，假设我们有两个类型为`long`和`Long`的变量(一个是原始类型，另一个是引用类型):

```java
long l = 10L;
Long obj = 15L;
```

我们可以简单地使用`Long`类的`toString()`方法将它们转换成`String`:

```java
String str1 = Long.toString(l);
String str2 = Long.toString(obj);

System.out.println(str1);
System.out.println(str2);
```

输出将如下所示:

```java
10
15
```

**如果我们的`obj`对象是`null`，我们会得到一个`NullPointerException`** 。

## 3.使用`String.valueOf()`

我们可以使用`String`类的`valueOf()`方法来实现同样的目标:

```java
String str1 = String.valueOf(l);
String str2 = String.valueOf(obj);
```

当`obj`为`null`时，该方法会将`str2`设置为“空”，而不是抛出一个`NullPointerException`。

## 4.使用`String.format()`

除了`String`类的`valueOf()`方法，我们还可以使用`format()`方法:

```java
String str1 = String.format("%d", l);
String str2 = String.format("%d", obj);
```

如果`obj`为`null`，则`str2`也将为“空”。

## 5.使用`Long`对象的`toString()`方法

我们的`obj`对象可以使用它的`toString()`方法来获得`String`表示:

```java
String str = obj.toString();
```

当然，如果`obj`是`null`，我们会得到一个`NullPointerException`。

## 6.使用+运算符

我们可以简单地使用+运算符和空的`String`来获得相同的结果:

```java
String str1 = "" + l;
String str2 = "" + obj;
```

如果`obj`为`null`，则`str2`将为“空”。

## 7.使用`StringBuilder`或`StringBuffer`

[`StringBuilder``StringBuffer`](/web/20221208143926/https://www.baeldung.com/java-string-builder-string-buffer)对象可用于将`Long`转换为`String`:

```java
String str1 = new StringBuilder().append(l).toString();
String str2 = new StringBuilder().append(obj).toString();
```

如果`obj`为`null`，则`str2`将为“空”。

## 8.使用`DecimalFormat`

最后，我们可以使用`format()`方法得到一个 [`DecimalFormat`](/web/20221208143926/https://www.baeldung.com/java-decimalformat) 对象:

```java
String str1 = new DecimalFormat("#").format(l);
String str2 = new DecimalFormat("#").format(obj);
```

小心因为**如果`obj`是`null`，我们会得到一个** `**IllegalArgumentException**.`

## 9.结论

总之，我们已经学习了在 Java 中将`Long`转换为`String`的不同方法**。选择使用哪种方法取决于我们，但通常最好使用简洁且不抛出异常的方法。**