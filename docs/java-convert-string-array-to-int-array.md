# 在 Java 中将字符串数组转换成 int 数组

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-convert-string-array-to-int-array>

## 1.概观

在这个快速教程中，让我们探索如何在 Java 中将数组`String`转换成数组`int`。

## 2.问题简介

首先，我们来看一个`String`数组例子:

```java
String[] stringArray = new String[] { "1", "2", "3", "4", "5", "6", "42" };
```

我们用七根弦创建了`stringArray`。现在，我们需要将`stringArray`转换成一个整数数组:

```java
int[] expected = new int[] { 1, 2, 3, 4, 5, 6, 42 };
```

如上例所示，需求非常简单。然而，在现实世界中，字符串数组可能来自不同的来源，如用户输入或另一个系统。因此，输入数组可能包含一些不是有效数字格式的值，例如:

```java
String[] stringArrayWithInvalidNum = new String[] { "1", "2", "hello", "4", "world", "6", "42" };
```

`“hello”`和`“world”`元素不是有效数字，尽管其他元素是有效数字。通常，当在实际项目中检测到这些类型的值时，我们将遵循特殊的错误处理规则——例如，中止数组转换，将特定的整数作为后备，等等。

在本教程中，**我们将使用 Java 的最小整数作为无效字符串元素**的后备:

```java
int[] expectedWithInvalidInput = new int[] { 1, 2, Integer.MIN_VALUE, 4, Integer.MIN_VALUE, 6, 42 };
```

接下来，让我们从包含所有有效元素的字符串数组开始，然后用错误处理逻辑扩展解决方案。

为了简单起见，我们将使用单元测试断言来验证我们的解决方案是否如预期的那样工作。

## 3.使用流 API

让我们首先使用[流 API](/web/20230103082815/https://www.baeldung.com/java-8-streams) 转换包含所有有效元素的字符串数组:

```java
int[] result = Arrays.stream(stringArray).mapToInt(Integer::parseInt).toArray();
assertArrayEquals(expected, result);
```

正如我们所看到的，`Arrays.stream()`方法将输入字符串数组变成了一个`Stream`。然后，**`mapToInt()`中间操作将我们的流转换成一个`IntStream `对象**。

我们已经使用`Integer.parseInt()`将字符串转换成整数。最后，`toArray()`将`IntStream`对象转换回数组。

因此，接下来，让我们看看无效数字格式场景中的元素。

**假设输入字符串的格式不是一个有效的数字，在这种情况下，`Integer.parseInt()`方法抛出** `**NumberFormatException**.`

因此，我们需要用 lambda 表达式替换`mapToInt()`方法中的[方法引用](/web/20230103082815/https://www.baeldung.com/java-method-references) `Integer::parseInt`，并用[处理 lambda 表达式](/web/20230103082815/https://www.baeldung.com/java-lambda-exceptions)中的`NumberFormatException`异常:

```java
int[] result = Arrays.stream(stringArrayWithInvalidNum).mapToInt(s -> {
    try {
        return Integer.parseInt(s);
    } catch (NumberFormatException ex) {
        // logging ...
        return Integer.MIN_VALUE;
    }
}).toArray();

assertArrayEquals(expectedWithInvalidInput, result);
```

然后，如果我们运行测试，它通过。

如上面的代码所示，我们只改变了`mapToInt()`方法中的实现。

值得一提的是 **Java Stream API 在 Java 8 及以后的版本**上都有。

## 4.在循环中实现转换

我们已经了解了 Stream API 如何解决这个问题。然而，如果我们正在使用一个旧的 Java 版本，我们需要用不同的方法来解决这个问题。

既然我们知道`Integer.parseInt()`完成了主要的转换工作，**我们可以遍历数组中的元素，并对每个字符串元素**调用`Integer.parseInt()`方法:

```java
int[] result = new int[stringArray.length];
for (int i = 0; i < stringArray.length; i++) {
    result[i] = Integer.parseInt(stringArray[i]);
}

assertArrayEquals(expected, result);
```

正如我们在上面的实现中所看到的，我们首先创建一个长度与输入字符串数组相同的整数数组。然后，我们执行转换并在`for`循环中填充结果数组。

接下来，让我们扩展实现以添加错误处理逻辑。类似于流 API 方法，只需**用`try-catch`块包装转换行就可以解决问题**:

```java
int[] result = new int[stringArrayWithInvalidNum.length];
for (int i = 0; i < stringArrayWithInvalidNum.length; i++) {
    try {
        result[i] = Integer.parseInt(stringArrayWithInvalidNum[i]);
    } catch (NumberFormatException exception) {
        // logging ...
        result[i] = Integer.MIN_VALUE;
    }
}

assertArrayEquals(expectedWithInvalidInput, result);
```

如果我们试一试，测试就会通过。

## 5.结论

在本文中，我们通过例子学习了两种将字符串数组转换为整数数组的方法。此外，我们已经讨论了当字符串数组包含无效的数字格式时如何处理转换。

如果我们的 Java 版本是 8 或更高版本，那么 Stream API 将是这个问题最直接的解决方案。否则，我们可以遍历字符串数组，并将每个字符串元素转换为一个整数。

和往常一样，本文中的所有代码片段都可以在 GitHub 上找到[。](https://web.archive.org/web/20230103082815/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-arrays-convert)