# Java While 循环

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-while-loop>

## 1。概述

在本文中，我们将了解 Java 语言的一个核心方面——使用`while`循环重复执行一条或一组语句。

## 2。`While` 循环

`while`循环是 Java 最基本的循环语句。它重复一个语句或一组语句`while`它的控制`Boolean-expression`为真。

`while` 循环的语法是:

```java
while (Boolean-expression) 
    statement;
```

**循环的`Boolean-expression`在循环的第一次迭代**之前被评估——这意味着如果条件被评估为假，循环可能一次也不会运行。

让我们看一个简单的例子:

```java
int i = 0;
while (i < 5) {
    System.out.println("While loop: i = " + i++);
}
```

## 3。结论

在这个快速教程中，我们探索了 Java 的`while`循环。

像往常一样，可以在 GitHub 上找到例子[。](https://web.archive.org/web/20220628084443/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-syntax)