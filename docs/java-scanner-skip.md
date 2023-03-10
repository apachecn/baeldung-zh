# Java Scanner.skip 方法及示例

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-scanner-skip>

## 1。概述

`java.util.Scanner`有许多我们可以用来验证输入的方法。其中之一就是 `skip()`法。

在本教程中，我们将学习**什么是`skip()` 方法以及如何使用它**。

## 2。`java.util.Scanner.skip()`法

`skip()`方法属于 Java `Scanner`类。它用于跳过与方法参数中传递的指定模式匹配的输入，忽略分隔符。

### 2.1.句法

`skip()`方法**有两个重载的方法签名**:

*   `skip(Pattern pattern)`–将`Scanner`应跳过的模式作为参数
*   `skip(String pattern)`–将指定要跳过的模式的`String`作为参数

### 2.2.返回

`skip()` 返回一个满足方法参数中指定模式的`Scanner`对象。**它还可以抛出两种类型的异常**:如果扫描仪关闭，抛出`IllegalStateException` ，如果没有找到指定模式的匹配，抛出`NoSuchElementException`。

请注意，通过使用不匹配任何内容的模式，可以跳过某些内容而不冒出现`NoSuchElementException`的风险——例如，`skip` ("[ \t]* ")。

## 3。示例

正如我们前面提到的，`skip`方法有两种重载形式。首先，让我们看看如何使用`skip`方法和`Pattern`:

```java
String str = "Java scanner skip tutorial"; 
Scanner sc = new Scanner(str); 
sc.skip(Pattern.compile(".ava"));
```

这里，我们使用了`skip(Pattern)`方法来跳过符合。ava”模式

同样地，`skip(String)` 方法将跳过符合从给定的`String`构造的给定模式的文本。在我们的例子中，我们跳过字符串“Java”:

```java
String str = "Java scanner skip tutorial";
Scanner sc = new Scanner(str); 
sc.skip("Java");
```

简而言之，**使用模式或字符串**两种方法的结果是相同的。

## 4。结论

在这篇短文中，我们已经检查了如何使用`String`或`Pattern`参数来使用`java.util.Scanner`类的`skip()`方法。

和往常一样，讨论中使用的代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220529155700/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-io-4)