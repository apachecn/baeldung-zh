# FindBugs 和 PMD 代码质量规则简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/code-quality-metrics>

## 1。概述

在本文中，我们将重点介绍一些代码分析工具中的重要规则，比如 FindBugs、PMD 和 CheckStyle。

## 2。圈复杂度

### 2.1。什么是圈复杂度？

代码复杂度很重要，但是很难衡量。PMD 在其[代码大小规则部分](https://web.archive.org/web/20220807181737/http://pmd.sourceforge.net/pmd-4.3.0/rules/codesize.html)下提供了一组可靠的规则，这些规则旨在检测关于方法大小和结构复杂性的违规。

CheckStyle 以其根据编码标准和格式规则分析代码的能力而闻名。然而，它也可以通过计算一些复杂性度量来检测类/方法设计中的问题。

两个工具中最相关的复杂性度量之一是 CC(圈复杂度)。

CC 值可以通过测量程序的独立执行路径的数量来计算。

例如，下面的方法将产生圈复杂度 3:

```
public void callInsurance(Vehicle vehicle) {
    if (vehicle.isValid()) {
        if (vehicle instanceof Car) {
            callCarInsurance();
        } else {
            delegateInsurance();
        }
    }
}
```

CC 考虑了条件语句和多部分布尔表达式的嵌套。

一般来说，CC 值大于 11 的代码被认为非常复杂，难以测试和维护。

静态分析工具使用的一些常用值如下所示:

*   1-4:低复杂性–易于测试
*   5-7:中等复杂程度–可以忍受
*   8-10:高复杂性——应该考虑重构来简化测试
*   11 +非常高的复杂性–非常难以测试

复杂程度也会影响代码的可测试性，**CC 越高，实现相关测试的难度就越高**。事实上，圈复杂度值准确地显示了达到 100%分支覆盖分数所需的测试用例的数量。

与`callInsurance()`方法相关的流程图为:

[![flowgraph_cc-1](img/bd01c70a2bed98fc9cb619ad7ceb2935.png)](/web/20220807181737/https://www.baeldung.com/wp-content/uploads/2016/11/flowgraph_CC-1.png)

可能的执行路径有:

*   0 => 3
*   0 => 1 => 3
*   0 => 2 => 3

从数学角度来说，CC 可以通过以下简单公式计算:

```
CC = E - N + 2P
```

*   e:总边数
*   n:节点总数
*   p:出口点总数

### 2.2。如何降低圈复杂度？

为了编写复杂程度大大降低的代码，开发人员可能倾向于根据情况使用不同的方法:

*   通过使用设计模式避免编写冗长的`switch`语句，例如，构建器和策略模式可能是处理代码大小和复杂性问题的良好候选
*   通过模块化代码结构，实现 [**单一责任原则**](https://web.archive.org/web/20220807181737/https://en.wikipedia.org/wiki/Single_responsibility_principle) ，编写可重用、可扩展的方法
*   **遵循其他 PMD [代码大小](https://web.archive.org/web/20220807181737/http://pmd.sourceforge.net/pmd-4.3.0/rules/codesize.html)规则可能会对 CC** 产生直接影响，例如过长的方法长度规则、单个类中过多的字段、单个方法中过多的参数列表…等等

你也可以考虑遵循关于代码大小和复杂性的原则和模式，例如 [**KISS(保持简单和愚蠢)原则**](https://web.archive.org/web/20220807181737/https://en.wikipedia.org/wiki/KISS_principle) 和 **[DRY(不要重复自己)](https://web.archive.org/web/20220807181737/https://en.wikipedia.org/wiki/Don't_repeat_yourself)。**

## 3。异常处理规则

与异常相关的缺陷可能是常见的，但其中一些被严重低估，应该被纠正以避免生产代码中的严重功能障碍。

PMD 和 FindBugs 都提供了一些关于异常的规则。下面是我们在处理异常时，在 Java 程序中可能被认为是关键的部分。

### 3.1。最后不要抛出异常

您可能已经知道，Java 中的`finally{}`块通常用于关闭文件和释放资源，将它用于其他目的可能会被认为是 [**码闻**](https://web.archive.org/web/20220807181737/https://en.wikipedia.org/wiki/Code_smell) 。

一个典型的容易出错的例程是在`finally{}`块中抛出一个异常:

```
String content = null;
try {
    String lowerCaseString = content.toLowerCase();
} finally {
    throw new IOException();
}
```

这个方法应该抛出一个`NullPointerException`，但令人惊讶的是它抛出了一个`IOException`，这可能会误导调用方法处理错误的异常。

### 3.2。在`finally`区返回

在`finally{}`块中使用 return 语句可能只会让人困惑。这条规则之所以如此重要，是因为无论何时代码抛出异常，它都会被`return`语句丢弃。

例如，以下代码运行时没有任何错误:

```
String content = null;
try {
    String lowerCaseString = content.toLowerCase();
} finally {
    return;
}
```

一个`NullPointerException`没有被捕获，仍然被`finally`块中的返回语句丢弃。

### 3.3。异常时无法关闭流

关闭流是我们使用`finally`块的主要原因之一，但这并不像看起来那样简单。

以下代码试图关闭`finally`块中的两个流:

```
OutputStream outStream = null;
OutputStream outStream2 = null;
try {
    outStream = new FileOutputStream("test1.txt");
    outStream2  = new FileOutputStream("test2.txt");
    outStream.write(bytes);
    outStream2.write(bytes);
} catch (IOException e) {
    e.printStackTrace();
} finally {
    try {
        outStream.close();
        outStream2.close();
    } catch (IOException e) {
        // Handling IOException
    }
}
```

如果`outStream.close()`指令抛出一个`IOException`，那么`outStream2.close()`将被跳过。

一个快速的解决方法是使用一个单独的 try/catch 块来关闭第二个流:

```
finally {
    try {
        outStream.close();
    } catch (IOException e) {
        // Handling IOException
    }
    try {
        outStream2.close();
    } catch (IOException e) {
        // Handling IOException
    }
}
```

如果你想要一个避免连续`try/catch`块的好方法，检查来自 Apache commons 的 [IOUtils.closeQuiety](https://web.archive.org/web/20220807181737/https://commons.apache.org/proper/commons-io/javadocs/api-release/org/apache/commons/io/IOUtils.html) 方法，它使得处理流关闭而不抛出`IOException`变得简单。

## 5。不良做法

### 5.1。类定义了 compareto()并使用 Object.equals()

每当您实现`compareTo()` 方法时，不要忘记对`equals()`方法做同样的事情，否则，这段代码返回的结果可能会令人困惑:

```
Car car = new Car();
Car car2 = new Car();
if(car.equals(car2)) {
    logger.info("They're equal");
} else {
    logger.info("They're not equal");
}
if(car.compareTo(car2) == 0) {
    logger.info("They're equal");
} else {
    logger.info("They're not equal");
}
```

结果:

```
They're not equal
They're equal
```

为了消除混淆，建议确保在实现 *Comparable，*时不要调用`Object.equals()`，相反，您应该尝试用如下代码覆盖它:

```
boolean equals(Object o) { 
    return compareTo(o) == 0; 
}
```

### 5.2。可能的空指针取消引用

`NullPointerException` (NPE)被认为是 Java 编程中遇到最多的`Exception`，FindBugs 抱怨 Null PointeD dereference 是为了避免抛出。

下面是投掷 NPE 的最基本的例子:

```
Car car = null;
car.doSomething();
```

避免 npe 的最简单方法是执行空检查:

```
Car car = null;
if (car != null) {
    car.doSomething();
}
```

空检查可能会避免 npe，但是当广泛使用时，它们肯定会影响代码的可读性。

所以这里有一些技巧可以用来避免没有空检查的 npe:

*   **编写`:`** 代码时避免使用关键字`null` 这个规则很简单，在初始化变量或返回值时避免使用关键字`null`
*   **用 [`@NotNull`](https://web.archive.org/web/20220807181737/https://docs.oracle.com/javaee/7/api/javax/validation/constraints/NotNull.html) 和 [`@Nullable`](https://web.archive.org/web/20220807181737/http://findbugs.sourceforge.net/api/edu/umd/cs/findbugs/annotations/Nullable.html) 标注**
*   **使用`java.util.Optional`**
*   [**实现空对象模式**](https://web.archive.org/web/20220807181737/https://en.wikipedia.org/wiki/Null_Object_pattern)

## 6。结论

在本文中，我们对静态分析工具检测到的一些关键缺陷进行了全面的观察，并提供了适当解决检测到的问题的基本指南。

你可以通过访问下面的链接来浏览它们的完整规则: [FindBugs](https://web.archive.org/web/20220807181737/http://findbugs.sourceforge.net/bugDescriptions.html) ， [PMD](https://web.archive.org/web/20220807181737/http://pmd.sourceforge.net/pmd-4.3.0/rules/index.html) 。