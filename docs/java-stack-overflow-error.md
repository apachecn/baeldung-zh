# Java 中的 StackOverflowError

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-stack-overflow-error>

## 1。概述

对于 Java 开发人员来说，这可能很烦人，因为这是我们可能遇到的最常见的运行时错误之一。

在本文中，我们将通过查看各种代码示例来了解这个错误是如何发生的，以及我们可以如何处理它。

## 2。堆栈帧和`StackOverflowError`如何发生

让我们从基础开始。**当一个方法被调用时，一个新的堆栈框架在[调用堆栈](/web/20221008115247/https://www.baeldung.com/cs/call-stack)上被创建。**该堆栈框架保存被调用方法的参数、其局部变量和方法的返回地址，即在被调用方法返回后方法执行应该继续的点。

堆栈框架的创建将继续，直到到达嵌套方法中的方法调用的末尾。

**在这个过程中，如果 JVM 遇到没有空间来创建新的堆栈帧的情况，就会抛出`StackOverflowError`。**

JVM 遇到这种情况的最常见原因是**未终止/无限递归**——`StackOverflowError`的 Javadoc 描述提到该错误是由于特定代码片段中的深度递归而引发的。

然而，递归并不是这个错误的唯一原因。这也可能发生在这样的情况下，应用程序一直从方法内部调用方法，直到堆栈耗尽。这是一种罕见的情况，因为没有开发人员会故意遵循糟糕的编码实践。另一个罕见的原因是**在一个方法**中有大量的局部变量。

当一个应用程序被设计成在类之间有**c**循环关系时，也可以抛出 `StackOverflowError`。在这种情况下，彼此的构造函数会被重复调用，从而引发此错误。这也可以被认为是递归的一种形式。

导致这个错误的另一个有趣的场景是，如果一个**类在同一个类中被实例化为那个类**的一个实例变量。这将导致同一个类的构造函数被一次又一次地(递归地)调用，最终导致一个`StackOverflowError.`

在下一节中，我们将看一些演示这些场景的代码示例。

## 3。`StackOverflowError`在行动中

在下面的例子中，由于意外的递归，将会抛出一个`StackOverflowError` ,因为开发人员忘记了为递归行为指定一个终止条件:

```java
public class UnintendedInfiniteRecursion {
    public int calculateFactorial(int number) {
        return number * calculateFactorial(number - 1);
    }
}
```

这里，对于传递到方法中的任何值，都会在所有情况下引发错误:

```java
public class UnintendedInfiniteRecursionManualTest {
    @Test(expected = StackOverflowError.class)
    public void givenPositiveIntNoOne_whenCalFact_thenThrowsException() {
        int numToCalcFactorial= 1;
        UnintendedInfiniteRecursion uir 
          = new UnintendedInfiniteRecursion();

        uir.calculateFactorial(numToCalcFactorial);
    }

    @Test(expected = StackOverflowError.class)
    public void givenPositiveIntGtOne_whenCalcFact_thenThrowsException() {
        int numToCalcFactorial= 2;
        UnintendedInfiniteRecursion uir 
          = new UnintendedInfiniteRecursion();

        uir.calculateFactorial(numToCalcFactorial);
    }

    @Test(expected = StackOverflowError.class)
    public void givenNegativeInt_whenCalcFact_thenThrowsException() {
        int numToCalcFactorial= -1;
        UnintendedInfiniteRecursion uir 
          = new UnintendedInfiniteRecursion();

        uir.calculateFactorial(numToCalcFactorial);
    }
}
```

然而，在下一个示例中，指定了终止条件，但是如果将值 `-1`传递给`calculateFactorial()` 方法，则永远不会满足终止条件，这会导致未终止/无限递归:

```java
public class InfiniteRecursionWithTerminationCondition {
    public int calculateFactorial(int number) {
       return number == 1 ? 1 : number * calculateFactorial(number - 1);
    }
}
```

这组测试演示了这个场景:

```java
public class InfiniteRecursionWithTerminationConditionManualTest {
    @Test
    public void givenPositiveIntNoOne_whenCalcFact_thenCorrectlyCalc() {
        int numToCalcFactorial = 1;
        InfiniteRecursionWithTerminationCondition irtc 
          = new InfiniteRecursionWithTerminationCondition();

        assertEquals(1, irtc.calculateFactorial(numToCalcFactorial));
    }

    @Test
    public void givenPositiveIntGtOne_whenCalcFact_thenCorrectlyCalc() {
        int numToCalcFactorial = 5;
        InfiniteRecursionWithTerminationCondition irtc 
          = new InfiniteRecursionWithTerminationCondition();

        assertEquals(120, irtc.calculateFactorial(numToCalcFactorial));
    }

    @Test(expected = StackOverflowError.class)
    public void givenNegativeInt_whenCalcFact_thenThrowsException() {
        int numToCalcFactorial = -1;
        InfiniteRecursionWithTerminationCondition irtc 
          = new InfiniteRecursionWithTerminationCondition();

        irtc.calculateFactorial(numToCalcFactorial);
    }
}
```

在这种特殊情况下，如果终止条件简单地表示为:

```java
public class RecursionWithCorrectTerminationCondition {
    public int calculateFactorial(int number) {
        return number <= 1 ? 1 : number * calculateFactorial(number - 1);
    }
}
```

下面的测试实际展示了这种情况:

```java
public class RecursionWithCorrectTerminationConditionManualTest {
    @Test
    public void givenNegativeInt_whenCalcFact_thenCorrectlyCalc() {
        int numToCalcFactorial = -1;
        RecursionWithCorrectTerminationCondition rctc 
          = new RecursionWithCorrectTerminationCondition();

        assertEquals(1, rctc.calculateFactorial(numToCalcFactorial));
    }
}
```

现在让我们看一个场景，其中`StackOverflowError`作为类之间循环关系的结果而发生。让我们考虑一下`ClassOne` 和`ClassTwo`，它们在构造函数中实例化了彼此，从而导致了一种循环关系:

```java
public class ClassOne {
    private int oneValue;
    private ClassTwo clsTwoInstance = null;

    public ClassOne() {
        oneValue = 0;
        clsTwoInstance = new ClassTwo();
    }

    public ClassOne(int oneValue, ClassTwo clsTwoInstance) {
        this.oneValue = oneValue;
        this.clsTwoInstance = clsTwoInstance;
    }
}
```

```java
public class ClassTwo {
    private int twoValue;
    private ClassOne clsOneInstance = null;

    public ClassTwo() {
        twoValue = 10;
        clsOneInstance = new ClassOne();
    }

    public ClassTwo(int twoValue, ClassOne clsOneInstance) {
        this.twoValue = twoValue;
        this.clsOneInstance = clsOneInstance;
    }
}
```

现在，假设我们尝试实例化`ClassOne`,如本测试所示:

```java
public class CyclicDependancyManualTest {
    @Test(expected = StackOverflowError.class)
    public void whenInstanciatingClassOne_thenThrowsException() {
        ClassOne obj = new ClassOne();
    }
}
```

这以一个`StackOverflowError`结束，因为`ClassOne` 的构造函数正在实例化`ClassTwo,` ，而`ClassTwo` 的构造函数再次实例化`ClassOne.` ，这重复发生，直到溢出堆栈。

接下来，我们将看看当一个类作为该类的实例变量在同一个类中被实例化时会发生什么。

如下例所示，`AccountHolder` 将其自身实例化为实例变量`jointAccountHolder`:

```java
public class AccountHolder {
    private String firstName;
    private String lastName;

    AccountHolder jointAccountHolder = new AccountHolder();
}
```

当`AccountHolder` 类被实例化`,`时，由于构造函数的递归调用，抛出了一个`StackOverflowError`，如本测试所示:

```java
public class AccountHolderManualTest {
    @Test(expected = StackOverflowError.class)
    public void whenInstanciatingAccountHolder_thenThrowsException() {
        AccountHolder holder = new AccountHolder();
    }
}
```

## 4。`StackOverflowError`对付

当遇到`StackOverflowError`时，最好的办法是仔细检查堆栈跟踪，以识别行号的重复模式。这将使我们能够定位有递归问题的代码。

让我们检查一下由我们前面看到的代码示例引起的一些堆栈跟踪。

如果我们省略了`expected` 异常声明，这个堆栈跟踪由`InfiniteRecursionWithTerminationConditionManualTest` 产生:

```java
java.lang.StackOverflowError

 at c.b.s.InfiniteRecursionWithTerminationCondition
  .calculateFactorial(InfiniteRecursionWithTerminationCondition.java:5)
 at c.b.s.InfiniteRecursionWithTerminationCondition
  .calculateFactorial(InfiniteRecursionWithTerminationCondition.java:5)
 at c.b.s.InfiniteRecursionWithTerminationCondition
  .calculateFactorial(InfiniteRecursionWithTerminationCondition.java:5)
 at c.b.s.InfiniteRecursionWithTerminationCondition
  .calculateFactorial(InfiniteRecursionWithTerminationCondition.java:5)
```

在这里，可以看到第 5 行在重复。这是进行递归调用的地方。现在只需要检查代码，看看递归是否以正确的方式完成。

下面是我们通过执行`CyclicDependancyManualTest` 得到的堆栈跟踪(同样，没有`expected` 异常):

```java
java.lang.StackOverflowError
  at c.b.s.ClassTwo.<init>(ClassTwo.java:9)
  at c.b.s.ClassOne.<init>(ClassOne.java:9)
  at c.b.s.ClassTwo.<init>(ClassTwo.java:9)
  at c.b.s.ClassOne.<init>(ClassOne.java:9)
```

这个堆栈跟踪显示了在具有循环关系的两个类中导致问题的行号。`ClassTwo` 的第 9 行和`ClassOne`的第 9 行指向构造函数内部试图实例化另一个类的位置。

彻底检查完代码后，如果下列情况(或任何其他代码逻辑错误)都不是错误的原因:

*   递归实现不正确(即没有终止条件)
*   类之间的循环依赖
*   将同一类中的一个类实例化为该类的一个实例变量

尝试增加堆栈大小是个好主意。根据所安装的 JVM，默认的堆栈大小可能会有所不同。

从项目的配置或命令行中，`-Xss`标志可以用来增加堆栈的大小。

## 5。结论

在本文中，我们仔细研究了`StackOverflowError`，包括 Java 代码如何导致它，以及我们如何诊断和修复它。

与本文相关的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221008115247/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-exceptions)