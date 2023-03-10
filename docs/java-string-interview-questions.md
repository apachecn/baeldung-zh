# Java 字符串面试问答

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-string-interview-questions>

## 1。简介

[`String`类](/web/20221208143926/https://www.baeldung.com/java-string)是 Java 中使用最广泛的类之一，这促使语言设计者对它进行特殊处理。这种特殊的行为使它成为 Java 面试中最热门的话题之一。

在本教程中，我们将讨论一些最常见的关于`String`的面试问题。

## 2。`String`基本面

这一部分由关于`String`内部结构和记忆的问题组成。

### Q1。Java 中的字符串是什么？

在 Java 中，`String`在内部由一组`byte`值(或 JDK 9 之前的`char`值)表示。

在 Java 8 及更高版本中，`String`由不可变的 Unicode 字符数组组成。然而，大多数字符只需要 8 位(1 个`byte)`来表示它们，而不是 16 位`(char`大小)。

为了改善内存消耗和性能，Java 9 引入了[紧凑字符串](/web/20221208143926/https://www.baeldung.com/java-9-compact-string)。这意味着如果一个`String`只包含 1 字节的字符，它将使用`Latin-1`编码来表示。如果一个`String`包含至少 1 个多字节字符，它将使用 UTF-16 编码表示为每个字符 2 个字节。

在 C 和 C++中，`String`也是一个字符数组，但在 Java 中，它是一个独立的对象，有自己的 API。

### Q2。怎样才能在 Java 中创建一个 String 对象？

[`java.lang.String`](https://web.archive.org/web/20221208143926/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/String.html) 定义了[创建`String`](https://web.archive.org/web/20221208143926/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/String.html#constructor.summary) 的 13 种不同方式。一般来说，有两个:

*   穿过
*   通过`new`关键字:

    ```java
    String s = new String("abc");
    ```

Java 中的所有字符串都是`String`类的实例。

### Q3。`String`是原语还是派生类型？

`A String`是派生类型，因为它有状态和行为。例如，它有原语所没有的`substring()`、`indexOf()`和`equals(), `等方法。

但是，由于我们都经常使用它，它有一些特殊的特征，使它感觉像一个原始人:

*   虽然字符串不像原语那样存储在调用栈中，但是它们**存储在一个叫做[字符串池](/web/20221208143926/https://www.baeldung.com/java-string-pool)的特殊内存区域中**
*   像原语一样，我们可以在字符串上使用`+ `操作符
*   同样，像原语一样，我们可以创建一个没有关键字`new `的`String `的实例

### Q4。字符串不可变有什么好处？

根据詹姆斯·高斯林的采访，字符串是不可变的，以提高性能和安全性。

实际上，我们看到了拥有不可变字符串的几个[好处:](/web/20221208143926/https://www.baeldung.com/java-string-immutable)

*   **字符串池只有在字符串一旦被创建就不会被更改的情况下才是可能的，**因为它们应该被重用
*   代码可以**安全地将一个字符串传递给另一个方法**，知道它不能被那个方法修改
*   不变地**自动使这个类成为线程安全的**
*   因为这个类是线程安全的，**不需要同步公共数据**，这反过来提高了性能
*   由于保证它们不会改变，**它们的 hashcode 可以很容易地被缓存**

### Q5。一个字符串是如何存储在内存中的？

根据 JVM 规范，`String`文字存储在一个[运行时常量池](https://web.archive.org/web/20221208143926/https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-2.html#jvms-2.5.5)中，该池是从 JVM 的[方法区](https://web.archive.org/web/20221208143926/https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-2.html#jvms-2.5.4)中分配的。

虽然方法区在逻辑上是堆内存的一部分，但规范并没有规定位置、内存大小或垃圾收集策略。它可以是特定于实现的。

这个类或接口的运行时常量池是在 JVM 创建类或接口时构建的。

### Q6。在 Java 中，被拘留的字符串有资格进行垃圾收集吗？

是的，如果没有来自程序的引用，字符串池中的所有`String`都有资格进行垃圾收集。

### Q7。什么是字符串常量池？

[字符串池](/web/20221208143926/https://www.baeldung.com/java-string-pool)，也称为`String`常量池或`String`实习生池，是 JVM 存储`String`实例的一个特殊内存区域。

**它通过减少分配字符串的频率和数量来优化应用性能**:

*   JVM 在池中只存储一个特定`String`的副本
*   当创建新的`String`时，JVM 在池中搜索具有相同值的`String`
*   如果找到了，JVM 返回对那个`String`的引用，而不分配任何额外的内存
*   如果没有找到，那么 JVM 会将它添加到池中(实习它),并返回它的引用

### Q8。String 是线程安全的吗？怎么会？

字符串确实是完全线程安全的，因为它们是不可变的。任何不可变的类都自动符合线程安全的条件，因为它的不变性保证了它的实例不会在多个线程中被改变。

**例如，如果一个线程改变了一个字符串的值，一个新的`String`会被创建，而不是修改现有的。**

### Q9。为哪些字符串操作提供区域设置很重要？

`Locale`类允许我们区分不同的文化区域，并适当地格式化我们的内容。

说到`String `类，当在`format`中呈现字符串或者小写或大写字符串时，我们需要它。

事实上，如果我们忘记这样做，我们可能会在可移植性、安全性和可用性方面遇到问题。

### Q10。字符串的底层字符编码是什么？

[根据`String'` s Javadocs](https://web.archive.org/web/20221208143926/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/String.html) 对于 Java 8 及以下版本，字符串在内部以 UTF-16 格式存储。

`char`数据类型和`java.lang.Character`对象[也是基于最初的 Unicode 规范](https://web.archive.org/web/20221208143926/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Character.html#unicode)，它将字符定义为固定宽度的 16 位实体。

从 JDK 9 开始，仅包含 1 字节字符的`Strings` 使用`Latin-1`编码，而至少包含 1 个多字节字符的`Strings`使用 UTF-16 编码。

## 3。`String` API

在这一节中，我们将讨论一些与`String` API 相关的问题。

### Q11。如何在 Java 中比较两个字符串？`str1 == str2`和`str1.equals(str2)`有什么区别？

我们可以用两种不同的方式来比较字符串:使用等于运算符(==)和使用`equals()`方法。

两者都截然不同:

*   **操作符(`str1 == str2` )** 检查引用是否相等
*   **方法(`str1.equals(str2)` )** 检查词法是否相等

不过，如果两个字符串在词汇上相等，那么`str1.intern() == str2.intern() `也是`true`也是正确的。

**通常，为了比较两个`Strings`的内容，我们应该总是使用`String.equals`。**

### Q12。如何在 Java 中拆分一个字符串？

`String`类本身为我们提供了[`String#``split `方法](/web/20221208143926/https://www.baeldung.com/string/split)，它接受一个正则表达式分隔符。它返回给我们一个`String[]`数组:

```java
String[] parts = "john,peter,mary".split(",");
assertEquals(new String[] { "john", "peter", "mary" }, parts);
```

**关于`split`的一件棘手的事情是，当拆分一个空字符串**时，我们可能会得到一个非空数组:

```java
assertEquals(new String[] { "" }, "".split(","));
```

当然，`split `只是[拆分一个 Java `String`](/web/20221208143926/https://www.baeldung.com/java-split-string) 的众多方式之一。

### Q13。什么是 Stringjoiner？

`[StringJoiner](/web/20221208143926/https://www.baeldung.com/java-string-joiner) `是 Java 8 中引入的一个类，用于将单独的字符串连接成一个字符串，就像**获取一个颜色列表并以逗号分隔的字符串**的形式返回它们。我们可以提供分隔符以及前缀和后缀:

```java
StringJoiner joiner = new StringJoiner(",", "[", "]");
joiner.add("Red")
  .add("Green")
  .add("Blue");

assertEquals("[Red,Green,Blue]", joiner.toString());
```

### Q14。String、Stringbuffer 和 Stringbuilder 的区别？

字符串是不可变的。这意味着**如果我们试图改变它的值，那么 Java 会创建一个全新的** `**String**. `

例如，如果我们在字符串`str1 `创建后添加它:

```java
String str1 = "abc";
str1 = str1 + "def";
```

然后 JVM 不是修改`str1`，而是创建一个全新的`String`。

但是，对于大多数简单的情况，编译器内部会使用`StringBuilder`并优化上面的代码。

但是，对于更复杂的代码，比如循环，**会产生一个全新的`String`，恶化性能**。这就是`StringBuilder`和`StringBuffer`有用的地方。

Java 中的 [`StringBuilder`和`StringBuffer`都创建持有可变字符序列的对象。 **`StringBuffer`是同步的，因此是线程安全的，而`StringBuilder`不是。**](/web/20221208143926/https://www.baeldung.com/java-string-builder-string-buffer)

由于`StringBuffer `中的额外同步通常是不必要的，我们通常可以通过选择`StringBuilder.`来获得性能提升

### Q15。为什么将密码存储在 Char[]数组中比存储在字符串中更安全？

因为字符串是不可变的，所以不允许修改。**这种行为使我们无法覆盖、修改或清空其内容，使`Strings`不适合存储敏感信息。**

我们必须依靠垃圾收集器来移除字符串的内容。此外，在 Java 版本 6 和更低版本中，字符串存储在 PermGen 中，这意味着一旦创建了一个`String`，它就不会被垃圾收集。

**通过使用`char[]`数组，我们可以完全控制这些信息。** **我们甚至不需要依靠垃圾收集器就可以对其进行修改或彻底擦除。**

使用`char[]`而不是`String`并不能完全保护信息；这只是一种额外的措施，可以减少恶意用户获取敏感信息的机会。

### Q16。String 的 intern()方法是做什么的？

**方法 [`intern()`](/web/20221208143926/https://www.baeldung.com/string/intern) 在堆中创建一个`String`对象的精确副本，并将其存储在由 JVM 维护的`String `常量池中。**

Java 会自动替换所有使用字符串文字创建的字符串，但是如果我们使用 new 运算符创建一个`String`，例如`String str = new String(“abc”)`，那么 Java 会将它添加到堆中，就像其他任何对象一样。

我们可以调用`intern()`方法告诉 JVM 将它添加到字符串池中(如果它还不存在的话),并返回一个被保留的字符串的引用:

```java
String s1 = "Baeldung";
String s2 = new String("Baeldung");
String s3 = new String("Baeldung").intern();

assertThat(s1 == s2).isFalse();
assertThat(s1 == s3).isTrue();
```

### Q17。如何在 Java 中实现字符串到整数，整数到字符串的转换？

将`String`转换为`Integer` 的最直接的方法是使用`Integer#` `parseInt`:

```java
int num = Integer.parseInt("22");
```

反过来，我们可以使用`Integer#` `toString`:

```java
String s = Integer.toString(num);
```

### Q18。什么是 String.format()以及如何使用它？

[`String#format`](/web/20221208143926/https://www.baeldung.com/string/format) 使用指定的格式字符串和参数返回格式化字符串。

```java
String title = "Baeldung"; 
String formatted = String.format("Title is %s", title);
assertEquals("Title is Baeldung", formatted);
```

我们还需要记住指定用户的`Locale, `,除非我们接受操作系统的默认设置:

```java
Locale usersLocale = Locale.ITALY;
assertEquals("1.024",
  String.format(usersLocale, "There are %,d shirts to choose from. Good luck.", 1024))
```

### Q19。如何将一个字符串转换成大写和小写？

`String`隐式提供了 [`String#toUpperCase`](/web/20221208143926/https://www.baeldung.com/string/to-upper-case) 将大小写改为大写。

尽管如此，Javadocs 提醒我们需要指定用户的`L` `ocale`以确保正确性:

```java
String s = "Welcome to Baeldung!";
assertEquals("WELCOME TO BAELDUNG!", s.toUpperCase(Locale.US));
```

同样，要转换成小写，我们有 [`String#toLowerCase`](/web/20221208143926/https://www.baeldung.com/string/to-lower-case) :

```java
String s = "Welcome to Baeldung!";
assertEquals("welcome to baeldung!", s.toLowerCase(Locale.UK));
```

### Q20。如何从 String 中得到一个字符数组？

`String`提供`toCharArray`，它返回其内部`char`数组在 JDK9 之前的副本(并将`String`转换为 JDK9+中新的`char`数组):

```java
char[] hello = "hello".toCharArray();
assertArrayEquals(new String[] { 'h', 'e', 'l', 'l', 'o' }, hello);
```

### Q21。我们如何将一个 Java 字符串转换成一个字节数组？

默认情况下， [`String#getBytes()`](/web/20221208143926/https://www.baeldung.com/string/get-bytes) 方法使用平台的默认字符集将一个字符串编码成一个字节数组。

虽然 API 不要求我们指定字符集，但是为了确保安全性和可移植性，我们应该指定字符集:

```java
byte[] byteArray2 = "efgh".getBytes(StandardCharsets.US_ASCII);
byte[] byteArray3 = "ijkl".getBytes("UTF-8");
```

## 4。基于`String`的算法

在这一节，我们将讨论一些与`String` s 相关的编程问题。

### Q22。如何在 Java 中检查两个字符串是否是变位词？

变位词是通过重新排列另一个给定单词的字母而形成的单词，例如，“car”和“arc”。

首先，我们首先检查两个`Strings`是否长度相等。

然后我们[将它们转换成`char[]`数组，对它们进行排序，然后检查相等性](/web/20221208143926/https://www.baeldung.com/java-sort-string-alphabetically)。

### Q23。我们如何计算一个给定字符在一个字符串中出现的次数？

Java 8 真正简化了如下聚合任务:

```java
long count = "hello".chars().filter(ch -> (char)ch == 'l').count();
assertEquals(2, count);
```

此外，还有其他几种计算 l 的好方法，包括循环、递归、正则表达式和外部库。

### Q24。如何在 Java 中反转一个字符串？

有许多方法可以做到这一点，最直接的方法是使用来自`StringBuilder`(或`StringBuffer`)的`reverse`方法:

```java
String reversed = new StringBuilder("baeldung").reverse().toString();
assertEquals("gnudleab", reversed);
```

### Q25。怎样才能检验一个字符串是不是回文？

一个[回文](/web/20221208143926/https://www.baeldung.com/java-palindrome-substrings)是任何一个倒着读和正着读一样的字符序列，比如“madam”、“radar”或者“level”。

为了[检查一个字符串是否是回文](/web/20221208143926/https://www.baeldung.com/java-palindrome)，我们可以开始在一个循环中向前和向后迭代给定的字符串，一次一个字符。循环在第一次不匹配时退出。

## 5。结论

在这篇文章中，我们讨论了一些最常见的面试问题。

这里使用的所有代码示例都可以在 GitHub 上获得[。](https://web.archive.org/web/20221208143926/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-strings)