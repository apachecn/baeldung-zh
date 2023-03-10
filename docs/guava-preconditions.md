# 番石榴的先决条件指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/guava-preconditions>

## 1。概述

在本教程中，我们将展示如何使用谷歌番石榴的`Preconditions`类。

`Preconditions`类提供了一个静态方法列表，用于检查方法或构造函数是否被有效的参数值调用。如果一个前提条件失败，就会抛出一个定制的异常。

## 2。谷歌番石榴的`Preconditions`

`Preconditions`类中的每个静态方法都有三种变体:

*   没有争论。异常在没有错误信息的情况下被抛出
*   一个额外的`Object`参数作为错误消息。异常与错误信息一起抛出
*   一个额外的字符串参数，带有任意数量的额外的`Object`参数，作为带有占位符的错误消息。这有点像`printf`，但是为了 GWT 兼容性和效率，它只允许`%s`指示器

让我们来看看如何使用`Preconditions`类。

### 2.1。Maven 依赖关系

让我们从在`pom.xml`中添加 Google 的番石榴库依赖项开始:

```java
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>31.0.1-jre</version>
</dependency>
```

依赖关系的最新版本可以在[这里](https://web.archive.org/web/20220815032052/https://search.maven.org/classic/#search|gav|1|g%3A%22com.google.guava%22%20AND%20a%3A%22guava%22)查看。

## 3。`checkArgument`

*`Preconditions class`的*方法`checkArgument`保证了传递给调用方法的参数的真实性。这个方法接受一个布尔条件，当条件为假时抛出一个`IllegalArgumentException`。

让我们通过一些例子来看看如何使用这个方法。

### 3.1。没有错误消息

我们可以使用`checkArgument`而不传递任何额外的参数给`checkArgument`方法:

```java
@Test
public void whenCheckArgumentEvaluatesFalse_throwsException() {
    int age = -18;

    assertThatThrownBy(() -> Preconditions.checkArgument(age > 0))
      .isInstanceOf(IllegalArgumentException.class)
      .hasMessage(null).hasNoCause();
}
```

### 3.2。出现错误消息

通过传递一条错误消息，我们可以从`checkArgument`方法中获得一条有意义的错误消息:

```java
@Test
public void givenErrorMsg_whenCheckArgEvalsFalse_throwsException() {
    int age = -18;
    String message = "Age can't be zero or less than zero.";

    assertThatThrownBy(() -> Preconditions.checkArgument(age > 0, message))
      .isInstanceOf(IllegalArgumentException.class)
      .hasMessage(message).hasNoCause();
}
```

### 3.3。带有模板错误信息

通过传递错误消息，我们可以从`checkArgument`方法获得有意义的错误消息和动态数据:

```java
@Test
public void givenTemplateMsg_whenCheckArgEvalsFalse_throwsException() {
    int age = -18;
    String message = "Age should be positive number, you supplied %s.";

    assertThatThrownBy(
      () -> Preconditions.checkArgument(age > 0, message, age))
      .isInstanceOf(IllegalArgumentException.class)
      .hasMessage(message, age).hasNoCause();
} 
```

## 4。`checkElementIndex`

方法`checkElementIndex`检查索引在指定大小的列表、字符串或数组中是否是有效的索引。元素索引的范围可以从 0(包括 0)到大小(不包括 0)。你不直接传递一个列表，字符串或者数组，你只传递它的大小。如果索引不是有效的元素索引，这个方法抛出一个`IndexOutOfBoundsException`,否则它返回一个传递给这个方法的索引。

让我们看看如何使用这个方法，通过在抛出异常时传递一条错误消息来显示来自`checkElementIndex`方法的有意义的错误消息:

```java
@Test
public void givenArrayAndMsg_whenCheckElementEvalsFalse_throwsException() {
    int[] numbers = { 1, 2, 3, 4, 5 };
    String message = "Please check the bound of an array and retry";

    assertThatThrownBy(() -> 
      Preconditions.checkElementIndex(6, numbers.length - 1, message))
      .isInstanceOf(IndexOutOfBoundsException.class)
      .hasMessageStartingWith(message).hasNoCause();
}
```

## 5。`checkNotNull`

方法`checkNotNull`检查作为参数提供的值是否为空。它返回已经检查过的值。如果传递给这个方法的值是 null，那么抛出一个`NullPointerException`。

接下来，我们将通过展示如何通过传递错误消息从`checkNotNull`方法获得有意义的错误消息来展示如何使用该方法:

```java
@Test
public void givenNullString_whenCheckNotNullWithMessage_throwsException () {
    String nullObject = null;
    String message = "Please check the Object supplied, its null!";

    assertThatThrownBy(() -> Preconditions.checkNotNull(nullObject, message))
      .isInstanceOf(NullPointerException.class)
      .hasMessage(message).hasNoCause();
}
```

我们还可以通过向错误消息传递一个参数，从`checkNotNull`方法获得基于动态数据的有意义的错误消息:

```java
@Test
public void whenCheckNotNullWithTemplateMessage_throwsException() {
    String nullObject = null;
    String message = "Please check the Object supplied, its %s!";

    assertThatThrownBy(
      () -> Preconditions.checkNotNull(nullObject, message, 
        new Object[] { null }))
      .isInstanceOf(NullPointerException.class)
      .hasMessage(message, nullObject).hasNoCause();
}
```

## 6。`checkPositionIndex`

方法`checkPositionIndex`检查作为参数传递给该方法的索引是否是指定大小的列表、字符串或数组中的有效索引。位置索引的范围可以从 0 到大小，包括 0 和大小。你不直接传递列表，字符串或者数组，你只传递它的大小。

如果传递的索引不在 0 和给定的大小之间，这个方法抛出一个`IndexOutOfBoundsException`，否则它返回索引值。

让我们看看如何从`checkPositionIndex`方法中获得有意义的错误消息:

```java
@Test
public void givenArrayAndMsg_whenCheckPositionEvalsFalse_throwsException() {
    int[] numbers = { 1, 2, 3, 4, 5 };
    String message = "Please check the bound of an array and retry";

    assertThatThrownBy(
      () -> Preconditions.checkPositionIndex(6, numbers.length - 1, message))
      .isInstanceOf(IndexOutOfBoundsException.class)
      .hasMessageStartingWith(message).hasNoCause();
}
```

## 7。`checkState`

方法`checkState`检查对象状态的有效性，并且不依赖于方法参数。例如，`Iterator`可能会使用它来检查在调用 remove 之前是否调用了 next。如果对象的状态(作为参数传递给该方法的布尔值)处于无效状态，该方法抛出一个`IllegalStateException`。

让我们看看如何使用这个方法，通过在抛出异常时传递一条错误消息来显示来自`checkState`方法的有意义的错误消息:

```java
@Test
public void givenStatesAndMsg_whenCheckStateEvalsFalse_throwsException() {
    int[] validStates = { -1, 0, 1 };
    int givenState = 10;
    String message = "You have entered an invalid state";

    assertThatThrownBy(
      () -> Preconditions.checkState(
        Arrays.binarySearch(validStates, givenState) > 0, message))
      .isInstanceOf(IllegalStateException.class)
      .hasMessageStartingWith(message).hasNoCause();
}
```

## 8。结论

在本教程中，我们展示了来自 Guava 库的`PreConditions`类的方法。`Preconditions`类提供了一个静态方法的集合，这些方法用于验证方法或构造函数是用有效的参数值调用的。

属于上述例子的代码可以在 GitHub 项目中找到——这是一个基于 Maven 的项目，所以它应该很容易导入和运行。