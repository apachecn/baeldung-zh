# 使用 Spock 和 Groovy 进行测试的介绍

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/groovy-spock>

## 1。简介

在本文中，我们将看看 [Spock](https://web.archive.org/web/20221218041156/http://spockframework.org/) ，一个 [Groovy](https://web.archive.org/web/20221218041156/http://groovy-lang.org/) 测试框架。主要是，Spock 的目标是通过利用 Groovy 特性，成为传统 JUnit 堆栈的更强大的替代者。

Groovy 是一种基于 JVM 的语言，它与 Java 无缝集成。在互操作性之上，它提供了额外的语言概念，如动态、可选类型和元编程。

通过使用 Groovy，Spock 引入了测试 Java 应用程序的新的、富有表现力的方法，这在普通的 Java 代码中是不可能的。在本文中，我们将探索 Spock 的一些高级概念，并提供一些实际的分步示例。

## 2。Maven 依赖关系

在我们开始之前，让我们添加我们的 [Maven 依赖项](https://web.archive.org/web/20221218041156/https://search.maven.org/classic/#search%7Cga%7C1%7C%20(g%3A%22org.spockframework%22%20AND%20a%3A%22spock-core%22)%20OR%20(g%3A%22org.codehaus.groovy%22%20AND%20a%3A%22groovy-all%22)):

```java
<dependency>
    <groupId>org.spockframework</groupId>
    <artifactId>spock-core</artifactId>
    <version>1.0-groovy-2.4</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.codehaus.groovy</groupId>
    <artifactId>groovy-all</artifactId>
    <version>2.4.7</version>
    <scope>test</scope>
</dependency>
```

我们添加了 Spock 和 Groovy，就像添加任何标准库一样。然而，由于 Groovy 是一种新的 JVM 语言，为了能够编译和运行它，我们需要包含`gmavenplus` 插件:

```java
<plugin>
    <groupId>org.codehaus.gmavenplus</groupId>
    <artifactId>gmavenplus-plugin</artifactId>
    <version>1.5</version>
    <executions>
        <execution>
            <goals>
                <goal>compile</goal>
                <goal>testCompile</goal>
            </goals>
        </execution>
     </executions>
</plugin>
```

现在我们准备编写我们的第一个 Spock 测试，它将用 Groovy 代码编写。请注意，我们使用 Groovy 和 Spock 只是为了测试目的，这就是为什么这些依赖项是测试范围的。

## 3。Spock 测试的结构

### 3.1。规格和特点

当我们用 Groovy 编写测试时，我们需要将它们添加到`src/test/groovy` 目录中，而不是`src/test/java.` 。让我们在这个目录中创建我们的第一个测试，命名为`Specification.groovy:`

```java
class FirstSpecification extends Specification {

}
```

注意，我们正在扩展`Specification`接口。每个 Spock 类都必须扩展这一点，以使框架对它可用。这样做可以让我们实现我们的第一个`feature:`

```java
def "one plus one should equal two"() {
  expect:
  1 + 1 == 2
}
```

在解释代码之前，还值得注意的是，在 Spock 中，我们提到的`feature` 在某种程度上与我们在 JUnit 中看到的`test` 同义。所以**每当我们提到`feature` 时，我们实际上指的是`test.`**

现在，我们来分析一下我们的`feature`。这样做的话，我们应该能够立即看到它和 Java 之间的一些差异。

第一个区别是特性方法名被写成一个普通的字符串。在 JUnit 中，我们会有一个使用 camelcase 或下划线来分隔单词的方法名，这种方法的表达性和可读性都不好。

其次，我们的测试代码存在于一个`expect` 块中。我们将很快更详细地介绍模块，但是本质上它们是分割我们测试的不同步骤的一种逻辑方式。

最后，我们意识到没有断言。这是因为断言是隐式的，当我们的语句等于`true`时通过，当它等于`false`时失败。同样，我们将很快更详细地讨论断言。

### 3.2。区块

有时，当编写 JUnit 测试时，我们可能会注意到没有一种表达方式来将它分成几个部分。例如，如果我们遵循行为驱动开发，我们可能最终使用注释来表示`given when then` 部分:

```java
@Test
public void givenTwoAndTwo_whenAdding_thenResultIsFour() {
   // Given
   int first = 2;
   int second = 4;

   // When
   int result = 2 + 2;

   // Then
   assertTrue(result == 4)
}
```

斯波克用积木解决了这个问题。 **Blocks 是 Spock 使用标签分解测试阶段的固有方式。**他们给我们贴上`given when then` 等标签:

1.  `Setup`(别名为 Given)–在这里，我们执行测试运行前所需的任何设置。这是一个隐式块，不在任何块中的代码都成为它的一部分
2.  `When`–这是我们为被测对象提供一个`stimulus` 的地方。换句话说，当我们调用测试中的方法时
3.  这是断言的归属。在 Spock 中，这些被评估为简单的布尔断言，这将在后面介绍
4.  `Expect`–这是在同一个街区内执行我们的`stimulus` 和`assertion` 的一种方式。取决于我们发现什么更有表现力，我们可能会或可能不会选择使用这个块
5.  在这里，我们拆除任何测试依赖资源，否则这些资源会被留下。例如，我们可能想要从文件系统中删除任何文件，或者删除写入数据库的测试数据

让我们再次尝试实现我们的测试，这次充分利用块:

```java
def "two plus two should equal four"() {
    given:
        int left = 2
        int right = 2

    when:
        int result = left + right

    then:
        result == 4
}
```

正如我们所看到的，块帮助我们的测试变得更具可读性。

### 3.3。利用 Groovy 特性进行断言

**在`then`和`expect` 块中，断言是隐式的**。

大多数情况下，每个语句都会被求值，如果不是`true`，那么就会失败。当把它与各种 Groovy 特性结合起来时，它很好地消除了对断言库的需求。让我们试着用一个 [`list`](https://web.archive.org/web/20221218041156/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/List.html) 的断言来论证这一点:

```java
def "Should be able to remove from list"() {
    given:
        def list = [1, 2, 3, 4]

    when:
        list.remove(0)

    then:
        list == [2, 3, 4]
}
```

虽然在本文中我们只是简单地介绍了 Groovy，但是有必要解释一下这里发生了什么。

首先，Groovy 为我们提供了更简单的创建列表的方法。我们可以用方括号声明我们的元素，在内部一个`list`将被实例化。

其次，由于 Groovy 是动态的，我们可以使用`def` ，这意味着我们没有为变量声明类型。

最后，在简化测试的背景下，所展示的最有用的特性是操作符重载。这意味着在内部，不是像在 Java 中那样进行引用比较，而是调用`equals()` 方法来比较两个列表。

同样值得演示的是当我们的测试失败时会发生什么。让我们中断它，然后查看控制台的输出:

```java
Condition not satisfied:

list == [1, 3, 4]
|    |
|    false
[2, 3, 4]
 <Click to see difference>

at FirstSpecification.Should be able to remove from list(FirstSpecification.groovy:30)
```

当所有正在进行的事情都是在两个列表上调用`equals()` 时，Spock 足够聪明，可以对失败的断言进行分解，给我们提供有用的调试信息。

### 3.4。断言异常

Spock 还为我们提供了一种检查异常的表达方式。在 JUnit 中，我们的一些选择可能是使用一个`try-catch` 块，在测试的顶部声明`expected` ，或者使用第三方库。Spock 的本地断言提供了一种开箱即用的异常处理方式:

```java
def "Should get an index out of bounds when removing a non-existent item"() {
    given:
        def list = [1, 2, 3, 4]

    when:
        list.remove(20)

    then:
        thrown(IndexOutOfBoundsException)
        list.size() == 4
}
```

这里，我们不需要引入额外的库。另一个优点是`thrown()` 方法将断言异常的类型，但不会暂停测试的执行。

## 4。数据驱动测试

### 4.1。什么是数据驱动测试？

本质上，**数据驱动测试是指我们用不同的参数和断言对相同的行为进行多次测试**。一个典型的例子是测试一个数学运算，比如平方一个数。根据操作数的不同排列，结果会有所不同。在 Java 中，我们可能更熟悉的术语是参数化测试。

### 4.2。用 Java 实现参数化测试

对于某些上下文，使用 JUnit 实现参数化测试是值得的:

```java
@RunWith(Parameterized.class)
public class FibonacciTest {
    @Parameters
    public static Collection<Object[]> data() {
        return Arrays.asList(new Object[][] {     
          { 1, 1 }, { 2, 4 }, { 3, 9 }  
        });
    }

    private int input;

    private int expected;

    public FibonacciTest (int input, int expected) {
        this.input = input;
        this.expected = expected;
    }

    @Test
    public void test() {
        assertEquals(fExpected, Math.pow(3, 2));
    }
}
```

正如我们所看到的，有相当多的冗长，代码不是很可读。我们必须创建一个位于测试之外的二维对象数组，甚至创建一个包装对象来注入各种测试值。

### 4.3。使用 Spock 中的数据表

与 JUnit 相比，Spock 的一个优势是它干净利落地实现了参数化测试。同样，在 Spock 中，这被称为**数据驱动测试。**现在，让我们再次实现相同的测试，只是这次我们将使用带有**数据表**的 Spock，这提供了一种更方便的方式来执行参数化测试:

```java
def "numbers to the power of two"(int a, int b, int c) {
  expect:
      Math.pow(a, b) == c

  where:
      a | b | c
      1 | 2 | 1
      2 | 2 | 4
      3 | 2 | 9
  }
```

正如我们所看到的，我们只有一个包含所有参数的简单明了的数据表。

此外，它属于它应该在的地方，在测试旁边，没有样板文件。这个测试很有表现力，有一个人可读的名字，纯`expect` 和`where` 块来分解逻辑部分。

### 4.4。当数据表失败时

同样值得一提的是，当我们的测试失败时会发生什么:

```java
Condition not satisfied:

Math.pow(a, b) == c
     |   |  |  |  |
     4.0 2  2  |  1
               false

Expected :1

Actual   :4.0
```

再次，斯波克给了我们一个非常有用的错误信息。我们可以确切地看到数据表的哪一行导致了失败以及失败的原因。

## 5。嘲讽

### 5.1。什么是嘲讽？

模仿是一种改变与我们的测试服务合作的类的行为的方式。这是一种有用的方法，可以测试业务逻辑的独立性。

一个典型的例子是用一个简单地假装进行网络调用的类来代替一个进行网络调用的类。更深入的解释，值得一读[这篇文章](/web/20221218041156/https://www.baeldung.com/mockito-vs-easymock-vs-jmockit)。

### 5.2。用斯波克嘲讽

Spock 有自己的模仿框架，利用 Groovy 带给 JVM 的有趣概念。首先，让我们实例化一个`Mock:`

```java
PaymentGateway paymentGateway = Mock()
```

在这种情况下，我们的模拟类型是由变量 type 推断出来的。由于 Groovy 是一种动态语言，我们还可以提供一个类型参数，这样我们就不必将模拟赋给任何特定的类型:

```java
def paymentGateway = Mock(PaymentGateway)
```

现在，每当我们调用我们的`PaymentGateway` mock `,` 上的方法时，都会给出一个默认响应，而不需要调用一个真实的实例:

```java
when:
    def result = paymentGateway.makePayment(12.99)

then:
    result == false
```

这个术语叫做`lenient mocking`。这意味着尚未定义的模拟方法将返回合理的默认值，而不是抛出异常。这是 Spock 的设计，为了使模拟和测试不那么脆弱。

### 5.3。`Mocks` 上的拔法

我们还可以配置 mock 上调用的方法，以某种方式响应不同的参数。当我们支付`20:`时，让我们的`PaymentGateway` 模拟返回`true`

```java
given:
    paymentGateway.makePayment(20) >> true

when:
    def result = paymentGateway.makePayment(20)

then:
    result == true
```

这里有趣的是，Spock 如何利用 Groovy 的操作符重载来存根方法调用。使用 Java，我们必须调用真正的方法，这可能意味着产生的代码更加冗长，表达能力可能更差。

现在，让我们尝试更多类型的存根。

如果我们不再关心我们的方法参数，而总是想返回`true,` ,我们可以只使用下划线:

```java
paymentGateway.makePayment(_) >> true
```

如果我们希望在不同的响应之间进行切换，我们可以提供一个列表，其中的每个元素将按顺序返回:

```java
paymentGateway.makePayment(_) >>> [true, true, false, true]
```

还有更多的可能性，这些可能会在以后关于嘲讽的更高级的文章中涉及到。

### 5.4。验证

我们可能想对 mocks 做的另一件事是断言用预期的参数对它们调用了各种方法。换句话说，我们应该用我们的模拟来验证交互。

验证的一个典型用例是我们的 mock 上的方法是否有一个`void` 返回类型。在这种情况下，由于没有结果可供我们操作，所以没有推断出的行为可供我们通过测试中的方法进行测试。一般来说，如果返回了某个东西，那么测试中的方法可以对它进行操作，而操作的结果就是我们断言的结果。

让我们尝试验证是否调用了具有 void 返回类型的方法:

```java
def "Should verify notify was called"() {
    given:
        def notifier = Mock(Notifier)

    when:
        notifier.notify('foo')

    then:
        1 * notifier.notify('foo')
} 
```

Spock 再次利用了 Groovy 操作符重载。通过将我们的 mocks 方法调用乘以 1，我们可以知道我们期望它被调用了多少次。

如果我们的方法根本没有被调用，或者调用的次数没有达到我们指定的次数，那么我们的测试就不会给出一个信息丰富的 Spock 错误消息。让我们通过期望它被调用两次来证明这一点:

```java
2 * notifier.notify('foo')
```

接下来，让我们看看错误消息是什么样子的。我们会像往常一样；它的信息量很大:

```java
Too few invocations for:

2 * notifier.notify('foo')   (1 invocation)
```

就像 stubbing 一样，我们也可以进行更宽松的验证匹配。如果我们不关心我们的方法参数是什么，我们可以使用下划线:

```java
2 * notifier.notify(_)
```

或者，如果我们想确保它不是用特定的参数调用的，我们可以使用 not 运算符:

```java
2 * notifier.notify(!'foo')
```

同样，还有更多的可能性，可能会在未来更高级的文章中涉及到。

## 6。结论

在这篇文章中，我们已经给出了一个快速切片通过测试与斯波克。

我们已经展示了如何利用 Groovy，使我们的测试比典型的 JUnit 堆栈更具表现力。我们已经解释了`specifications`和`features`的结构。

我们已经展示了执行数据驱动测试是多么容易，以及通过原生的 Spock 功能进行模拟和断言是多么容易。

这些例子的实现可以在 GitHub 的[中找到。这是一个基于 Maven 的项目，所以应该很容易运行。](https://web.archive.org/web/20221218041156/https://github.com/eugenp/tutorials/tree/master/testing-modules/groovy-spock)