# Java 中的控制结构

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-control-structures>

## 1。概述

从最基本的意义上说，程序是一系列指令。控制结构是程序块，它可以改变我们执行这些指令的路径。

在本教程中，我们将探索 Java 中的控制结构。

**有三种控制结构:**

*   条件分支，我们使用**在两条或多条路径之间进行选择。**Java 中有三种类型:`if/else/else if`、`ternary operator`和`switch`。
*   用于**的循环遍历多个值/对象，并重复运行特定的代码块。**Java 中基本的循环类型有`for`、`while`和`do while`。
*   分支语句，用于**改变循环中的控制流。**Java 中有两种类型:`break`和`continue`。

## 2.如果/否则/否则如果

`if/else`语句是[最基本的控制结构](/web/20221126221703/https://www.baeldung.com/java-if-else)，但也可以被认为是编程决策的基础。

虽然`if`可以单独使用，但最常见的使用场景是使用`if/else`在两条路径之间进行选择:

```java
if (count > 2) {
    System.out.println("Count is higher than 2");
} else {
    System.out.println("Count is lower or equal than 2");
}
```

**理论上，我们可以无限链接或嵌套`if/else`块，但这会损害代码的可读性，这就是为什么不建议这样做。**

我们将在本文的其余部分探索替代语句。

## 3.三元运算符

**我们可以[使用三元运算符](/web/20221126221703/https://www.baeldung.com/java-ternary-operator)作为速记表达式，其工作方式类似于`if/else`语句。**

让我们再看看我们的`if/else`例子:

```java
if (count > 2) {
    System.out.println("Count is higher than 2");
} else {
    System.out.println("Count is lower or equal than 2");
}
```

我们可以用三元组来重构它，如下所示:

```java
System.out.println(count > 2 ? "Count is higher than 2" : "Count is lower or equal than 2");
```

虽然三进制可以很好地提高我们代码的可读性，但它并不总是`if/else.`的好替代品

## 4.转换

**如果我们有多个案例可供选择，我们可以使用`switch `语句。**

让我们再看一个简单的例子:

```java
int count = 3;
switch (count) {
case 0:
    System.out.println("Count is equal to 0");
    break;
case 1:
    System.out.println("Count is equal to 1");
    break;
default:
    System.out.println("Count is either negative, or higher than 1");
    break;
}
```

三个或更多的`if/else `语句可能很难读懂。作为一种可能的变通方法，我们可以使用如上所示的`switch, `。

还要记住 [*开关*有范围和输入限制](/web/20221126221703/https://www.baeldung.com/java-switch)，我们需要在使用它之前记住。

## 5.环

当我们需要连续多次重复相同的代码时，我们使用[循环](/web/20221126221703/https://www.baeldung.com/java-loops)。

让我们来看一个比较`for`和`while`类型循环的快速示例:

```java
for (int i = 1; i <= 50; i++) {
    methodToRepeat();
}

int whileCounter = 1;
while (whileCounter <= 50) {
    methodToRepeat();
    whileCounter++;
} 
```

上面的两个代码块都会调用`methodToRepeat` 50 次。

## 6.破裂

**我们需要使用 [`break`](/web/20221126221703/https://www.baeldung.com/java-continue-and-break) 来提前退出一个循环。**

让我们看一个简单的例子:

```java
List<String> names = getNameList();
String name = "John Doe";
int index = 0;
for ( ; index < names.length; index++) {
    if (names[index].equals(name)) {
        break;
    }
}
```

在这里，我们在一个名字列表中寻找一个名字，一旦找到就不想再找了。

一个循环通常会完成，但是我们在这里使用了`break`来缩短循环并提前退出。

## 7.继续

简单来说， [`continue`](/web/20221126221703/https://www.baeldung.com/java-continue-and-break) **的意思是跳过我们所在的循环的其余部分:**

```java
List<String> names = getNameList();
String name = "John Doe";
String list = "";
for (int i = 0; i < names.length; i++) { 
    if (names[i].equals(name)) {
        continue;
    }
    list += names[i];
}
```

这里，我们跳过将重复的`names` 追加到列表中。

**正如我们在这里看到的，`break `和`continue` 在迭代时会很方便，尽管它们经常可以用`return`语句或其他逻辑重写。**

## 8.结论

在这篇简短的文章中，我们学习了什么是控制结构，以及如何在 Java 程序中使用它们来管理流控制。

本文中的所有代码都可以在 GitHub 上的[处获得。](https://web.archive.org/web/20221126221703/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-syntax-2)