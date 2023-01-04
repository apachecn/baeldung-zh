# Java 中的无限循环

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/infinite-loops-java>

## 1.概观

在这个快速教程中，我们将探索在 Java 中创建无限循环的方法。

简单地说，无限循环是一个指令序列，当终止条件不满足时，它会无休止地循环。创建无限循环可能是编程错误，但也可能是基于应用程序行为的故意行为。

## 2.使用`while`

让我们从`while`循环开始。这里我们将使用布尔文字`true `来编写`while`循环条件:

```
public void infiniteLoopUsingWhile() {
    while (true) {
        // do something
    }
}
```

## 3.使用`for`

现在，让我们使用`for`循环来创建一个无限循环:

```
public void infiniteLoopUsingFor() {
    for (;;) {
        // do something
    }
}
```

## 4.使用`do-while`

使用 Java 中不太常见的`do-while`循环也可以创建无限循环。这里，循环条件在第一次执行后进行评估:

```
public void infiniteLoopUsingDoWhile() {
    do {
        // do something
    } while (true);
}
```

## 5.结论

即使在大多数情况下，我们会避免创建无限循环，但也有一些情况下，我们需要创建一个。在这种情况下，当应用程序退出时，循环将终止。

上面的代码样本可以在 GitHub 库的[中找到。](https://web.archive.org/web/20221126215734/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang)