# Java 中的字符串初始化

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-string-initialization>

## 1。简介

Java *String* 是最重要的类之一，我们已经在与 [`String`相关的系列教程](/web/20221127015222/https://www.baeldung.com/java-string)中介绍了它的很多方面。

在本教程中，我们将关注 Java 中的`String`初始化。

## 2.创造

首先，我们要记住`String`在 Java 中是如何创建的。

我们可以使用`new`关键字或字面语法:

```java
String usingNew = new String("baeldung");
String usingLiteral = "baeldung";
```

同样重要的是，我们要了解如何在专用池中管理 [`String`。](/web/20221127015222/https://www.baeldung.com/java-string-pool)

## 3.`String`仅声明

首先，让我们仅仅用**声明一个`String`** ，而不用显式赋值。

我们可以在本地或作为成员变量来实现这一点:

```java
public class StringInitialization {

    String fieldString;

    void printDeclaredOnlyString() {
        String localVarString;

        // System.out.println(localVarString); -> compilation error
        System.out.println(fieldString);
    }
}
```

正如我们所看到的，如果我们试图在给它一个值之前使用`localVarString` ，我们将得到一个编译错误。另一方面，控制台会显示`null”`为`fieldString`的值。

参见，**成员变量在构造类时用默认值**初始化，在`String`的情况下用`null`初始化。但是，**我们必须自己初始化局部变量。**

如果我们给`localVarString` 一个值`null`，我们会看到这两者现在确实相等:

```java
String localVarString = null;
assertEquals(fieldString, localVarString);
```

## 4.`String`使用文字初始化

现在让我们使用相同的文字创建两个`String`:

```java
String literalOne = "Baeldung";
String literalTwo = "Baeldung";
```

我们将通过比较引用来确认只创建了一个对象:

```java
assertTrue(literalOne == literalTwo);
```

这样做的原因可以追溯到这样一个事实，即 [`String`存储在一个池](/web/20221127015222/https://www.baeldung.com/java-string-pool)中。 **`literalOne `将`String `“拜尔东”添加到池中，`literalTwo `重用它。**

## 5.`String`初始化使用`new`

但是，如果我们使用`new` 关键字，我们会看到一些不同的行为。

```java
String newStringOne = new String("Baeldung");
String newStringTwo = new String("Baeldung");
```

尽管两个`String`的值将与前面的相同，但这次我们将不得不使用不同的对象:

```java
assertFalse(newStringOne == newStringTwo);
```

## 6.空的

现在让我们创建三个空的`String`:

```java
String emptyLiteral = "";
String emptyNewString = new String("");
String emptyNewStringTwo = new String();
```

正如我们现在所知道的，**`emptyLiteral`将被添加到`String`池中，而另外两个将直接进入堆中。**

虽然这些[不会是相同的对象，但是它们都具有相同的值](/web/20221127015222/https://www.baeldung.com/java-compare-strings):

```java
assertFalse(emptyLiteral == emptyNewString)
assertFalse(emptyLiteral == emptyNewStringTwo)
assertFalse(emptyNewString == emptyNewStringTwo)
assertEquals(emptyLiteral, emptyNewString);
assertEquals(emptyNewString, emptyNewStringTwo);
```

## 7.`null`值

最后，让我们看看 null `String`的行为。

让我们声明并初始化一个空值`String`:

```java
String nullValue = null;
```

如果我们打印`nullValue`，我们会看到单词“null”，就像我们之前看到的一样。而且，如果我们试图调用`nullValue,` 上的任何方法，我们会得到预期的`NullPointerException,`。

但是，**为什么“null”会被打印出来呢？`null`究竟是什么？**

嗯， [JVM 规范](https://web.archive.org/web/20221127015222/https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-2.html#jvms-2.4)说`null`是所有引用的缺省值，所以它并不特别依赖于`String`。实际上，规范并没有强制要求`null`有任何具体的值编码。

**那么，打印一个`String` 的“null”从何而来呢？**

如果我们看一下`PrintStream#` `println`的实现，我们会看到它调用了`String#valueOf`:

```java
public void println(Object x) {
    String s = String.valueOf(x);
    synchronized (this) {
        print(s);
        newLine();
    }
}
```

并且，**如果我们看`String#valueOf,` 我们得到我们的答案:**

```java
public static String valueOf(Object obj) {
    return (obj == null) ? "null" : obj.toString();
}
```

很明显，这就是“空”的原因。

## 8.结论

在本文中，我们探讨了`String`初始化。我们解释了声明和初始化之间的区别。我们还提到了使用`new`和字面语法。

最后，我们看了一下将一个`null`值赋给一个`String`值意味着什么，`null`值在内存中是如何表示的，以及当我们打印它时它是什么样子。

本文中使用的所有代码示例都可以从 Github 上的[处获得。](https://web.archive.org/web/20221127015222/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-string-operations-2)