# JUnit 中的测试顺序

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/junit-5-test-order>

## 1。概述

默认情况下， **JUnit 使用确定但不可预测的顺序** ( `MethodSorters.DEFAULT`)运行测试。

在大多数情况下，这种行为是完全可以接受的。但是在某些情况下，我们需要执行特定的命令。

## 2。JUnit 5 中的测试订单

在 JUnit 5 中，**我们可以使用`@TestMethodOrder`来控制测试的执行顺序。**

我们可以使用自己的`MethodOrderer`，稍后我们会看到。

或者，我们可以从三个内置排序器中选择一个:

1.  `Alphanumeric`订单
2.  `@Order`注释
3.  随机位

### 2.1。使用`Alphanumeric`命令

JUnit 5 带有一组内置的`MethodOrderer`实现，以字母数字顺序运行测试。

例如，它提供了从`MethodOrderer.MethodName`到**的分类测试方法，根据它们的名字和它们的形参表**:

```java
@TestMethodOrder(MethodOrderer.MethodName.class)
public class AlphanumericOrderUnitTest {
    private static StringBuilder output = new StringBuilder("");

    @Test
    void myATest() {
        output.append("A");
    }

    @Test
    void myBTest() {
        output.append("B");        
    }

    @Test
    void myaTest() {
        output.append("a");
    }

    @AfterAll
    public static void assertOutput() {
        assertEquals("ABa", output.toString());
    }
}
```

类似地，我们可以使用`[MethodOrderer.DisplayName](https://web.archive.org/web/20221126213825/https://junit.org/junit5/docs/current/api/org.junit.jupiter.api/org/junit/jupiter/api/MethodOrderer.DisplayName.html) `到**的排序方法，根据它们的显示名称进行字母数字排序。**

请记住， [`MethodOrderer.Alphanumeric`](https://web.archive.org/web/20221126213825/https://junit.org/junit5/docs/current/api/org.junit.jupiter.api/org/junit/jupiter/api/MethodOrderer.Alphanumeric.html) 是另一种选择。但是，此实现已被弃用，将在 6.0 中删除。

### 2.2。使用`@Order`标注

我们可以使用`@Order`注释来强制测试以特定的顺序运行。

在下面的例子中，这些方法将运行`firstTest()`，然后运行`secondTest()`，最后运行`thirdTest()`:

```java
@TestMethodOrder(OrderAnnotation.class)
public class OrderAnnotationUnitTest {
    private static StringBuilder output = new StringBuilder("");

    @Test
    @Order(1)    
    void firstTest() {
        output.append("a");
    }

    @Test
    @Order(2)    
    void secondTest() {
        output.append("b");
    }

    @Test
    @Order(3)    
    void thirdTest() {
        output.append("c");
    }

    @AfterAll
    public static void assertOutput() {
        assertEquals("abc", output.toString());
    }
}
```

### 2.3。使用随机顺序

我们还可以使用`MethodOrderer.Random` 实现伪随机地排序测试方法:

```java
@TestMethodOrder(MethodOrderer.Random.class)
public class RandomOrderUnitTest {

    private static StringBuilder output = new StringBuilder("");

    @Test
    void myATest() {
        output.append("A");
    }

    @Test
    void myBTest() {
        output.append("B");
    }

    @Test
    void myCTest() {
        output.append("C");
    }

    @AfterAll
    public static void assertOutput() {
        assertEquals("ACB", output.toString());
    }

}
```

事实上，JUnit 5 使用`System.nanoTime()`作为默认种子对测试方法进行排序。**这意味着在可重复测试中，方法的执行顺序可能不一样。**

然而，我们可以使用`junit.jupiter.execution.order.random.seed`属性配置一个定制种子来创建可重复的构建。

我们可以在`junit-platform.properties`文件中指定自定义种子的值:

```java
junit.jupiter.execution.order.random.seed=100
```

### 2.4。使用定制订单

最后，**我们可以通过实现** `**MethodOrderer**` **接口** **来使用自己定制的订单。**

在我们的`CustomOrder`中，我们将按照不区分大小写的字母数字顺序对测试进行排序:

```java
public class CustomOrder implements MethodOrderer {
    @Override
    public void orderMethods(MethodOrdererContext context) {
        context.getMethodDescriptors().sort(
         (MethodDescriptor m1, MethodDescriptor m2)->
           m1.getMethod().getName().compareToIgnoreCase(m2.getMethod().getName()));
    }
}
```

然后，我们将使用`CustomOrder` 按照`myATest()`、`myaTest()`和最后`myBTest()`的顺序运行与上一个例子相同的测试:

```java
@TestMethodOrder(CustomOrder.class)
public class CustomOrderUnitTest {

    // ...

    @AfterAll
    public static void assertOutput() {
        assertEquals("AaB", output.toString());
    }
}
```

### 2.5。设置默认顺序

JUnit 5 提供了一种通过`junit.jupiter.testmethod.order.default` 参数设置默认方法排序器的便捷方式。

同样，我们可以在`junit-platform.properties`文件中配置我们的参数:

```java
junit.jupiter.testmethod.order.default = org.junit.jupiter.api.MethodOrderer$DisplayName
```

**默认排序器将应用于所有不符合`@TestMethodOrder`的测试。**

还有一点要提的是，指定的类必须实现`MethodOrderer`接口。

## 3.JUnit 4 中的测试顺序

对于那些仍在使用 JUnit 4 的人来说，订购测试的 API 略有不同。

让我们看看在以前的版本中实现这一点的选项。

### 3.1.使用`MethodSorters.DEFAULT`

这种默认策略使用散列码来比较测试方法。

在哈希冲突的情况下，使用字典顺序:

```java
@FixMethodOrder(MethodSorters.DEFAULT)
public class DefaultOrderOfExecutionTest {
    private static StringBuilder output = new StringBuilder("");

    @Test
    public void secondTest() {
        output.append("b");
    }

    @Test
    public void thirdTest() {
        output.append("c");
    }

    @Test
    public void firstTest() {
        output.append("a");
    }

    @AfterClass
    public static void assertOutput() {
        assertEquals(output.toString(), "cab");
    }
}
```

当我们运行上面类中的测试时，我们会看到它们都通过了，包括`assertOutput()`。

### 3.2。使用`MethodSorters.JVM`

另一种订购策略是`MethodSorters.JVM`。

**这种策略利用了自然的 JVM 排序，每次运行都有所不同**:

```java
@FixMethodOrder(MethodSorters.JVM)
public class JVMOrderOfExecutionTest {    
    // same as above
}
```

每次我们在这个类中运行测试，都会得到不同的结果。

### 3.3。使用`MethodSorters.NAME_ASCENDING`

最后，这种策略可以用于按照字典顺序运行测试:

```java
@FixMethodOrder(MethodSorters.NAME_ASCENDING)
public class NameAscendingOrderOfExecutionTest {
    // same as above

    @AfterClass
    public static void assertOutput() {
        assertEquals(output.toString(), "abc");
    }
}
```

当我们运行这个类中的测试时，我们看到它们都通过了，包括`assertOutput()`。这确认了我们用注释设置的执行顺序。

## 4。结论

在这篇简短的文章中，我们介绍了 JUnit 中设置执行顺序的方法。

和往常一样，本文中使用的例子可以在 GitHub 上找到[。](https://web.archive.org/web/20221126213825/https://github.com/eugenp/tutorials/tree/master/testing-modules/junit-5)