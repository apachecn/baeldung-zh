# 理解 Java 中的 NumberFormatException

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-number-format-exception>

## 1.介绍

Java 抛出了*NumberFormatException-*[一个未检查的异常](/web/20220817021512/https://www.baeldung.com/java-exceptions)——当它不能[将一个`String`转换成一个数字类型](/web/20220817021512/https://www.baeldung.com/java-convert-string-to-int-or-integer)时。

因为它是[未选中的](/web/20220817021512/https://www.baeldung.com/java-exceptions)，Java 并不强迫我们处理或声明它。

在这个快速教程中，我们将描述和演示 Java 中的**是什么导致了`NumberFormatException`以及如何避免或处理它**。

## 2.`NumberFormatException`的原因

有各种各样的问题导致`NumberFormatException`。例如，Java 中的一些构造函数和方法会抛出这个异常。

我们将在下面的小节中讨论其中的大部分。

### 2.1。传递给构造函数的非数字数据

让我们看一下用非数字数据构造一个`Integer`或`Double`对象的尝试。

这两条语句都会抛出`NumberFormatException`:

```java
Integer aIntegerObj = new Integer("one");
Double doubleDecimalObj = new Double("two.2");
```

让我们看看当我们将无效输入“一”传递给第 1 行的`Integer`构造函数时得到的堆栈跟踪:

```java
Exception in thread "main" java.lang.NumberFormatException: For input string: "one"
	at java.lang.NumberFormatException.forInputString(NumberFormatException.java:65)
	at java.lang.Integer.parseInt(Integer.java:580)
	at java.lang.Integer.<init>(Integer.java:867)
	at MainClass.main(MainClass.java:11)
```

它扔了`NumberFormatException`。在内部使用`parseInt()` 试图理解输入时，`Integer`构造函数失败。

Java Number API 不把单词解析成数字，所以我们只需简单地把代码改成一个期望值就可以了:

```java
Integer aIntegerObj = new Integer("1");
Double doubleDecimalObj = new Double("2.2");
```

### 2.2。解析包含非数字数据的字符串

类似于 Java 对构造函数中解析的支持，我们有专门的解析方法，如`par` `seInt(), parseDouble(),` `valueOf()`和`decode()` `.`

如果我们尝试用这些做同样的转换:

```java
int aIntPrim = Integer.parseInt("two");
double aDoublePrim = Double.parseDouble("two.two");
Integer aIntObj = Integer.valueOf("three");
Long decodedLong = Long.decode("64403L");
```

然后我们会看到同样的错误行为。

而且，我们可以用类似的方法修复它们:

```java
int aIntPrim = Integer.parseInt("2");
double aDoublePrim = Double.parseDouble("2.2");
Integer aIntObj = Integer.valueOf("3");
Long decodedLong = Long.decode("64403");
```

### 2.3。传递带有无关字符的字符串

或者，如果我们试图将一个字符串转换成一个输入中带有**无关数据的数字，比如空格或特殊字符:**

```java
Short shortInt = new Short("2 ");
int bIntPrim = Integer.parseInt("_6000");
```

那么，我们会有和以前一样的问题。

我们可以通过一点字符串操作来纠正这些错误:

```java
Short shortInt = new Short("2 ".trim());
int bIntPrim = Integer.parseInt("_6000".replaceAll("_", ""));
int bIntPrim = Integer.parseInt("-6000");
```

注意第 3 行中的**允许负数**，使用连字符作为减号。

### 2.4。特定于地区的数字格式

让我们来看一个特定于地区的数字的特例。在欧洲地区，逗号可以代表小数点。例如，“4000，1”可能代表十进制数“4000.1”。

默认情况下，我们将通过尝试解析包含逗号的值来获取`NumberFormatException`:

```java
double aDoublePrim = Double.parseDouble("4000,1");
```

在这种情况下，我们需要允许逗号并避免例外。要做到这一点，Java 需要将这里的逗号理解为小数。

**我们可以允许欧洲地区使用逗号，并通过使用`NumberFormat`来避免例外。**

让我们以法国的`Locale`为例来看看它的实际应用:

```java
NumberFormat numberFormat = NumberFormat.getInstance(Locale.FRANCE);
Number parsedNumber = numberFormat.parse("4000,1");
assertEquals(4000.1, parsedNumber.doubleValue());
assertEquals(4000, parsedNumber.intValue()); 
```

## 3.最佳实践

让我们来谈谈几个可以帮助我们应对`NumberFormatException`的好做法:

1.  不要试图将字母或特殊字符转换成数字——Java Number API 无法做到这一点。
2.  我们可能希望使用正则表达式来验证输入字符串，并抛出无效字符的异常。
3.  我们可以使用像`trim()`和`replaceAll()`这样的方法，针对可预见的已知问题净化输入。
4.  在某些情况下，输入中的特殊字符可能是有效的。所以，我们为此做了特殊处理，例如使用`NumberFormat,`，它支持[多种格式](/web/20220817021512/https://www.baeldung.com/java-double-to-string)。

## 4.结论

在本教程中，我们讨论了 Java 中的`NumberFormatException`及其原因。理解这个异常可以帮助我们创建更健壮的应用程序。

此外，我们学习了避免一些无效输入字符串异常的策略。

最后，我们看到了一些处理`NumberFormatException`的最佳实践。

像往常一样，示例中使用的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220817021512/https://github.com/eugenp/tutorials/tree/master/core-java-modules)