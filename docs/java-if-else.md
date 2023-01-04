# Java 中的 If-Else 语句

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-if-else>

## 1。概述

在本教程中，我们将学习如何在 Java 中使用`if-else`语句。

`if-else`语句是所有控制结构中最基本的，也可能是**编程中最常见的决策语句**。

它允许我们**仅在满足特定条件时执行某个代码段**。

## 2.`If-Else`的语法

`if`语句**总是需要一个 [`boolean`](/web/20220926185233/https://www.baeldung.com/java-primitives) 表达式作为其参数**。

```
if (condition) {
    // Executes when condition is true.
} else {
    // Executes when condition is false.
}
```

它后面可以跟一个可选的`else`语句，如果布尔表达式是`false` *，将执行该语句的内容。*

## 3.`If`的例子

所以，让我们从最基本的开始。

假设我们只希望在我们的`count`变量大于 1 的情况下发生一些事情:

```
if (count > 1) {
    System.out.println("Count is higher than 1");
}
```

只有当条件通过时，才会打印出消息`Count is higher than 1`。

另外，请注意，在这种情况下，我们在技术上可以删除大括号，因为块中只有一行。但是，**我们应该总是使用大括号来提高可读性**；即使只是一句俏皮话。

当然，如果我们愿意，我们可以向块中添加更多的指令:

```
if (count > 1) {
    System.out.println("Count is higher than 1");
    System.out.println("Count is equal to: " + count);
}
```

## 4.`If-Else`的例子

接下来，我们可以**一起使用`if`和`else`在两个动作过程**中进行选择:

```
if (count > 2) {
    System.out.println("Count is higher than 2");
} else {
    System.out.println("Count is lower or equal than 2");
}
```

**请注意`else`不可能是自己。**必须用一个`if`连接。

## 5.`If-Else If-Else`的例子

最后，让我们以一个组合的`if/else/else if`语法示例结束。

我们可以用它来**在三个或更多选项中选择**:

```
if (count > 2) {
    System.out.println("Count is higher than 2");
} else if (count <= 0) {
    System.out.println("Count is less or equal than zero");
} else {
    System.out.println("Count is either equal to one, or two");
}
```

## 6。结论

在这篇简短的文章中，我们学习了什么是`if-else`语句，以及如何在 Java 程序中使用它来管理流控制。

本文中的所有代码都可以在 GitHub 上的[处获得。](https://web.archive.org/web/20220926185233/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-syntax-2)