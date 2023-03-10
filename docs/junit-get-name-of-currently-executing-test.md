# 获取 JUnit 中当前正在执行的测试的名称

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/junit-get-name-of-currently-executing-test>

## 1.概观

当使用 JUnit 时，我们可能需要我们的测试来访问它们的名字。这可能有助于处理错误消息，尤其是对于系统生成名称的测试。

在这个简短的教程中，我们将看看如何在 [JUnit 4](/web/20220524050301/https://www.baeldung.com/junit) 和 [JUnit 5](/web/20220524050301/https://www.baeldung.com/junit-5) 中获取当前测试用例的名称。

## 2.JUnit 5 方法

我们来看两个场景。首先，我们将看到如何访问单个测试的名称。这个名字通常是可预测的，因为它可能是函数的名字或者是 [`@DisplayName`](/web/20220524050301/https://www.baeldung.com/junit-5#2-displayname-and-disabled) 注释的值。然而，如果我们使用参数化测试或[显示名称生成器](/web/20220524050301/https://www.baeldung.com/junit-custom-display-name-generator)，那么我们可能需要知道 JUnit 提供的名称。

**JUnit 5 可以在我们的测试**中注入一个`TestInfo`对象来显示当前测试用例的名称。

### 2.1.个别测验

让我们在测试函数中注入一个`TestInfo`对象:

```java
@Test
void givenNumbers_whenOddCheck_thenVerify(TestInfo testInfo) {
    System.out.println("displayName = " + testInfo.getDisplayName());
    int number = 5;
    assertTrue(oddCheck(number));
}
```

这里我们使用了接口`TestInfo` 的 **`getDisplayName` 方法来显示测试**的名称。当我们运行测试时，我们得到测试名:

```java
displayName = givenNumbers_whenOddCheck_thenVerify(TestInfo)
```

### 2.2.参数化测试

让我们用一个[参数化测试](/web/20220524050301/https://www.baeldung.com/parameterized-tests-junit-5)来尝试一下。这里我们将使用`@ParameterizedTest` 注释的`name`字段向 JUnit 描述如何为我们生成测试名称:

```java
private TestInfo testInfo;

@BeforeEach
void init(TestInfo testInfo) {
    this.testInfo = testInfo;
}

@ParameterizedTest(name = "givenNumbers_whenOddCheck_thenVerify{0}")
@ValueSource(ints = { 1, 3, 5, -3, 15 })
void givenNumbers_whenOddCheck_thenVerify(int number) {
    System.out.println("displayName = " + testInfo.getDisplayName());
    assertTrue(oddCheck(number));
}
```

我们应该注意，与单独测试不同，我们不能将`TestInfo` 注入到函数中。这是因为函数参数必须与参数化数据相关。为了解决这个问题，我们需要**通过 `beforeEach`方法** `.`将`TestInfo` 存储在测试类的一个字段中

当我们运行测试时，我们得到测试名称:

```java
displayName = givenNumbers_whenOddCheck_thenVerify5
displayName = givenNumbers_whenOddCheck_thenVerify-3
displayName = givenNumbers_whenOddCheck_thenVerify3
displayName = givenNumbers_whenOddCheck_thenVerify1
displayName = givenNumbers_whenOddCheck_thenVerify15
```

## 3.JUnit 4 方法

**JUnit 4 可以在我们的测试**中填充一个 **`TestName`** 对象。`TestName`是一个 [JUnit 规则](/web/20220524050301/https://www.baeldung.com/junit-4-rules)，规则作为 JUnit 测试执行的一部分被执行，向它们显示当前正在运行的测试的细节。

### 3.1.个别测验

让我们考虑一个单独的测试:

```java
@Rule
public TestName name = new TestName();

@Test
public void givenString_whenSort_thenVerifySortForString() {
    System.out.println("displayName = " + name.getMethodName());
    String s = "abc";
    assertEquals(s, sortCharacters("cba"));
}
```

如上图所示，我们可以使用`TestName` 类的 **`getMethodName` 方法来显示测试**的名称。

让我们进行测试:

```java
displayName = givenString_whenSort_thenVerifySortForString
```

### 3.2.参数化测试

现在让我们使用相同的方法来显示为参数化测试生成的测试名称。首先，我们需要用特殊的测试运行程序来注释测试:

```java
@RunWith(Parameterized.class)
public class JUnit4ParameterizedTestNameUnitTest {
}
```

然后，我们可以使用`TestName` 规则和字段以及构造函数来实现测试，以分配当前测试的参数值:

```java
@Rule
public TestName name = new TestName();
```

```java
private String input;
private String expected;  public JUnit4ParameterizedTestNameUnitTest(String input, String expected) {
    this.input = input;
    this.expected = expected;
}

@Parameterized.Parameters(name = "{0}")
public static Collection<Object[]> suppliedData() {
    return Arrays.asList(new Object[][] { 
      { "abc", "abc" }, { "cba", "abc" }, { "onm", "mno" }, { "a", "a" }, { "zyx", "xyz" }});
}

@Test
public void givenString_whenSort_thenVerifySortForString() {
    System.out.println("displayName = " + name.getMethodName());
    assertEquals(expected, sortCharacters(input));
}
```

在这个测试中，我们提供测试数据`Collection `,它包含输入字符串和预期字符串。这是通过用`@Parameterized.Parameters`注释标注的`suppliedData`函数来完成的。这个注释也允许我们描述测试名称。

当我们运行测试时，`TestName`规则被赋予了每个测试的名称供我们查看:

```java
displayName = givenString_whenSort_thenVerifySortForString[abc]
displayName = givenString_whenSort_thenVerifySortForString[cba]
displayName = givenString_whenSort_thenVerifySortForString[onm]
displayName = givenString_whenSort_thenVerifySortForString[a]
displayName = givenString_whenSort_thenVerifySortForString[zyx]
```

## 4.结论

在本文中，我们讨论了如何在 JUnit 4 和 5 中找到当前测试的名称。

我们看到了如何为单独的测试和参数化的测试做到这一点。

和往常一样，完整的源代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220524050301/https://github.com/eugenp/tutorials/tree/master/testing-modules/junit-5-advanced)