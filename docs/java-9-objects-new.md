# Java 9 java.util.Objects 新增内容

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-9-objects-new>

## 1。简介

从 1.7 版本开始，class 就成为 Java 的一部分。这个类为对象提供了静态的实用方法，这些方法可以用来执行一些日常任务，比如检查相等性、`null`检查等等。

在本文中，我们将看看 Java 9 的`java.util.Objects`类中引入的新方法。

## 2。`requireNonNullElse`法

该方法接受两个参数，如果不是`null`，则返回第一个参数，否则返回第二个参数。如果两个参数都是`null`，它抛出`NullPointerException`:

```
private List<String> aMethodReturningNullList(){
    return null;
}

@Test
public void givenNullObject_whenRequireNonNullElse_thenElse() {
    List<String> aList = Objects.<List>requireNonNullElse(
      aMethodReturningNullList(), Collections.EMPTY_LIST);

    assertThat(aList, is(Collections.EMPTY_LIST));
}

private List<String> aMethodReturningNonNullList() {
    return List.of("item1", "item2");
}

@Test
public void givenObject_whenRequireNonNullElse_thenObject() {
    List<String> aList = Objects.<List>requireNonNullElse(
      aMethodReturningNonNullList(), Collections.EMPTY_LIST);

    assertThat(aList, is(List.of("item1", "item2")));
}

@Test(expected = NullPointerException.class)
public void givenNull_whenRequireNonNullElse_thenException() {
    Objects.<List>requireNonNullElse(null, null);
}
```

## 3。使用`requireNonNullElseGet`

这个方法类似于`requireNonNullElse`，除了第二个参数**是一个`java.util.function.Supplier`接口，它允许对所提供的集合进行惰性实例化。**`Supplier`实现负责返回一个非空对象，如下所示:

```
@Test
public void givenObject_whenRequireNonNullElseGet_thenObject() {
    List<String> aList = Objects.<List>requireNonNullElseGet(
      null, List::of);
    assertThat(aList, is(List.of()));
}
```

## 4。使用`checkIndex`

此方法用于检查索引是否在给定长度内。如果`0 <= index < length`，则返回索引。否则，它抛出一个`IndexOutOfBoundsException` ，如下所示:

```
@Test
public void givenNumber_whenInvokeCheckIndex_thenNumber() {
    int length = 5;

    assertThat(Objects.checkIndex(4, length), is(4));
}

@Test(expected = IndexOutOfBoundsException.class)
public void givenOutOfRangeNumber_whenInvokeCheckIndex_thenException() {
    int length = 5;
    Objects.checkIndex(5, length);
}
```

## 5。使用`checkFromToIndex`

该方法用于检查`[fromIndex, toIndex)`形成的给定子范围是否在`[0, length)`形成的范围内。如果子范围有效，则它返回下限，如下所示:

```
@Test
public void givenSubRange_whenCheckFromToIndex_thenNumber() {
    int length = 6;

    assertThat(Objects.checkFromToIndex(2,length,length), is(2));
}

@Test(expected = IndexOutOfBoundsException.class)
public void givenInvalidSubRange_whenCheckFromToIndex_thenException() {
    int length = 6;
    Objects.checkFromToIndex(2,7,length);
}
```

注:在数学中，以[a，b]形式表示的范围表示该范围包含 a 且不包含 b，[和]表示包含该数，(和)表示不包含该数。

## 6。使用`checkFromIndexSize`

该方法类似于`checkFromToIndex`，除了我们提供子范围的大小和下限，而不是提供子范围的上限。

在这种情况下，子范围是`[fromIndex, fromIndex + size)`，并且该方法检查子范围是否在由`[0, length)`形成的范围内:

```
@Test
public void givenSubRange_whenCheckFromIndexSize_thenNumber() {
    int length = 6;

    assertThat(Objects.checkFromIndexSize(2,3,length), is(2));
}

@Test(expected = IndexOutOfBoundsException.class)
public void givenInvalidSubRange_whenCheckFromIndexSize_thenException() {
    int length = 6;
    Objects.checkFromIndexSize(2, 6, length);
}
```

## 7。结论

JDK 9 中的`java.util.Objects`类包含了一些新的实用方法。这也是令人鼓舞的，因为这个服务类自从在 Java 7 中引入以来一直在定期更新。

这篇文章的代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221206215338/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-9-improvements)