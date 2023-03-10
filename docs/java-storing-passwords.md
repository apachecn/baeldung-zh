# 在 Java 中使用字符串上的 char[]数组来操作密码？

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-storing-passwords>

## 1。概述

在本文中，我们将解释为什么在 Java 中我们应该使用`char[]`数组而不是`String`来表示密码。

请注意，本教程关注的是在内存中操作密码的方法，而不是存储密码的实际方法，后者通常在持久层中处理。

**我们还假设我们无法控制密码的格式(例如，密码以`String`的形式来自第三方 API)。尽管使用类型为`java.lang.String`的对象来操作密码似乎是显而易见的，但是 Java 团队自己推荐使用`char[]`来代替。**

例如，如果我们看一下`javax.swing`的 [`JPasswordField`](https://web.archive.org/web/20220630131119/https://docs.oracle.com/en/java/javase/11/docs/api/java.desktop/javax/swing/JPasswordField.html) ，我们可以看到，从 Java 2 开始，返回`String` 的方法`getText()`被弃用，取而代之的是返回`char[]`的`getPassword()`方法。

因此，让我们探讨一下为什么会这样的几个强有力的原因。

## 2。字符串是不可变的

Java 中的是不可变的，这意味着我们不能使用任何高级 API 来改变它们。对一个`String`对象的任何改变都会产生一个新的`String`，将旧的保存在内存中。

因此，存储在`String`中的密码将在内存中可用，直到垃圾收集器将其清除。我们不能控制它什么时候发生，但是这个时间可能比常规对象长得多，因为`Strings`被保存在一个字符串池中，用于重用目的。

因此，任何有权访问内存转储的人都可以从内存中检索密码。

**用`char[]`数组代替`String`，我们可以在完成预期的工作后显式地擦除数据。这样，我们将确保在垃圾收集发生之前就从内存中删除密码。**

现在让我们看一下代码片段，它演示了我们刚刚讨论的内容。

第一个为`String`:

```java
System.out.print("Original String password value: ");
System.out.println(stringPassword);
System.out.println("Original String password hashCode: "
  + Integer.toHexString(stringPassword.hashCode()));

String newString = "********";
stringPassword.replace(stringPassword, newString);

System.out.print("String password value after trying to replace it: ");
System.out.println(stringPassword);
System.out.println(
  "hashCode after trying to replace the original String: "
  + Integer.toHexString(stringPassword.hashCode()));
```

输出将是:

```java
Original String password value: password
Original String password hashCode: 4889ba9b
String value after trying to replace it: password
hashCode after trying to replace the original String: 4889ba9b
```

现在为`char[]`:

```java
char[] charPassword = new char[]{'p', 'a', 's', 's', 'w', 'o', 'r', 'd'};

System.out.print("Original char password value: ");
System.out.println(charPassword);
System.out.println(
  "Original char password hashCode: " 
  + Integer.toHexString(charPassword.hashCode()));

Arrays.fill(charPassword, '*');

System.out.print("Changed char password value: ");
System.out.println(charPassword);
System.out.println(
  "Changed char password hashCode: " 
  + Integer.toHexString(charPassword.hashCode()));
```

输出是:

```java
Original char password value: password
Original char password hashCode: 7cc355be
Changed char password value: ********
Changed char password hashCode: 7cc355be
```

正如我们所看到的，在我们试图替换原始`String`的内容之后，值保持不变，并且在应用程序的同一次执行中`hashCode()`方法没有返回不同的值，这意味着原始的`String`保持不变。

对于`char[]`数组，我们能够改变同一个对象中的数据。

## 3。我们可能会不小心打印出密码

在`char[]`数组中使用密码的另一个好处是防止在控制台、监视器或其他或多或少不安全的地方意外记录密码。

让我们看看下一段代码:

```java
String passwordString = "password";
char[] passwordArray = new char[]{'p', 'a', 's', 's', 'w', 'o', 'r', 'd'};
System.out.println("Printing String password -> " + passwordString);
System.out.println("Printing char[] password -> " + passwordArray);
```

使用输出:

```java
Printing String password -> password
Printing char[] password -> [[[email protected]](/web/20220630131119/https://www.baeldung.com/cdn-cgi/l/email-protection)
```

我们看到在第一种情况下内容本身是打印出来的，而在第二种情况下，数据并没有那么有用，这使得`char[]`不那么容易受到攻击。

## 4。结论

在这篇简短的文章中，我们强调了不应该使用`String`收集密码以及应该使用`char[]`数组的几个原因。

和往常一样，代码片段可以在 GitHub 上找到[。](https://web.archive.org/web/20220630131119/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-strings)