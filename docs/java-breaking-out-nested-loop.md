# 打破嵌套循环

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-breaking-out-nested-loop>

## 1.概观

在本教程中，我们将创建一些例子来展示在一个循环中使用 [`break`](/web/20221205173945/https://www.baeldung.com/java-continue-and-break) 的不同方式。接下来，我们还将看到如何在完全不使用` break`的情况下终止一个循环。

## 2.问题是

嵌套循环非常有用，例如，在列表列表中搜索。

一个例子是学生列表，其中每个学生都有一个计划课程的列表。假设我们想找到一个策划`course 0`的人的名字。

首先，我们循环查看学生名单。然后，在这个循环中，我们会遍历计划好的课程列表。

当我们打印学生和课程的名字时，我们将得到以下结果:

```java
student 0
  course 0
  course 1
student 1
  course 0
  course 1
```

我们想找到第一个计划`course 0`的学生。然而，如果我们只是使用循环，那么应用程序将在找到课程后继续搜索。

在我们找到一个计划了具体课程的人后，我们想停止搜索。继续搜索会花费更多的时间和资源，而我们并不需要额外的信息。**这就是为什么我们想要打破**的嵌套循环。

## 3.破裂

我们跳出嵌套循环的第一个选择是简单地使用`break`语句:

```java
String result = "";
for (int outerCounter = 0; outerCounter < 2; outerCounter++) {
    result += "outer" + outerCounter;
    for (int innerCounter = 0; innerCounter < 2; innerCounter++) {
        result += "inner" + innerCounter;
        if (innerCounter == 0) {
            break;
        }
    }
}
return result;
```

我们有一个外循环和一个内循环，两个循环都有两次迭代。如果内部循环的计数器等于 0，我们执行`break`命令。当我们运行该示例时，它将显示以下结果:

```java
outer0inner0outer1inner0
```

或者我们可以调整代码，使其更具可读性:

```java
outer 0
  inner 0
outer 1
  inner 0
```

这是我们想要的吗？

几乎，**找到 0 后，内循环被 break 语句** **终止。然而外循环还在继续，**这不是我们想要的。我们想一有答案就完全停止处理。

## 4.标记分隔符

前面的例子是朝着正确方向迈出的一步，但是我们需要对它进行一点改进。我们可以通过使用一个`labeled break`来实现:

```java
String result = "";
myBreakLabel:
for (int outerCounter = 0; outerCounter < 2; outerCounter++) {
    result += "outer" + outerCounter;
    for (int innerCounter = 0; innerCounter < 2; innerCounter++) {
        result += "inner" + innerCounter;
        if (innerCounter == 0) {
            break myBreakLabel;
        }
    }
}
return result;
```

**一个`labeled`中断将终止外环，而不仅仅是内环。**我们通过在循环外添加`myBreakLabel`并更改 break 语句来停止 `myBreakLabel`来实现这一点。运行该示例后，我们得到以下结果:

```java
outer0inner0
```

我们可以通过一些格式更好地阅读它:

```java
outer 0
  inner 0
```

如果我们查看结果，我们可以看到**内循环和外循环都终止了，**这就是我们想要实现的。

## 5.返回

或者，我们也可以使用`return`语句在找到结果时直接返回结果:

```java
String result = "";
for (int outerCounter = 0; outerCounter < 2; outerCounter++) {
    result += "outer" + outerCounter;
    for (int innerCounter = 0; innerCounter < 2; innerCounter++) {
        result += "inner" + innerCounter;
        if (innerCounter == 0) {
            return result;
        }
    }
}
return "failed";
```

标签被移除，`break`语句被替换为`return`语句。

当我们执行上面的代码时，我们得到了与标记的 break 相同的结果。注意，为了使这个策略有效，我们通常需要将循环块移动到它自己的方法中。

## 6.结论

所以，我们刚刚看了当我们需要提前从循环中退出时该怎么做，比如当我们找到了我们要寻找的项目。`break` 关键字对于单循环很有帮助，我们可以对嵌套循环使用带标签的`break`。

**或者，我们可以使用`return`语句。**使用 return 使代码可读性更好，更不容易出错，因为我们不必考虑未标记的和标记的断点之间的区别。

请随意查看 GitHub 上的代码[。](https://web.archive.org/web/20221205173945/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-syntax)