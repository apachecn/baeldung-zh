# JUnit 4 和 JUnit 5 中的断言

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/junit-assertions>

## 1。简介

在本文中，我们将详细探讨 JUnit 中可用的断言。

继[从 JUnit 4 迁移到 JUnit 5](/web/20220830145527/https://www.baeldung.com/junit-5-migration) 和[JUnit 5 指南](/web/20220830145527/https://www.baeldung.com/junit-5)文章之后，我们现在将详细讨论 JUnit 4 和 JUnit 5 中可用的不同断言。

我们还将强调 JUnit 5 对断言的增强。

## 2。断言

**断言是支持测试中断言条件的实用方法**；这些方法可以通过 JUnit 4 中的`Assert` 类和 JUnit 5 中的`Assertions` 类来访问。

为了增加测试和断言本身的可读性，总是建议静态地对各自的类进行`import`。这样，我们可以直接引用断言方法本身，而不用表示类作为前缀。

让我们开始探索 JUnit 4 中可用的断言。

## 3。JUnit 4 中的断言

在这个版本的库中，断言可用于所有原语类型，`Objects,`和`arrays`(原语或`Objects).`)

断言中的参数顺序是期望值后跟实际值；可选地，第一个参数可以是代表被评估条件的消息输出的`String`消息。

在如何定义`assertThat`断言方面只有一点轻微的不同，但是我们将在后面讨论它。

让我们从`assertEquals`开始。

### 3.1。`assertEquals`

`assertEquals`断言验证预期值和实际值是否相等:

```java
@Test
public void whenAssertingEquality_thenEqual() {
    String expected = "Baeldung";
    String actual = "Baeldung";

    assertEquals(expected, actual);
}
```

还可以指定在断言失败时显示的消息:

```java
assertEquals("failure - strings are not equal", expected, actual);
```

### 3.2。`assertArrayEquals`

如果我们想断言两个数组相等，我们可以使用`assertArrayEquals:`

```java
@Test
public void whenAssertingArraysEquality_thenEqual() {
    char[] expected = {'J','u','n','i','t'};
    char[] actual = "Junit".toCharArray();

    assertArrayEquals(expected, actual);
}
```

如果两个数组都是`null`，断言将认为它们相等:

```java
@Test
public void givenNullArrays_whenAssertingArraysEquality_thenEqual() {
    int[] expected = null;
    int[] actual = null;

    assertArrayEquals(expected, actual);
}
```

### 3.3。`assertNotNull`和`assertNull`

当我们想测试一个对象是否为`null`时，我们可以使用`assertNull`断言:

```java
@Test
public void whenAssertingNull_thenTrue() {
    Object car = null;

    assertNull("The car should be null", car);
}
```

相反，如果我们想断言一个对象不应该为空，我们可以使用`assertNotNull assertion.`

### 3.4。`assertNotSame`和`assertSame`

使用`assertNotSame`，可以验证两个变量是否引用同一个对象:

```java
@Test
public void whenAssertingNotSameObject_thenDifferent() {
    Object cat = new Object();
    Object dog = new Object();

    assertNotSame(cat, dog);
}
```

否则，当我们想要验证两个变量引用同一个对象时，我们可以使用`assertSame`断言。

### 3.5。`assertTrue`和`assertFalse`

如果我们想验证某个条件是`true`或`false`，我们可以分别使用`assertTrue`断言或`assertFalse`断言:

```java
@Test
public void whenAssertingConditions_thenVerified() {
    assertTrue("5 is greater then 4", 5 > 4);
    assertFalse("5 is not greater then 6", 5 > 6);
}
```

### 3.6。T3 `fail`

`fail`断言测试失败，抛出`AssertionFailedError`。它可以用来验证是否抛出了一个实际的异常，或者在开发过程中我们想让一个测试失败。

让我们看看如何在第一个场景中使用它:

```java
@Test
public void whenCheckingExceptionMessage_thenEqual() {
    try {
        methodThatShouldThrowException();
        fail("Exception not thrown");
    } catch (UnsupportedOperationException e) {
        assertEquals("Operation Not Supported", e.getMessage());
    }
}
```

### 3.7。`assertThat`

与其他断言相比，`assertThat`断言是 JUnit 4 中唯一一个参数顺序相反的断言。

在这种情况下，断言有一个可选的失败消息、实际值和一个`Matcher`对象。

让我们看看如何使用这个断言来检查数组是否包含特定的值:

```java
@Test
public void testAssertThatHasItems() {
    assertThat(
      Arrays.asList("Java", "Kotlin", "Scala"), 
      hasItems("Java", "Kotlin"));
} 
```

关于对`Matcher`对象的`assertThat`断言的强大使用的附加信息，可在使用 Hamcrest 的[测试中获得。](/web/20220830145527/https://www.baeldung.com/java-junit-hamcrest-guide)

## 4。JUnit 5 断言

JUnit 5 保留了 JUnit 4 的许多断言方法，同时添加了一些利用 Java 8 支持的新方法。

同样在这个版本的库中，断言可用于所有原语类型、`Objects,`和数组(原语或对象)。

断言参数的顺序发生了变化，将输出消息参数作为最后一个参数。由于 Java 8 的支持，输出消息可以是一个`Supplier`，允许对它进行惰性评估。

让我们开始回顾已经有 JUnit 4 等价物的断言。

### 4.1。`assertArrayEquals`

`assertArrayEquals`断言验证预期数组和实际数组是否相等:

```java
@Test
public void whenAssertingArraysEquality_thenEqual() {
    char[] expected = { 'J', 'u', 'p', 'i', 't', 'e', 'r' };
    char[] actual = "Jupiter".toCharArray();

    assertArrayEquals(expected, actual, "Arrays should be equal");
}
```

如果数组不相等，消息“`Arrays should be equal`”将显示为输出。

### 4.2。`assertEquals`

如果我们想要断言两个`floats`相等，我们可以使用简单的`assertEquals`断言:

```java
@Test
void whenAssertingEquality_thenEqual() {
    float square = 2 * 2;
    float rectangle = 2 * 2;

    assertEquals(square, rectangle);
}
```

然而，如果我们想要断言实际值与期望值相差一个预定义的差值，我们仍然可以使用`assertEquals`，但是我们必须将差值作为第三个参数传递:

```java
@Test
void whenAssertingEqualityWithDelta_thenEqual() {
    float square = 2 * 2;
    float rectangle = 3 * 2;
    float delta = 2;

    assertEquals(square, rectangle, delta);
}
```

### 4.3。`assertTrue`和`assertFalse`

使用`assertTrue`断言，可以验证所提供的条件是否为`true`:

```java
@Test
void whenAssertingConditions_thenVerified() {
    assertTrue(5 > 4, "5 is greater the 4");
    assertTrue(null == null, "null is equal to null");
}
```

由于 lambda 表达式的支持，可以为断言提供一个`BooleanSupplier`而不是一个`boolean`条件。

让我们看看如何使用`assertFalse`断言来断言`BooleanSupplier`的正确性:

```java
@Test
public void givenBooleanSupplier_whenAssertingCondition_thenVerified() {
    BooleanSupplier condition = () -> 5 > 6;

    assertFalse(condition, "5 is not greater then 6");
}
```

### 4.4。`assertNull`和`assertNotNull`

当我们想要断言一个对象不是`null`时，我们可以使用`assertNotNull`断言:

```java
@Test
void whenAssertingNotNull_thenTrue() {
    Object dog = new Object();

    assertNotNull(dog, () -> "The dog should not be null");
}
```

相反，我们可以使用`assertNull`断言来检查实际值是否为`null`:

```java
@Test
public void whenAssertingNull_thenTrue() {
    Object cat = null;

    assertNull(cat, () -> "The cat should be null");
}
```

在这两种情况下，失败消息将以一种懒惰的方式被检索，因为它是一个`Supplier`。

### 4.5。`assertSame`和`assertNotSame`

当我们想断言预期和实际指的是同一个`Object`时，我们必须使用`assertSame`断言:

```java
@Test
void whenAssertingSameObject_thenSuccessfull() {
    String language = "Java";
    Optional<String> optional = Optional.of(language);

    assertSame(language, optional.get());
}
```

反过来，我们可以用`assertNotSame`这个。

### 4.6。`fail`

`fail`断言未能通过测试，提供了失败消息以及潜在原因。这有助于在测试开发未完成时对其进行标记:

```java
@Test
public void whenFailingATest_thenFailed() {
    // Test not completed
    fail("FAIL - test not completed");
}
```

### 4.7。`assertAll`

JUnit 5 中引入的新断言之一是`assertAll`。

这个断言允许创建成组的断言，其中所有的断言都被执行，它们的失败被一起报告。具体来说，这个断言接受一个标题，它将包含在`MultipleFailureError`的消息字符串中，以及一个`Executable.`的`Stream`

让我们定义一个分组断言:

```java
@Test
void givenMultipleAssertion_whenAssertingAll_thenOK() {
    Object obj = null;
    assertAll(
      "heading",
      () -> assertEquals(4, 2 * 2, "4 is 2 times 2"),
      () -> assertEquals("java", "JAVA".toLowerCase()),
      () -> assertNull(obj, "obj is null")
    );
}
```

只有当其中一个可执行程序抛出黑名单异常时(例如`OutOfMemoryError`)，分组断言的执行才会被中断。

### 4.8。`assertIterableEquals`

`assertIterableEquals`断言预期的和实际的可重复项完全相等。

为了相等，两个 iterable 必须以相同的顺序返回相等的元素，并且不要求两个 iterable 为相同的类型。

考虑到这一点，让我们看看如何断言两个不同类型的列表(例如`LinkedList`和`ArrayList` )是相等的:

```java
@Test
void givenTwoLists_whenAssertingIterables_thenEquals() {
    Iterable<String> al = new ArrayList<>(asList("Java", "Junit", "Test"));
    Iterable<String> ll = new LinkedList<>(asList("Java", "Junit", "Test"));

    assertIterableEquals(al, ll);
}
```

与`assertArrayEquals`的方式相同，如果两个 iterables 都为空，则认为它们相等。

### 4.9。`assertLinesMatch`

`assertLinesMatch`断言`String`的预期列表与实际列表匹配。

该方法与`assertEquals`和`assertIterableEquals`不同，因为对于每一对预期线和实际线，该方法执行以下算法:

1.  检查预期行是否等于实际行。如果是，则继续下一对
2.  将预期行视为正则表达式，并用`String`执行检查。`matches()`方法。如果是，则继续下一对
3.  检查预期行是否是快进标记。如果是，应用快进并从步骤 1 开始重复算法

让我们看看如何使用这个断言来断言两个列表`String`有匹配的行:

```java
@Test
void whenAssertingEqualityListOfStrings_thenEqual() {
    List<String> expected = asList("Java", "\\d+", "JUnit");
    List<String> actual = asList("Java", "11", "JUnit");

    assertLinesMatch(expected, actual);
}
```

### 4.10。`assertNotEquals`

作为对`assertEquals`的补充，`assertNotEquals`断言断言期望值和实际值不相等:

```java
@Test
void whenAssertingEquality_thenNotEqual() {
    Integer value = 5; // result of an algorithm

    assertNotEquals(0, value, "The result cannot be 0");
}
```

如果两者都是`null`，则断言失败。

### 4.11。`assertThrows`

为了增加简单性和可读性，新的`assertThrows`断言允许我们以一种清晰简单的方式来断言可执行文件是否抛出了指定的异常类型。

让我们看看如何断言抛出的异常:

```java
@Test
void whenAssertingException_thenThrown() {
    Throwable exception = assertThrows(
      IllegalArgumentException.class, 
      () -> {
          throw new IllegalArgumentException("Exception message");
      }
    );
    assertEquals("Exception message", exception.getMessage());
}
```

如果没有抛出异常，或者抛出了不同类型的异常，断言将失败。

### 4.12。`assertTimeout`和`assertTimeoutPreemptively`

如果我们想要断言所提供的`Executable`的执行在给定的`Timeout`之前结束，我们可以使用`assertTimeout`断言:

```java
@Test
void whenAssertingTimeout_thenNotExceeded() {
    assertTimeout(
      ofSeconds(2), 
      () -> {
        // code that requires less than 2 minutes to execute
        Thread.sleep(1000);
      }
    );
}
```

然而，使用`assertTimeout`断言，提供的可执行文件将在调用代码的同一个线程中执行。因此，如果超时，供应商的执行不会被抢先中止。

如果我们想确保一旦超时，可执行文件的执行将被中止，我们可以使用`assertTimeoutPreemptively`断言。

这两个断言都可以接受，而不是一个`Executable,` 一个`ThrowingSupplier`，代表任何返回一个对象并可能抛出一个`Throwable.`的通用代码块

## 5。结论

在本教程中，我们讨论了 JUnit 4 和 JUnit 5 中所有可用的断言。

我们简要强调了 JUnit 5 中的改进，引入了新的断言和对 lambdas 的支持。

和往常一样，本文的完整源代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220830145527/https://github.com/eugenp/tutorials/tree/master/testing-modules/junit5-migration)