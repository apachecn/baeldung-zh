# 用 Maven 构建一个 Jar，忽略测试结果

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/maven-ignore-test-results>

## 1。简介

这篇快速指南展示了如何在忽略测试结果的情况下用 Maven 构建一个 jar。

默认情况下，Maven 会在构建项目时自动运行单元测试。然而，在极少数情况下，测试可以被跳过，我们需要构建项目而不考虑测试结果。

## 2。构建项目

让我们创建一个简单的项目，其中包括一个小的测试用例:

```java
public class TestFail {
    @Test
    public void whenMessageAssigned_thenItIsNotNull() {
        String message = "hello there";
        assertNotNull(message);
    }
}
```

让我们通过执行以下 Maven 命令来构建一个 jar 文件:

```java
mvn package
```

**这将导致编译源代码并在/target 目录下生成一个`maven-0.0.1-SNAPSHOT.jar`文件。**

现在，让我们稍微改变一下测试，这样测试开始失败。

```java
@Test
public void whenMessageAssigned_thenItIsNotNull() {
    String message = null;
    assertNotNull(message);
}
```

这一次，当我们再次尝试运行`mvn package`命令时，构建失败，并且没有创建 maven-0.0.1-SNAPSHOT.jar 文件。

这意味着，如果我们的应用程序中有一个失败的测试，我们不能提供一个可执行文件，除非我们修复这个测试。

那么如何才能解决这个问题呢？

## 3。Maven 参数

Maven 有自己的观点来处理这个问题:

*   `**-Dmaven.test.failure.ignore=true**` **忽略测试执行过程中发生的任何故障**
*   `**-Dmaven.test.skip=true **` **不会编译测试**
*   **`-fn`，`-fae`无论测试结果如何都不会失败的构建**

让我们运行`mvn ` `package -Dmaven.test.skip=true`命令，看看结果:

```java
[INFO] Tests are skipped.
[INFO] BUILD SUCCESS
```

这意味着项目将在不编译测试的情况下构建。

现在让我们运行`mvn ` `package -Dmaven.test.failure.ignore=true `命令:

```java
[INFO] Running testfail.TestFail
[ERROR] whenMessageAssigned_thenItIsNotNull java.lang.AssertionError
[INFO] BUILD SUCCESS
```

我们的单元测试在断言时失败，但是构建是成功的。

最后，我们来测试一下`-fn`、`-fae` 选项。无论`whenMessageAssigned_thenItIsNotNull()`测试失败与否，`package -fn`和`package -fae`命令都会构建`jar`文件并产生`BUILD SUCCESS`输出。

**在多模块项目的情况下，应使用`-fn`选项。`-fae`继续测试失败的模块，但跳过所有相关模块。**

## 4。Maven Surefire 插件

实现我们目标的另一个便捷方法是使用 Maven 的 Surefire 插件。

关于 Surefire 插件的扩展概述，请参考本文。

**要忽略测试失败，我们可以简单地将`testFailureIgnore`属性设置为`true`** :

```java
<plugin>
    <artifactId>maven-surefire-plugin</artifactId>
    <version>${maven.surefire.version}</version>
    <configuration>
        <includes>
            <include>TestFail.java</include>
        </includes>
        <testFailureIgnore>true</testFailureIgnore>
    </configuration>
</plugin>
```

现在，让我们看看`package`命令的输出:

```java
[INFO]  T E S T S
[INFO] Running testfail.TestFail
[ERROR] Tests run: 1, Failures: 1, Errors: 0, Skipped: 0, <<< FAILURE! - in testfail.TestFail
```

从运行测试的输出中，我们可以看到`TestFail`类失败了。但是进一步观察，我们看到构建成功消息也在那里，并且 maven-0.0.1-SNAPSHOT.jar 文件已经编译。

根据我们的需要，我们可以根本不运行测试。为此，我们可以将`testFailureIgnore`行替换为:

```java
<skipTests>true</skipTests>
```

或者设置命令行参数`-DskipTests`。这将编译测试类，但是完全跳过测试执行。

## 5。结论

在本文中，我们学习了如何用 Maven 构建我们的项目，而不管测试结果如何。我们讨论了跳过失败测试或者完全排除测试编译的实际例子。

和往常一样，这篇文章的完整代码可以在 GitHub 项目的[中找到。](https://web.archive.org/web/20220627074421/https://github.com/eugenp/tutorials/tree/master/maven-modules/maven-integration-test)