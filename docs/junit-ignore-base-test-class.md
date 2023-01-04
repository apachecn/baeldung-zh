# JUnit 4 关于如何忽略一个基本测试类

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/junit-ignore-base-test-class>

## 1.概观

本教程将讨论在 JUnit 4 中跳过从基本测试类运行测试的可能解决方案。出于本教程的目的，**基类只有助手方法，而子类将扩展它并运行实际的测试**。

## 2.绕过基本测试类

让我们假设我们有一个带有一些助手方法的`BaseUnitTest`类:

```java
public class BaseUnitTest {
    public void helperMethod() {
        // ...
    }
}
```

现在，让我们用一个包含测试的类来扩展它:

```java
public class ExtendedBaseUnitTest extends BaseUnitTest {
    @Test
    public void whenDoTest_thenAssert() {
        // ...        
    }
}
```

如果我们用 IDE 或者 Maven 构建运行测试，**我们可能会得到一个错误，告诉我们在** `BaseUnitTest` 中没有可运行的测试方法。 **我们不想在这个类中运行测试，** 所以我们在寻找避免这个错误的方法。

我们来看看三种不同的可能性。如果用 IDE 运行测试，我们可能会得到不同的结果，这取决于 IDE 的插件以及我们如何配置它来运行 JUnit 测试。

### 2.1.重命名类别

我们可以将我们的类重命名为一个名称，这个名称将被构建约定排除在测试运行之外。例如，如果我们使用 Maven，我们可以检查 [Maven Surefire 插件](https://web.archive.org/web/20221207193138/https://maven.apache.org/surefire/maven-surefire-plugin/examples/inclusion-exclusion.html "Maven Surefire Plugin")的默认值。

**我们可以将名称从`BaseUnitTest` 改为`BaseUnitTestHelper` 或类似的**:

```java
public class BaseUnitTestHelper {
    public void helperMethod() {
        // ...
    }
}
```

### 2.2.忽视

第二种选择是使用 JUnit `@Ignore`注释暂时禁用测试。我们可以在类级别添加它来禁用一个类中的所有测试:

```java
@Ignore("Class not ready for tests")
public class IgnoreClassUnitTest {
    @Test
    public void whenDoTest_thenAssert() {
        // ...
    }
}
```

同样地，**我们可以在方法级别**添加它，以防我们仍然需要在类中运行其他测试，但只是排除一个或几个:

```java
public class IgnoreMethodTest {
    @Ignore("This method not ready yet")
    @Test
    public void whenMethodIsIgnored_thenTestsDoNotRun() {
        // ...
    }
}
```

如果使用 Maven 运行，我们将看到如下输出:

```java
Tests run: 1, Failures: 0, Errors: 0, Skipped: 1, Time elapsed: 0.041 s - in com.baeldung.IgnoreMethodTest
```

**从 JUnit 5** 开始，`@Disabled`标注取代了`@Ignore`。

### 2.3.制作基类`abstract`

可能是 **最好的办法是做一个基础测试类`[abstract](/web/20221207193138/https://www.baeldung.com/java-abstract-class)`** 。抽象需要一个具体的类来扩展它。这就是为什么 JUnit 在任何情况下都不会将其视为测试实例。

让我们将我们的`BaseUnitTest`类抽象化:

```java
public abstract class BaseUnitTest {
    public void helperMethod() {
        // ...
    }
}
```

## 3.结论

在本文中， **我们已经看到了一些在 JUnit 4** 中从运行测试中排除一个基础测试类的例子。最好的方法是创建抽象类。

JUnit 的`@Ignore`注释也被广泛使用，但被认为是一种不好的做法。 **通过忽略测试，我们可能会忘记他们以及他们忽略** 的原因。

和往常一样，本文中的代码可以从 GitHub 上的[处获得。](https://web.archive.org/web/20221207193138/https://github.com/eugenp/tutorials/tree/master/testing-modules/junit-4)