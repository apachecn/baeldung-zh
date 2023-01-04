# 将基元数组转换为对象数组

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-primitive-array-to-object-array>

## 1.介绍

在这个简短的教程中，我们将展示如何将原语数组转换为对象数组，反之亦然。

## 2.问题

假设我们有一个原语数组，比如`int[]`，我们想把它转换成一个对象数组`Integer[]`。我们可能会直觉地尝试铸造:

```java
Integer[] integers = (Integer[])(new int[]{0,1,2,3,4});
```

但是，由于不可转换的类型，这将导致编译错误。那是因为 **[自动装箱](/web/20221208143903/https://www.baeldung.com/java-wrapper-classes#autoboxing-and-unboxing)只适用于单个元素**，而不适用于数组或者[集合](/web/20221208143903/https://www.baeldung.com/java-primitive-array-to-list)。

因此，我们需要逐个转换元素。让我们来看几个选项。

## 3.循环

让我们看看如何在迭代中使用自动装箱。

首先，让我们从原始数组转换为对象数组:

```java
int[] input = new int[] { 0, 1, 2, 3, 4 };
Integer[] expected = new Integer[] { 0, 1, 2, 3, 4 };

Integer[] output = new Integer[input.length];
for (int i = 0; i < input.length; i++) {
    output[i] = input[i];
}

assertArrayEquals(expected, output);
```

现在，让我们将对象数组转换为基元数组:

```java
Integer[] input = new Integer[] { 0, 1, 2, 3, 4 };
int[] expected = new int[] { 0, 1, 2, 3, 4 };

int[] output = new int[input.length];
for (int i = 0; i < input.length; i++) {
    output[i] = input[i];
}

assertArrayEquals(expected, output);
```

正如我们所看到的，这一点也不复杂，但是一个可读性更强的解决方案，比如 Stream API，可能更适合我们的需要。

## 4.流

从 Java 8 开始，我们可以使用 [Stream API](/web/20221208143903/https://www.baeldung.com/java-streams) 来编写流畅的代码。

首先，让我们看看如何对原始数组的元素进行装箱:

```java
int[] input = new int[] { 0, 1, 2, 3, 4 };
Integer[] expected = new Integer[] { 0, 1, 2, 3, 4 };

Integer[] output = Arrays.stream(input)
  .boxed()
  .toArray(Integer[]::new);

assertArrayEquals(expected, output);
```

注意`toArray`方法中的`Integer[]::new`参数。如果没有这个参数，流将返回一个`Object[]`，而不是`Integer[]`。

接下来，为了将它们转换回来，我们将使用`mapToInt`方法和`Integer`的拆箱方法:

```java
Integer[] input = new Integer[] { 0, 1, 2, 3, 4 };
int[] expected = new int[] { 0, 1, 2, 3, 4 };

int[] output = Arrays.stream(input)
  .mapToInt(Integer::intValue)
  .toArray();

assertArrayEquals(expected, output);
```

使用 Stream API，我们创建了一个可读性更好的解决方案，但是如果我们仍然希望它更简洁，我们可以尝试一个库，比如 Apache Commons。

## 5.Apache common(Apache 公共)

首先，让我们添加 [Apache Commons Lang](https://web.archive.org/web/20221208143903/https://search.maven.org/artifact/org.apache.commons/commons-lang3) 库作为依赖项:

```java
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-lang3</artifactId>
    <version>3.12.0</version>
</dependency>
```

然后，为了将原语数组转换成它的装箱副本，让我们使用`ArrayUtils.toObject`方法:

```java
int[] input = new int[] { 0, 1, 2, 3, 4 };
Integer[] expected = new Integer[] { 0, 1, 2, 3, 4 };

Integer[] output = ArrayUtils.toObject(input);

assertArrayEquals(expected, output);
```

最后，为了将装箱的元素转换回原语，让我们使用`ArrayUtils.toPrimitives`方法:

```java
Integer[] input = new Integer[] { 0, 1, 2, 3, 4 };
int[] expected = new int[] { 0, 1, 2, 3, 4 };

int[] output = ArrayUtils.toPrimitive(input);

assertArrayEquals(expected, output);
```

Apache Commons Lang 库为我们的问题提供了一个简洁、易用的解决方案，代价是必须添加一个依赖项。

## 6.结论

在本文中，我们研究了几种将原语数组转换为其装箱副本数组的方法，然后，将装箱的元素转换回其原语副本。

和往常一样，这篇文章的代码示例可以在 GitHub 的[上找到。](https://web.archive.org/web/20221208143903/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-oop-types-2)