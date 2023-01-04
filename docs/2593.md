# 在 JUnit 4 中有条件地运行或忽略测试

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/junit-conditional-assume>

## 1.概观

假设我们有一个依赖于操作系统的代码测试，并且只有在我们的测试机器运行在 Linux 上时才运行。如果它运行在任何其他操作系统上，我们希望测试不会失败，而是在运行时被忽略。

第一种方法是使用几个`if`语句，通过`System`类属性来检查这个条件。这当然是可行的，但是 JUnit 有一个更干净、更优雅的方法。

在这个简短的教程中，我们将看看如何使用`[Assume](https://web.archive.org/web/20221128054937/https://junit.org/junit4/javadoc/4.12/org/junit/Assume.html) `类在 [JUnit](/web/20221128054937/https://www.baeldung.com/junit) 4 中有条件地运行或忽略测试。

## 2.`Assume`类

**这个类提供了一套方法来支持基于特定条件的条件测试执行**。只有满足所有这些条件，我们的测试才会运行。如果没有， **JUnit 将跳过它的执行，并在测试报告**中将其标记为通过。后者是与 [`Assert`](/web/20221128054937/https://www.baeldung.com/junit-assertions#assertions-junit4) 类的主要区别，其中失败条件导致测试以`failing`结束。

需要注意的重要一点是，**我们为`Assume`类描述的行为是默认的 JUnit runner** 独有的。有了[定制跑腿](/web/20221128054937/https://www.baeldung.com/junit-4-custom-runners)，事情可能就不一样了。

最后，和使用`Assert`一样，我们可以在`@Before`或`@BeforeClass`带注释的方法中或者在`@Test `方法本身中调用`Assume`方法。

现在让我们通过展示一些例子来了解一下`Assume`类最有用的方法。对于下面所有的例子，让我们假设`getOsName()`返回`Linux.`

### 2.1.使用`assumeThat`

`assumeThat()` 方法检查状态——在本例中是`getOsName()`——是否满足传入的匹配器的条件:

```
@Test
public void whenAssumeThatAndOSIsLinux_thenRunTest() {
    assumeThat(getOsName(), is("Linux"));

    assertEquals("run", "RUN".toLowerCase());
}
```

在这个例子中，**我们检查了`getOsName()`是否等于`Linux`。随着`getOsName()`返回`Linux`，测试将运行**。注意，我们在这里使用 [Hamcrest 匹配器](/web/20221128054937/https://www.baeldung.com/hamcrest-core-matchers)方法`is(T)` 作为匹配器。

### 2.2。使用`assumeTrue`

类似地，我们可以使用`assumeTrue()`方法来指定一个布尔表达式，这个布尔表达式必须计算为`true`才能运行测试。如果评估为`false`，测试将被忽略:

```
private boolean isExpectedOS(String osName) {
    return "Linux".equals(osName);
}

@Test 
public void whenAssumeTrueAndOSIsLinux_thenRunTest() {
    assumeTrue(isExpectedOS(getOsName()));

    assertEquals("run", "RUN".toLowerCase());
} 
```

在本例中， **`isExpectedOs()`返回`true`** 。因此，**测试运行的** **条件已经满足，测试将运行**。

### 2.3.使用`assumeFalse`

最后，我们可以使用相反的`assumeFalse()`方法来指定一个布尔表达式，这个布尔表达式的值必须为*假*，这样测试才能运行。如果评估结果为`true`，测试将被忽略:

```
@Test
public void whenAssumeFalseAndOSIsLinux_thenIgnore() {
    assumeFalse(isExpectedOS(getOsName()));

    assertEquals("run", "RUN".toLowerCase());
}
```

在这种情况下，由于`isExpectedOs()` **也** **返回** `**true,** ` **测试运行的** **条件没有满足，测试将被忽略**。

### 2.4.使用`assumeNotNull`

当我们想忽略某个表达式为`null,` 的测试时，我们可以使用 `assumeNotNull()`方法:

```
@Test
public void whenAssumeNotNullAndNotNullOSVersion_thenRun() {
    assumeNotNull(getOsName());

    assertEquals("run", "RUN".toLowerCase());
}
```

由于`getOsName()`正在返回一个非空值，测试运行的条件已经满足，测试将会运行。

### 2.5.使用`assumeNoException`

最后，如果抛出异常，我们可能希望忽略测试。为此，我们可以使用`assumeNoException()`:

```
@Test
public void whenAssumeNoExceptionAndExceptionThrown_thenIgnore() {
    assertEquals("everything ok", "EVERYTHING OK".toLowerCase());
    String t=null;
    try {
        t.charAt(0);
    } catch(NullPointerException npe){
        assumeNoException(npe);
    }
    assertEquals("run", "RUN".toLowerCase());
}
```

在本例中，由于`t`是`null,` **，抛出了`NullPointerException`异常，因此** **测试运行的** **条件不满足，测试将被忽略**。

## 3.我们应该将`assumeXXX`呼叫放在哪里？

重要的是要注意到**`assumeXXX`方法的行为取决于我们在测试中把它们放在什么地方**。

让我们稍微修改一下我们的`assumeThat`例子，让`assertEquals()`调用先进行。还有，让我们让 `assertEquals()`失败:

```
@Test
public void whenAssumeFalseAndOSIsLinux_thenIgnore() {
    assertEquals("run", "RUN");
    assumeFalse(isExpectedOS(getOsName()));
} 
```

当我们运行这个例子时，我们将有:

```
org.junit.ComparisonFailure: 
Expected :run
Actual   :RUN
```

在这种情况下，**我们的测试不会被忽略，因为它在我们到达 `assumeThat()`调用之前就已经失败了。**所有的`assumeXXX`方法都会发生同样的情况。因此，我们需要**确保我们将它们放在测试方法**中的正确位置。

## 4.结论

在这个简短的教程中，我们已经看到了如何使用 JUnit 4 中的`Assume`类有条件地决定一个测试是否应该运行。**如果我们使用的是 JUnit 5，它在 5.4 或更高版本中也是可用的**。

和往常一样，我们所经历的例子的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221128054937/https://github.com/eugenp/tutorials/tree/master/testing-modules/junit-4)