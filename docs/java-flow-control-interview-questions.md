# Java 流量控制面试问题(+答案)

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-flow-control-interview-questions>

[This article is part of a series:](javascript:void(0);)[• Java Collections Interview Questions](/web/20221129015953/https://www.baeldung.com/java-collections-interview-questions)
[• Java Type System Interview Questions](/web/20221129015953/https://www.baeldung.com/java-type-system-interview-questions)
[• Java Concurrency Interview Questions (+ Answers)](/web/20221129015953/https://www.baeldung.com/java-concurrency-interview-questions)
[• Java Class Structure and Initialization Interview Questions](/web/20221129015953/https://www.baeldung.com/java-classes-initialization-questions)
[• Java 8 Interview Questions(+ Answers)](/web/20221129015953/https://www.baeldung.com/java-8-interview-questions)
[• Memory Management in Java Interview Questions (+Answers)](/web/20221129015953/https://www.baeldung.com/java-memory-management-interview-questions)
[• Java Generics Interview Questions (+Answers)](/web/20221129015953/https://www.baeldung.com/java-generics-interview-questions)
• Java Flow Control Interview Questions (+ Answers) (current article)[• Java Exceptions Interview Questions (+ Answers)](/web/20221129015953/https://www.baeldung.com/java-exceptions-interview-questions)
[• Java Annotations Interview Questions (+ Answers)](/web/20221129015953/https://www.baeldung.com/java-annotations-interview-questions)
[• Top Spring Framework Interview Questions](/web/20221129015953/https://www.baeldung.com/spring-interview-questions)

## 1。简介

控制流语句允许开发人员使用决策、循环和分支来有条件地改变特定代码块的执行流。

在本文中，我们将讨论一些在面试中可能会出现的流程控制面试问题，并在适当的时候；我们将实现一些例子来更好地理解他们的答案。

## 2。问题

### Q1。描述`if-then`和`if-then-else`语句。什么类型的表达式可以用作条件？

这两个语句都告诉我们的程序，只有当一个特定的条件评估为`true`时，才执行其中的代码。然而，如果 if 子句的计算结果为`false`，那么`if-then-else`语句提供了第二条执行路径:

```java
if (age >= 21) {
    // ...
} else {
    // ...
}
```

与其他编程语言不同，Java 只支持`boolean`表达式作为条件。如果我们试图使用不同类型的表达式，我们会得到一个编译错误。

### Q2。描述`switch`语句。在`switch`子句中可以使用哪些对象类型？

开关允许根据变量值选择几个执行路径。

每条路径都标有`case`或`default`，`switch`语句评估每个`case`表达式是否匹配，并执行匹配标签后面的所有语句，直到找到`break`语句。如果找不到匹配，将改为执行`default`块:

```java
switch (yearsOfJavaExperience) {
    case 0:
        System.out.println("Student");
        break;
    case 1:
        System.out.println("Junior");
        break;
    case 2:
        System.out.println("Middle");
        break;
    default:
        System.out.println("Senior");
}
```

我们可以用`byte`、`short`、`char`、`int`，它们的包装版本、`enum` s、`String` s 作为`switch`值。

### Q3。当我们忘记在`switch`的`case`子句中放置`break`语句时会发生什么？

`switch`声明跌入低谷。这意味着它将继续执行所有的`case`标签，直到 if 找到一个`break`语句，即使这些标签与表达式的值不匹配。

这里有一个例子来说明这一点:

```java
int operation = 2;
int number = 10;

switch (operation) {
    case 1:
        number = number + 10;
        break;
    case 2:
        number = number - 4;
    case 3:
        number = number / 3;
    case 4:
        number = number * 10;
        break;
}
```

运行代码后，`number`保存的值是 20，而不是 6。这在我们希望将同一个操作与多个案例相关联的情况下非常有用。

### Q4。什么时候使用 S `witch`比 I `f-Then-Else`语句更好，反之亦然？

当针对多个单值测试单个变量时，或者当多个值将执行相同的代码时，更适合使用`switch`语句:

```java
switch (month) {
    case 1:
    case 3:
    case 5:
    case 7:
    case 8:
    case 10:
    case 12:
        days = 31;
        break;
case 2:
    days = 28;
    break;
default:
    days = 30;
}
```

当我们需要检查值的范围或多个条件时，最好使用`if-then-else`语句:

```java
if (aPassword == null || aPassword.isEmpty()) {
    // empty password
} else if (aPassword.length() < 8 || aPassword.equals("12345678")) {
    // weak password
} else {
    // good password
}
```

### Q5。Java 支持哪些类型的循环？

Java 提供了三种不同类型的循环:`for`、`while`和`do-while`。

一个`for`循环提供了一种迭代一系列值的方法。当我们预先知道一项任务将重复多少次时，这是最有用的:

```java
for (int i = 0; i < 10; i++) {
     // ...
}
```

当特定条件为`true`时，`while`循环可以执行一组语句:

```java
while (iterator.hasNext()) {
    // ...
}
```

`do-while`是`while`语句的变体，其中`boolean`表达式的求值在循环的底部。这保证了代码至少会执行一次:

```java
do {
    // ...
} while (choice != -1);
```

### Q6。什么是`enhanced for`循环？

`for`语句的另一个语法是用来遍历集合、数组、枚举或任何实现`Iterable`接口的对象的所有元素:

```java
for (String aString : arrayOfStrings) {
    // ...
}
```

### Q7。你如何从一个循环中提前退出？

使用`break`语句，我们可以立即终止循环的执行:

```java
for (int i = 0; ; i++) {
    if (i > 10) {
        break;
    }
}
```

### Q8。无标签和有标签的`break`语句有什么区别？

未标记的`break`语句终止最内层的`switch`、`for`、`while`或`do-while`语句，而标记的`break`语句结束外层语句的执行。

让我们创建一个示例来演示这一点:

```java
int[][] table = { { 1, 2, 3 }, { 25, 37, 49 }, { 55, 68, 93 } };
boolean found = false;
int loopCycles = 0;

outer: for (int[] rows : table) {
    for (int row : rows) {
        loopCycles++;
        if (row == 37) {
            found = true;
            break outer;
        }
    }
}
```

当找到数字 37 时，带标签的`break`语句终止最外层的`for`循环，不再执行更多的循环。因此，`loopCycles`以值 5 结束。

然而，未标记的`break`只结束最里面的语句，将控制流返回到最外面的`for`，该最外面的`for`继续循环到`table`变量中的下一个`row`，使得`loopCycles`以值 8 结束。

### Q9。无标签和有标签的`continue`语句有什么区别？

未标记的`continue`语句跳到最内层的`for`、`while`或`do-while`循环中当前迭代的末尾，而标记了的`continue`则跳到用给定标记标记的外层循环。

这里有一个例子来说明这一点:

```java
int[][] table = { { 1, 15, 3 }, { 25, 15, 49 }, { 15, 68, 93 } };
int loopCycles = 0;

outer: for (int[] rows : table) {
    for (int row : rows) {
        loopCycles++;
        if (row == 15) {
            continue outer;
        }
    }
}
```

推理和上一个问题一样。带标签的`continue`语句终止最外层的`for`循环。

因此，`loopCycles`以值 5 结束，而未标记的版本只终止最里面的语句，使得`loopCycles`以值 9 结束。

### Q10。描述一个`try-catch-finally`结构中的执行流程。

当一个程序进入了`try`块，并且在其中抛出了一个异常时，`try`块的执行被中断，控制流程继续进行，一个`catch`块可以处理抛出的异常。

如果不存在这样的块，则当前的方法执行停止，异常被抛出到调用堆栈上的前一个方法。或者，如果没有异常发生，所有`catch`程序块被忽略，程序继续正常执行。

无论在`try`块的主体内是否抛出异常，总是会执行`finally`块。

### Q11。在哪些情况下不能执行`finally`块？

当 JVM 在执行`try`或`catch`块时被终止，例如，通过调用`System.exit(),` 或者当执行线程被中断或杀死时，那么 finally 块不被执行。

### Q12。执行以下代码的结果是什么？

```java
public static int assignment() {
    int number = 1;
    try {
        number = 3;
        if (true) {
            throw new Exception("Test Exception");
        }
        number = 2;
    } catch (Exception ex) {
        return number;
    } finally {
        number = 4;
    }
    return number;
}

System.out.println(assignment());
```

代码输出数字 3。尽管`finally`块总是被执行，但这只发生在`try`块退出之后。

在这个例子中，`return`语句在`try-catch`块结束之前执行。因此，在`finally`块中对`number`的赋值不起作用，因为变量已经返回给了一个`ssignment`方法的调用代码。

### Q13。在哪些情况下`try-finally`块可能会被使用，即使异常可能不会被抛出？

当我们希望确保不会因为遇到`break`、`continue`或`return`语句而意外绕过代码中使用的资源清理时，这个块非常有用:

```java
HeavyProcess heavyProcess = new HeavyProcess();
try {
    // ...
    return heavyProcess.heavyTask();
} finally {
    heavyProcess.doCleanUp();
}
```

此外，我们可能会面临这样的情况:我们无法在本地处理抛出的异常，或者我们希望当前方法在允许我们释放资源的同时仍然抛出异常:

```java
public void doDangerousTask(Task task) throws ComplicatedException {
    try {
        // ...
        task.gatherResources();
        if (task.isComplicated()) {
            throw new ComplicatedException("Too difficult");
        }
        // ...
    } finally {
        task.freeResources();
    }
}
```

### Q14。`try-with-resources`是如何工作的？

`try-with-resources`语句在执行`try`块之前声明并初始化一个或多个资源，并在语句结束时自动关闭它们，而不管该块是正常完成还是突然完成。任何实现`AutoCloseable`或`Closeable`接口的对象都可以作为资源使用；

```java
try (StringWriter writer = new StringWriter()) {
    writer.write("Hello world!");
}
```

## 3。结论

在本文中，我们讨论了在 Java 开发人员的技术访谈中出现的一些关于控制流语句的最常见问题。这只是进一步研究的开始，而不是详尽的清单。

祝你面试好运。

Next **»**[Java Exceptions Interview Questions (+ Answers)](/web/20221129015953/https://www.baeldung.com/java-exceptions-interview-questions)**«** Previous[Java Generics Interview Questions (+Answers)](/web/20221129015953/https://www.baeldung.com/java-generics-interview-questions)