# 用 PITest 进行突变测试

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-mutation-testing-with-pitest>

## 1。概述

软件测试是指用于评估软件应用程序功能的技术。在本文中，我们将讨论软件测试行业中使用的一些度量标准，例如**代码覆盖率**和**突变测试**，并特别关注如何使用[**piest 库**](https://web.archive.org/web/20221207163049/http://pitest.org/) 执行突变测试。

为了简单起见，我们将把这个演示建立在一个基本的回文函数的基础上——注意回文是一个前后读起来相同的字符串。

## 2。Maven 依赖关系

正如你在 Maven dependencies 配置中看到的，我们将使用 JUnit 运行我们的测试，并使用 **PITest** 库将**突变体**引入我们的代码——不要担心，我们很快就会看到什么是突变体。通过点击这个[链接](https://web.archive.org/web/20221207163049/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22pitest-parent%22)，您可以随时在 maven 中央存储库中查找最新的依赖版本。

```java
<dependency>
    <groupId>org.pitest</groupId>
    <artifactId>pitest-parent</artifactId>
    <version>1.1.10</version>
    <type>pom</type>
</dependency> 
```

为了启动并运行 PITest 库，我们还需要在我们的`pom.xml`配置文件中包含`pitest-maven`插件:

```java
<plugin>
    <groupId>org.pitest</groupId>
    <artifactId>pitest-maven</artifactId>
    <version>1.1.10</version>
    <configuration>
        <targetClasses>
            <param>com.baeldung.testing.mutation.*</param>
        </targetClasses>
        <targetTests>
            <param>com.baeldung.mutation.test.*</param>
	</targetTests>
     </configuration>
</plugin> 
```

## 3。项目设置

现在我们已经配置了 Maven 依赖项，让我们看看这个不言自明的回文函数:

```java
public boolean isPalindrome(String inputString) {
    if (inputString.length() == 0) {
        return true;
    } else {
        char firstChar = inputString.charAt(0);
        char lastChar = inputString.charAt(inputString.length() - 1);
        String mid = inputString.substring(1, inputString.length() - 1);
        return (firstChar == lastChar) && isPalindrome(mid);
    }
} 
```

我们现在需要的只是一个简单的 JUnit 测试，以确保我们的实现以期望的方式工作:

```java
@Test
public void whenPalindrom_thenAccept() {
    Palindrome palindromeTester = new Palindrome();
    assertTrue(palindromeTester.isPalindrome("noon"));
} 
```

到目前为止，一切顺利，我们已经准备好作为 JUnit 测试成功运行我们的测试用例。

接下来，在本文中，我们将关注使用 PITest 库的**代码和变异覆盖率**。

## 4。代码覆盖率

[代码覆盖率](/web/20221207163049/https://www.baeldung.com/cs/code-coverage)已经在软件行业广泛使用，用来测量在自动化测试中**执行路径**的百分比。

我们可以使用 Eclipse IDE 上可用的工具，如 **[Eclemma](https://web.archive.org/web/20221207163049/http://www.eclemma.org/index.html)** 来测量基于执行路径的有效代码覆盖率。

在使用代码覆盖率运行`TestPalindrome` 之后，我们可以很容易地达到 100%的覆盖率——注意`isPalindrome`是递归的，所以很明显空的输入长度检查无论如何都会被覆盖。

不幸的是，代码覆盖度量有时可能相当**无效**，因为 100%的代码覆盖分数仅意味着所有行至少被执行一次，但它并没有说明**测试准确性**或**用例完整性**，这就是为什么突变测试实际上很重要。

## 5。突变覆盖率

突变测试是一种测试技术，用于**提高测试的充分性**和**识别代码中的缺陷**。这个想法是动态地改变产品代码并导致测试失败。

> 好的测试会失败

代码中的每一次变化都被称为**突变**，它会导致程序版本的改变，被称为**突变**。

如果这种突变能导致测试失败，我们说这种突变是被杀死的。我们还说，如果突变不能影响测试的行为，突变**存活了**。

现在让我们使用 Maven 运行测试，目标选项设置为:`org.pitest:pitest-maven:mutationCoverage`。

我们可以在`**target/pit-test/YYYYMMDDHHMI**`目录下查看 HTML 格式的报告:

*   100%线路覆盖率:7/7
*   63%的突变覆盖率:5/8

显然，我们的测试覆盖了所有的执行路径，因此，行覆盖率是 100%。另一方面，PITest 库引入了 8 个突变体，其中 5 个被杀死——导致失败——但 3 个存活下来。

我们可以查看`**com.baeldung.testing.mutation/Palindrome.java.html**`报告，了解更多关于所创造的变种人的细节:

[![mutations](img/27d8658bfbe9f5e68f3cf6dad5fadc84.png)](/web/20221207163049/https://www.baeldung.com/wp-content/uploads/2016/07/mutations.png)

* * *

* * *

这些是在运行变异覆盖测试时默认激活的**变异子**:

*   `INCREMENTS_MUTATOR`
*   `VOID_METHOD_CALL_MUTATOR`
*   `RETURN_VALS_MUTATOR`
*   `MATH_MUTATOR`
*   `NEGATE_CONDITIONALS_MUTATOR`
*   `INVERT_NEGS_MUTATOR`
*   `CONDITIONALS_BOUNDARY_MUTATOR`

关于 PITest mutators 的更多细节，可以查看官方 **[文档页面](https://web.archive.org/web/20221207163049/http://pitest.org/quickstart/mutators/)** 链接。

我们的突变覆盖率反映了测试用例的缺乏，因为我们不能确保我们的回文函数拒绝非回文和接近回文的字符串输入。

## 6。提高变异分数

现在我们知道了什么是突变，我们需要通过**杀死幸存的突变体**来提高我们的突变分数。

让我们以第 6 行的第一个变异——否定条件——为例。突变体存活了下来，因为即使我们改变了代码片段:

```java
if (inputString.length() == 0) {
    return true;
}
```

收件人:

```java
if (inputString.length() != 0) {
    return true;
}
```

测试会通过的，这就是为什么**突变存活了**。这个想法是实施一个新的测试，如果变异体被引入，那么**将会失败。对于剩余的突变体也可以这样做。**

```java
@Test
public void whenNotPalindrom_thanReject() {
    Palindrome palindromeTester = new Palindrome();
    assertFalse(palindromeTester.isPalindrome("box"));
}
@Test
public void whenNearPalindrom_thanReject() {
    Palindrome palindromeTester = new Palindrome();
    assertFalse(palindromeTester.isPalindrome("neon"));
}
```

现在，我们可以使用突变覆盖率插件运行我们的测试，以确保所有的突变都被杀死，正如我们在目标目录中生成的 PITest 报告中看到的。

*   100%线路覆盖率:7/7
*   100%突变覆盖率:8/8

## 7。PITest 测试配置

突变测试有时可能会占用大量资源，因此我们需要进行适当的配置来提高测试效率。我们可以使用 **`targetClasses`** 标签来定义要变异的类的列表。突变测试不能应用于真实项目中的所有类，因为它将会非常耗时，并且资源非常紧张。

为了最小化执行测试所需的计算资源，定义您计划在变异测试中使用的变异器也很重要:

```java
<configuration>
    <targetClasses>
        <param>com.baeldung.testing.mutation.*</param>
    </targetClasses>
    <targetTests>
        <param>com.baeldung.mutation.test.*</param>
    </targetTests>
    <mutators>
        <mutator>CONSTRUCTOR_CALLS</mutator>
        <mutator>VOID_METHOD_CALLS</mutator>
        <mutator>RETURN_VALS</mutator>
        <mutator>NON_VOID_METHOD_CALLS</mutator>
    </mutators>
</configuration>
```

此外，PITest 库提供了多种选项来**定制你的测试策略**，例如，你可以使用`maxMutationsPerClass`选项指定类引入的突变体的最大数量。官方 **[Maven 快速入门指南](https://web.archive.org/web/20221207163049/http://pitest.org/quickstart/maven/)** 中关于 PITest 选项的更多详情。

## 8。结论

请注意，代码覆盖率仍然是一个重要的指标，但有时它不足以保证一个经过良好测试的代码。因此，在本文中，我们通过使用 **PITest 库**，将**突变测试**作为确保测试质量和认可测试用例的更复杂的方法。

我们还看到了如何分析一个基本的 PITest 报告，同时提高**突变覆盖率得分**。

即使突变测试揭示了代码中的缺陷，也应该明智地使用它，因为这是一个极其昂贵和耗时的过程。

你可以在链接的 [**GitHub 项目**](https://web.archive.org/web/20221207163049/https://github.com/eugenp/tutorials/tree/master/testing-modules/testing-libraries) 中查看本文提供的例子。