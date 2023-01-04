# 通过 Gradle 使用 JUnit 5

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/junit-5-gradle>

## 1。概述

在本教程中，我们将使用 Gradle 构建工具在新的 JUnit 5 平台上运行测试。

我们将配置一个同时支持旧版本和新版本的项目。

想了解更多关于新版本的信息，请随意阅读 JUnit 5 指南。或者是 Gradle 的[介绍，以获得关于构建工具的深入信息。](/web/20220928125431/https://www.baeldung.com/gradle)

## 2\. Gradle Setup

首先，我们验证是否安装了版本 4.6 或更高版本的构建工具，因为这是与 JUnit 5 兼容的最早版本。

最简单的方法是运行`gradle -v`命令:

```java
$> gradle -v
------------------------------------------------------------
Gradle 4.10.2
------------------------------------------------------------
```

如果有必要，我们可以按照[安装](https://web.archive.org/web/20220928125431/https://gradle.org/install/)步骤来获得正确的版本。

**一旦我们安装好了所有的东西，我们就需要使用`build.gradle`文件来配置 Gradle。**

我们可以从向构建工具提供单元测试平台开始:

```java
test {
    useJUnitPlatform()
} 
```

既然我们已经指定了平台，我们需要提供 JUnit 依赖项。这就是我们在 JUnit 5 和早期版本之间看到的显著差异。

看，在早期版本中，我们只需要一个依赖项。然而，在 JUnit 5 中，API 与运行时是分离的，这意味着两个依赖关系。

API 是用`junit-jupiter-api`来表示的。JUnit 5 的运行时是`junit-jupiter-engine`，JUnit 3 或 4 的运行时是`junit-vintage-engine`。

我们将分别在`testImplementation `和`timeRuntimeOnly`中提供这两个:

```java
dependencies {
    testImplementation 'org.junit.jupiter:junit-jupiter-api:5.8.1'
    testRuntimeOnly 'org.junit.jupiter:junit-jupiter-engine:5.8.1'
}
```

## 3。创建测试

让我们编写第一个测试。它看起来就像早期版本:

```java
@Test
public void testAdd() {
    assertEquals(42, Integer.sum(19, 23));
}
```

现在，**我们可以通过执行`gradle clean test`命令**来运行测试。

为了验证我们使用的是 JUnit 5，我们可以查看导入。**`@Test`和`assertEquals`的进口应该有一个以`org.junit.jupiter.api:`** 开头的包

```java
import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.assertEquals;
```

因此，在最后一个例子中，我们创建了一个使用“旧”功能的测试，它已经工作了很多年。我们现在将创建另一个示例，它使用了 JUnit 5 中的一些新功能:

```java
@Test
public void testDivide() {
    assertThrows(ArithmeticException.class, () -> {
        Integer.divideUnsigned(42, 0);
    });
}
```

**`assertThrows`是 JUnit5 中的新断言，取代了旧样式`@Test(expected=ArithmeticException.class).`**

## 4。用 Gradle 配置 JUnit 5 测试

接下来，我们将探索 Gradle 和 JUnit5 之间更深层次的集成。

假设我们的套件中有两种类型的测试:长期运行和短期运行。我们可以使用 JUnit 5 `@Tag `注释:

```java
public class CalculatorJUnit5Test {
    @Tag("slow")
    @Test
    public void testAddMaxInteger() {
        assertEquals(2147483646, Integer.sum(2147183646, 300000));
    }

    @Tag("fast")
    @Test
    public void testDivide() {
        assertThrows(ArithmeticException.class, () -> {
            Integer.divideUnsigned(42, 0);
        });
    }
}
```

然后，我们告诉构建工具执行哪一个。在我们的例子中，让我们只执行短期(快速)测试:

```java
test {
    useJUnitPlatform {
    	includeTags 'fast'
        excludeTags 'slow'
    }
}
```

## 5。启用对旧版本的支持

现在，仍然可以用新的 Jupiter 引擎创建 JUnit 3 和 4 测试。甚至，我们可以在同一个项目中将它们与新版本混合，比如说，在一个迁移场景中。

首先，我们向现有的构建配置添加一些依赖项:

```java
testCompileOnly 'junit:junit:4.12' 
testRuntimeOnly 'org.junit.vintage:junit-vintage-engine:5.8.1'
```

**注意我们的项目现在既有`junit-jupiter-engine `也有`junit-vintage-engine.`和**

现在我们创建一个新类，并复制粘贴我们之前创建的`testDivide`方法。然后，我们添加`@Test`和`assertEquals`的进口。然而，这一次我们确保使用旧的版本 4 包开始哪个`org.junit:`

```java
import static org.junit.Assert.assertEquals;
import org.junit.Test;
public class CalculatorJUnit4Test {
    @Test
    public void testAdd() {
        assertEquals(42, Integer.sum(19, 23));
    }
}
```

## 6。结论

在本教程中，我们集成了 Gradle 和 JUnit 5。此外，我们还增加了对版本 3 和 4 的支持。

我们已经看到构建工具为新旧版本提供了出色的支持。因此，我们可以在现有的项目中使用新的特性，而不需要改变所有现有的测试。

完整的代码示例可以在 [GitHub 项目](https://web.archive.org/web/20220928125431/https://github.com/eugenp/tutorials/tree/master/gradle-modules/gradle/junit5)中找到。请随意将它作为您自己项目的起点。