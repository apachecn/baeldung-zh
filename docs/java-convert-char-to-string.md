# 在 Java 中将字符转换成字符串

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-convert-char-to-string>

## 1。简介

将 c `har`转换为`String`实例是一个非常常见的操作。在本文中，我们将展示处理这种情况的多种方法。

## 2。`String.valueOf()`

`String`类有一个静态方法`valueOf()`，它是为这个特殊的用例设计的。在这里你可以看到它的作用:

```java
@Test
public void givenChar_whenCallingStringValueOf_shouldConvertToString() {
    char givenChar = 'x';

    String result = String.valueOf(givenChar);

    assertThat(result).isEqualTo("x");
}
```

## 3。`Character.toString()`

`Character`类有一个专用的静态`toString()`方法。在这里你可以看到它的作用:

```java
@Test
public void givenChar_whenCallingToStringOnCharacter_shouldConvertToString() {
    char givenChar = 'x';

    String result = Character.toString(givenChar);

    assertThat(result).isEqualTo("x");
}
```

## 4。`Character's`建造者

您还可以实例化`Character`对象并使用标准的`toString()`方法:

```java
@Test
public void givenChar_whenCallingCharacterConstructor_shouldConvertToString() {
    char givenChar = 'x';

    String result = new Character(givenChar).toString();

    assertThat(result).isEqualTo("x");
}
```

## 5。隐投地`String`式

另一种方法是通过类型转换来扩大转换范围:

```java
@Test
public void givenChar_whenConcatenated_shouldConvertToString() {
    char givenChar = 'x';

    String result = givenChar + "";

    assertThat(result).isEqualTo("x");
}
```

## 6。`String.format()`

最后，您可以使用`String.format()`方法:

```java
@Test
public void givenChar_whenFormated_shouldConvertToString() {
    char givenChar = 'x';

    String result = String.format("%c", givenChar);

    assertThat(result).isEqualTo("x");
}
```

## 7。结论

在本文中，我们探索了多种将`char`实例转换为`String`实例的方法。

所有代码示例都可以在 [GitHub](https://web.archive.org/web/20220122045055/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-string-conversions) 资源库中找到。