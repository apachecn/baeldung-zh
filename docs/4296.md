# 在 Java 中将字符数组转换为字符串

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-char-array-to-string>

## 1.概观

在这个快速教程中，我们将介绍在 Java 中将字符数组转换成`String`的各种方法。

## 2.字符串构造函数

`String`类有一个**构造函数，它接受一个`char`数组作为参数:**

```
@Test 
public void whenStringConstructor_thenOK() {
    final char[] charArray = { 'b', 'a', 'e', 'l', 'd', 'u', 'n', 'g' };
    String string = new String(charArray);
    assertThat(string, is("baeldung"));
}
```

这是将`char`数组转换成`String`的最简单的方法之一。它在内部调用 `String#valueOf`来创建一个`String`对象。

## 3.`String.valueOf()`

说到`valueOf(),` ，我们甚至可以直接使用它:

```
@Test
public void whenStringValueOf_thenOK() {
    final char[] charArray = { 'b', 'a', 'e', 'l', 'd', 'u', 'n', 'g' };
    String string = String.valueOf(charArray);
    assertThat(string, is("baeldung"));
}
```

`String#copyValueOf`是另一个语义上等同于`valueOf()`方法的方法，但是只在最初的几个 Java 版本中有意义。从今天起，**`copyValueOf()`方法是多余的，我们不推荐使用它。**

## 4.`StringBuilder`的 `toString()`

如果我们想从一个`char`数组组成一个`String`数组呢？

然后，我们可以首先实例化一个`StringBuilder`实例，并使用它的`append(char[])`方法将所有内容追加在一起。

稍后，我们将使用` toString()`方法来获得它的`String`表示:

```
@Test
public void whenStringBuilder_thenOK() {
    final char[][] arrayOfCharArray = { { 'b', 'a' }, { 'e', 'l', 'd', 'u' }, { 'n', 'g' } };    
    StringBuilder sb = new StringBuilder();
    for (char[] subArray : arrayOfCharArray) {
        sb.append(subArray);
    }
    assertThat(sb.toString(), is("baeldung"));
}
```

我们可以通过实例化我们需要的精确长度的`StringBuilder`来进一步优化上面的代码。

## 5.Java 8 流

使用`Arrays.stream(T[] object)`方法，我们可以在`T`类型的数组上打开一个流。

考虑到我们有一个`Character`数组，**我们可以使用`Collectors.joining()`操作来形成一个`String`实例:**

```
@Test
public void whenStreamCollectors_thenOK() {
    final Character[] charArray = { 'b', 'a', 'e', 'l', 'd', 'u', 'n', 'g' };
    Stream<Character> charStream = Arrays.stream(charArray);
    String string = charStream.map(String::valueOf).collect(Collectors.joining());
    assertThat(string, is("baeldung"));
}
```

这种方法的注意事项是，我们在每个`Character`元素上调用`valueOf()`,所以会非常慢。

## 6.番石榴共基`Joiner`

假设我们需要创建的字符串是一个分隔字符串。番石榴给了我们一个简便的方法:

```
@Test
public void whenGuavaCommonBaseJoiners_thenOK() {
    final Character[] charArray = { 'b', 'a', 'e', 'l', 'd', 'u', 'n', 'g' };
    String string = Joiner.on("|").join(charArray);
    assertThat(string, is("b|a|e|l|d|u|n|g"));
}
```

再次注意，**`join()`方法将只接受一个*字符的*数组，而不是原始的`char`数组。**

## 7.结论

在本教程中，我们探索了将给定的字符数组转换成其在 Java 中的`String` 表示的方法。

像往常一样，所有代码示例都可以在 GitHub 上找到[。](https://web.archive.org/web/20221205161818/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-string-conversions-2)