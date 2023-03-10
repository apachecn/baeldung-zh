# 在 Java 中比较字符串

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-compare-strings>

## 1。概述

在本文中，我们将讨论 Java 中比较`Strings`的不同方式。

由于`String`是 Java 中最常用的数据类型之一，这自然是一个非常常用的操作。

## 2。`String`与`String`级的比较

### 2.1。使用`“==”`比较运算符

使用“==”运算符比较文本值是 Java 初学者最常犯的错误之一。这是不正确的，因为 **`“==”`只检查两个`Strings`** `,`含义的引用是否相等，它们是否引用同一个对象。

让我们来看一个这种行为的例子:

```java
String string1 = "using comparison operator";
String string2 = "using comparison operator";
String string3 = new String("using comparison operator");

assertThat(string1 == string2).isTrue();
assertThat(string1 == string3).isFalse();
```

在上面的例子中，第一个断言为真，因为两个变量指向同一个`String`文字。

另一方面，第二个断言为假，因为`string1`是用文字创建的，而`string3`是用`new`操作符创建的——因此它们引用了不同的对象。

### 2.2。使用`equals()`

`String`类覆盖了从`Object.` **继承的`equals()`。该方法逐个字符地比较两个`Strings`，忽略它们的地址。**

如果它们长度相同，字符顺序相同，则认为它们相等:

```java
String string1 = "using equals method";
String string2 = "using equals method";

String string3 = "using EQUALS method";
String string4 = new String("using equals method");

assertThat(string1.equals(string2)).isTrue();
assertThat(string1.equals(string4)).isTrue();

assertThat(string1.equals(null)).isFalse();
assertThat(string1.equals(string3)).isFalse();
```

在这个例子中，`string1, string2,`和`string4`变量是相等的，因为不管它们的地址如何，它们都具有相同的大小写和值。

对于`string3`,该方法返回`false,`,因为它区分大小写。

同样，如果两个字符串中的任何一个是`null`，那么该方法返回`false.`

### 2.3。使用`equalsIgnoreCase()`

`equalsIgnoreCase()`方法返回一个布尔值。顾名思义，这种方法**在比较`Strings`** `:`时忽略字符的大小写

```java
String string1 = "using equals ignore case";
String string2 = "USING EQUALS IGNORE CASE";

assertThat(string1.equalsIgnoreCase(string2)).isTrue();
```

### 2.4。使用`compareTo()`

`compareTo()`方法返回一个`int`类型值，而**基于字典或自然排序按字典顺序逐个字符比较两个`Strings`和**。

如果两个`Strings`相等，则该方法返回 0；如果第一个`String`在参数之前，则返回一个负数；如果第一个`String`在参数`String.`之后，则返回一个大于零的数

让我们看一个例子:

```java
String author = "author";
String book = "book";
String duplicateBook = "book";

assertThat(author.compareTo(book))
  .isEqualTo(-1);
assertThat(book.compareTo(author))
  .isEqualTo(1);
assertThat(duplicateBook.compareTo(book))
  .isEqualTo(0);
```

### 2.5。使用`compareToIgnoreCase()`

`compareToIgnoreCase()`类似于前面的方法，除了它忽略了大小写:

```java
String author = "Author";
String book = "book";
String duplicateBook = "BOOK";

assertThat(author.compareToIgnoreCase(book))
  .isEqualTo(-1);
assertThat(book.compareToIgnoreCase(author))
  .isEqualTo(1);
assertThat(duplicateBook.compareToIgnoreCase(book))
  .isEqualTo(0);
```

## 3。`String`与`Objects`级的比较

`Objects`是一个包含静态`equals()`方法的实用程序类，在这个场景中很有用——比较两个`Strings.`

如果两个`Strings`相等，则该方法返回`true`，首先通过****使用它们的地址**进行比较，即“`==”`”。因此，如果两个参数都是`null`，它将返回`true`，如果只有一个参数是`null`，它将返回 false。**

 **否则，它会简单地调用被传递参数的类型的类的`equals()`方法——在我们的例子中是`String's`类`equals()`方法。这个方法区分大小写，因为它在内部调用了`String`类的`equals()`方法。

让我们来测试一下:

```java
String string1 = "using objects equals";
String string2 = "using objects equals";
String string3 = new String("using objects equals");

assertThat(Objects.equals(string1, string2)).isTrue();
assertThat(Objects.equals(string1, string3)).isTrue();

assertThat(Objects.equals(null, null)).isTrue();
assertThat(Objects.equals(null, string1)).isFalse();
```

## 4。`String`与`Apache Commons`的比较

**Apache Commons 库包含一个名为`StringUtils`的实用程序类，用于与`String-`相关的操作**；这也有一些对`String`比较非常有益的方法。

### 4.1。使用`equals()`和`equalsIgnoreCase()`和

`StringUtils`类的`equals()`方法是`String`类方法`equals(),`的增强版本，它也处理空值:

```java
assertThat(StringUtils.equals(null, null))
  .isTrue();
assertThat(StringUtils.equals(null, "equals method"))
  .isFalse();
assertThat(StringUtils.equals("equals method", "equals method"))
  .isTrue();
assertThat(StringUtils.equals("equals method", "EQUALS METHOD"))
  .isFalse();
```

`StringUtils`的`equalsIgnoreCase()`方法返回一个`boolean`值。这与`equals(),` 类似，只是它忽略了`Strings:`中字符的大小写

```java
assertThat(StringUtils.equalsIgnoreCase("equals method", "equals method"))
  .isTrue();
assertThat(StringUtils.equalsIgnoreCase("equals method", "EQUALS METHOD"))
  .isTrue();
```

### 4.2。使用`equalsAny()`和`equalsAnyIgnoreCase()`和

`equalsAny()`方法的第一个参数是一个`String`，第二个是一个多参数类型`CharSequence.` ，如果任何其他给定的`Strings`敏感地匹配第一个`String`案例，该方法返回`true`。

否则，将返回 false:

```java
assertThat(StringUtils.equalsAny(null, null, null))
  .isTrue();
assertThat(StringUtils.equalsAny("equals any", "equals any", "any"))
  .isTrue();
assertThat(StringUtils.equalsAny("equals any", null, "equals any"))
  .isTrue();
assertThat(StringUtils.equalsAny(null, "equals", "any"))
  .isFalse();
assertThat(StringUtils.equalsAny("equals any", "EQUALS ANY", "ANY"))
  .isFalse();
```

`equalsAnyIgnoreCase()`方法的工作方式类似于`equalsAny()`方法，但也忽略了大小写:

```java
assertThat(StringUtils.equalsAnyIgnoreCase("ignore case", "IGNORE CASE", "any")).isTrue();
```

### 4.3。使用`compare()`和`compareIgnoreCase()`和

`StringUtils`类中的`compare()`方法是`String`类的`compareTo()` 方法的**空安全版本，并通过**处理小于`non-null`值的`null`值。**两个`null`值被认为相等。**

此外，该方法可用于对带有`null`条目的`Strings`列表进行排序:

```java
assertThat(StringUtils.compare(null, null))
  .isEqualTo(0);
assertThat(StringUtils.compare(null, "abc"))
  .isEqualTo(-1);
assertThat(StringUtils.compare("abc", "bbc"))
  .isEqualTo(-1);
assertThat(StringUtils.compare("bbc", "abc"))
  .isEqualTo(1);
```

`compareIgnoreCase()`方法的行为类似，只是它忽略了大小写:

```java
assertThat(StringUtils.compareIgnoreCase("Abc", "bbc"))
  .isEqualTo(-1);
assertThat(StringUtils.compareIgnoreCase("bbc", "ABC"))
  .isEqualTo(1);
assertThat(StringUtils.compareIgnoreCase("abc", "ABC"))
  .isEqualTo(0);
```

这两种方法也可以与`nullIsLess`选项一起使用。这是**第三个`boolean`参数，它决定了空值是否应该被认为更少**。

如果`nullIsLess`为真，则一个 `null`值低于另一个`String`，如果`nullIsLess`为假，则高于另一个`String`。

让我们试一试:

```java
assertThat(StringUtils.compare(null, "abc", true))
  .isEqualTo(-1);
assertThat(StringUtils.compare(null, "abc", false))
  .isEqualTo(1);
```

带有第三个`boolean`参数`compareIgnoreCase()`方法的工作方式类似，只是忽略了大小写。

## 5。结论

在这个快速教程中，我们讨论了比较`Strings.`的不同方法

和往常一样，例子的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221227210346/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-string-operations)**