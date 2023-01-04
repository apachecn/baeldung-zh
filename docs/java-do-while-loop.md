# Java Do-While 循环

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-do-while-loop>

## 1。概述

在本文中，我们将了解 Java 语言的一个核心方面——使用`do-while`循环重复执行一条或一组语句。

## 2。`Do-While`循环

除了**第一个条件评估发生在循环的第一次迭代之后:**之外，`do-while`循环的工作方式与`while`循环类似

```java
do {
    statement;
} while (Boolean-expression);
```

让我们看一个简单的例子:

```java
int i = 0;
do {
    System.out.println("Do-While loop: i = " + i++);
} while (i < 5);
```

## 3。结论

在这个快速教程中，我们探索了 Java 的`do-while`循环。

像往常一样，可以在 GitHub 上找到例子[。](https://web.archive.org/web/20220627182702/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-syntax)