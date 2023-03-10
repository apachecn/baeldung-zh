# 在 Java 中连接字符串

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-strings-concatenation>

## 1.介绍

Java 提供了大量的方法和类来连接和`**Strings.**`

在本教程中，我们将深入其中几个，并概述一些常见的陷阱和不良做法。

## `2\. StringBuilder`

首先是简陋的`StringBuilder. `这个类**提供了一系列`String-`建筑工具**，使得`String `的操作变得容易。

让我们使用`StringBuilder `类构建一个`String `连接的快速示例:

```java
StringBuilder stringBuilder = new StringBuilder(100);

stringBuilder.append("Baeldung");
stringBuilder.append(" is");
stringBuilder.append(" awesome");

assertEquals("Baeldung is awesome", stringBuilder.toString());
```

在内部， **`StringBuilder`维护一个可变的字符数组。**在我们的代码示例中，我们通过`StringBuilder `构造函数声明它的初始大小为 100 。**由于这种大小声明，`StringBuilder `可以成为连接`Strings`的一种非常有效的** **方式。**

同样值得注意的是， **`StringBuffer `类是** `**StringBuilder**. `的同步版本

尽管同步通常是线程安全的同义词，但由于`StringBuffer's `构建器模式，不建议在多线程应用中使用。虽然对一个同步方法的单个调用是线程安全的，[多个调用不是](https://web.archive.org/web/20221031124834/https://dzone.com/articles/why-synchronized-stringbuffer)。

## 3.加法算子

接下来是加法运算符(+)。这是导致数字相加的同一个运算符，当应用于`Strings.` 时，**被重载以连接**

让我们快速看一下这是如何工作的:

```java
String myString = "The " + "quick " + "brown " + "fox...";

assertEquals("The quick brown fox...", myString);
```

乍一看，这似乎比`StringBuilder `选项更简洁。然而，当源代码编译时，**+符号会转化为一连串的`StringBuilder.append()`调用。**由于这个原因，**混合了串联的`StringBuilder `和+方法**被**认为是不好的做法**。

此外，应该避免在循环中使用+运算符连接。由于`String `对象是不可变的，每次调用串联都会导致一个新的`String `对象被创建。

## 4.`String` 方法

`String `类本身提供了一整套连接`Strings.`的方法

### 4.1.`String.concat`

不出所料，`String.concat`方法是我们试图连接`String `对象时的第一个调用端口。**这个方法返回一个`String `对象，所以将方法链接在一起是一个有用的特性。**

```java
String myString = "Both".concat(" fickle")
  .concat(" dwarves")
  .concat(" jinx")
  .concat(" my")
  .concat(" pig")
  .concat(" quiz");

assertEquals("Both fickle dwarves jinx my pig quiz", myString);
```

在这个例子中，我们的链从一个`String `文字开始，然后`concat `方法允许我们将调用链接起来以追加进一步的`Strings`。

### 4.2.`String.format`

接下来是[`String.format `方法](/web/20221031124834/https://www.baeldung.com/java-string-formatter)，它允许我们将各种 Java `Objects `注入到一个`String `模板中。

`String.format `方法签名使用一个**信号`String `来表示我们的模板**。这个**模板包含“%”字符来表示各种`Objects` 应该放在**中的什么位置。

一旦我们的模板被声明，那么**接受一个 [varargs](/web/20221031124834/https://www.baeldung.com/java-varargs) `Object `数组，该数组被注入**到模板中。

让我们通过一个简单的例子来看看这是如何工作的:

```java
String myString = String.format("%s %s %.2f %s %s, %s...", "I",
  "ate",
  2.5056302,
  "blueberry",
  "pies",
  "oops");

assertEquals("I ate 2.51 blueberry pies, oops...", myString);
```

正如我们在上面看到的，该方法将我们的`Strings `注入了正确的格式。

### 4.3.`String.join` (Java 8+)

如果我们的**应用程序运行在 Java 8** **或更高版本**上，我们可以利用`String.join `方法。有了这个，我们可以用一个公共分隔符将**加入到一个`Strings `的数组中，确保没有空格被遗漏。**

```java
String[] strings = {"I'm", "running", "out", "of", "pangrams!"};

String myString = String.join(" ", strings);

assertEquals("I'm running out of pangrams!", myString); 
```

这种方法的一个巨大优势是不必担心字符串之间的分隔符。

## 5.`StringJoiner ` (Java 8+)

`StringJoiner`将所有的`String.join `功能抽象成一个简单易用的类。**构造函数接受一个分隔符，带有可选的前缀和后缀**。我们可以使用名副其实的`add `方法添加`Strings `。

```java
StringJoiner fruitJoiner = new StringJoiner(", ");

fruitJoiner.add("Apples");
fruitJoiner.add("Oranges");
fruitJoiner.add("Bananas");

assertEquals("Apples, Oranges, Bananas", fruitJoiner.toString());
```

通过使用这个类，代替`String.join `方法，**我们可以在程序运行**时追加`Strings `；不需要先创建数组！

阅读我们关于`StringJoiner` 的[文章，了解更多信息和例子。](/web/20221031124834/https://www.baeldung.com/java-string-joiner)

## 6.`Arrays.toString`

关于数组， [`Array `类](/web/20221031124834/https://www.baeldung.com/java-util-arrays)也包含**一个方便的`toString `方法，它很好地格式化了一个对象数组。**`Arrays.``toString `方法也调用任何封闭对象的`toString `方法——所以我们需要确保我们定义了一个方法。

```java
String[] myFavouriteLanguages = {"Java", "JavaScript", "Python"};

String toString = Arrays.toString(myFavouriteLanguages);

assertEquals("[Java, JavaScript, Python]", toString);
```

不幸的是，`Arrays.` `toString `方法是不可定制的，只有**输出方括号中的`String `。**

## 7.`Collectors.joining ` (Java 8+)

最后，让我们来看看`Collectors.joining `方法，该方法**允许我们将一个`Stream `的输出汇集到一个`String.`T4 中**

```java
List<String> awesomeAnimals = Arrays.asList("Shark", "Panda", "Armadillo");

String animalString = awesomeAnimals.stream().collect(Collectors.joining(", "));

assertEquals("Shark, Panda, Armadillo", animalString);
```

使用 streams 可以解锁与 [Java 8 `Stream ` API](/web/20221031124834/https://www.baeldung.com/java-streams) 相关的所有功能，比如过滤、映射、迭代等等。

## 8.包裹

在本文中，我们对 Java 语言中用于连接`Strings` 的众多类和方法进行了**式的深入探究。**

和往常一样，源代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20221031124834/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-string-operations-2)