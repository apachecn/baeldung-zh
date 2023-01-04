# Java 中的数组操作

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-common-array-operations>

## 1.概观

任何 Java 开发人员都知道，在处理数组操作时，产生一个干净、高效的解决方案并不总是容易实现的。尽管如此，它们仍然是 Java 生态系统中的核心部分——我们将不得不在多个场合与它们打交道。

出于这个原因，最好有一份“备忘单”——一份帮助我们快速解决难题的最常见程序的总结。本教程将在这些情况下派上用场。

## 2.数组和助手类

在继续之前，理解什么是 Java 中的数组以及如何使用它是很有用的。如果这是你第一次在 Java 中使用它，我们建议看一看[这篇之前的文章](/web/20220630132756/https://www.baeldung.com/java-arrays-guide)，在那里我们讨论了所有的基本概念。

请注意，阵列支持的基本操作在某种程度上是有限的。对于数组来说，用复杂的算法来执行相对简单的任务并不少见。

**由于这个原因，对于我们的大多数操作，我们将使用助手类和方法来帮助我们:Java 提供的 [`Arrays`](/web/20220630132756/https://www.baeldung.com/java-util-arrays) 类和 Apache 的 [`ArrayUtils`](/web/20220630132756/https://www.baeldung.com/array-processing-commons-lang) 类。**

为了在我们的项目中包含后者，我们必须添加 [Apache Commons](https://web.archive.org/web/20220630132756/https://commons.apache.org/) 依赖项:

```java
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-lang3</artifactId>
    <version>3.12.0</version>
</dependency>
```

我们可以在 Maven Central 上查看这个工件的最新版本[。](https://web.archive.org/web/20220630132756/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.apache.commons%22%20AND%20a%3A%22commons-lang3%22)

## 3.获取数组的第一个和最后一个元素

由于数组按索引访问的特性，这是最常见和最简单的任务之一。

让我们从声明和初始化一个`int`数组开始，这个数组将在我们所有的例子中使用(除非我们另外指定):

```java
int[] array = new int[] { 3, 5, 2, 5, 14, 4 };
```

知道数组的第一项与索引值 0 相关联，并且它有一个我们可以使用的`length`属性，那么就很容易弄清楚我们如何获得这两个元素:

```java
int firstItem = array[0];
int lastItem = array[array.length - 1];
```

## 4.从数组中获取一个随机值

通过使用`java.util.Random`对象，我们可以很容易地从数组中获得任何值:

```java
int anyValue = array[new Random().nextInt(array.length)];
```

## 5.向数组追加新项

众所周知，数组保存固定大小的值。所以不能随便加一项就超过这个限度。

我们首先需要声明一个新的更大的数组，并将基本数组的元素复制到第二个数组中。

幸运的是，`Arrays`类提供了一种简便的方法来将数组的值复制到一个新的不同大小的结构中:

```java
int[] newArray = Arrays.copyOf(array, array.length + 1);
newArray[newArray.length - 1] = newItem;
```

**可选地，如果`ArrayUtils`类在我们的项目中是可访问的，我们可以使用它的`add method `(或者它的`addAll`替代)来在一行语句中完成我们的目标:**

```java
int[] newArray = ArrayUtils.add(array, newItem);
```

我们可以想象，这个方法并不修改原来的`array`对象；我们必须将它的输出赋值给一个新的变量。

## 6.在两个值之间插入一个值

由于索引值的特性，在数组中的两个元素之间插入一个元素并不是一件简单的工作。

Apache 认为这是一个典型的场景，并在其`ArrayUtils`类中实现了一个方法来简化解决方案:

```java
int[] largerArray = ArrayUtils.insert(2, array, 77);
```

我们必须指定想要插入值的索引，输出将是一个包含大量元素的新数组。

最后一个参数是一个可变参数(也叫 T0)，因此我们可以在数组中插入任意数量的元素。

## 7.比较两个数组

尽管数组是`Object`的，因此提供了一个`equals`方法，但是它们使用它的默认实现，只依赖于引用相等。

无论如何，我们可以调用`java.util.Arrays` ' `equals`方法来检查两个数组对象是否包含相同的值:

```java
boolean areEqual = Arrays.equals(array1, array2);
```

注意:该方法对[交错数组](/web/20220630132756/https://www.baeldung.com/java-jagged-arrays)无效。验证多维结构相等的合适方法是`Arrays.deepEquals`方法。

## 8.检查数组是否为空

考虑到我们可以使用数组的`length`属性，这是一个简单的赋值:

```java
boolean isEmpty = array == null || array.length == 0;
```

此外，我们还可以在`ArrayUtils` helper 类中使用一个空安全的方法:

```java
boolean isEmpty = ArrayUtils.isEmpty(array);
```

**这个函数仍然依赖于数据结构的长度，数据结构也将空值和空子数组视为有效值，所以我们必须关注这些边缘情况:**

```java
// These are empty arrays
Integer[] array1 = {};
Integer[] array2 = null;
Integer[] array3 = new Integer[0];

// All these will NOT be considered empty
Integer[] array3 = { null, null, null };
Integer[][] array4 = { {}, {}, {} };
Integer[] array5 = new Integer[3];
```

## 9.如何打乱一个数组的元素

为了打乱数组中的项目，我们可以使用`ArrayUtil`的特性:

```java
ArrayUtils.shuffle(array);
```

**这是一个`void`方法，对数组的实际值进行操作。**

## 10.装箱和取消装箱数组

我们经常遇到只支持基于`Object`的数组的方法。

再次，`ArrayUtils` helper 类可以方便地获得原始数组的装箱版本:

```java
Integer[] list = ArrayUtils.toObject(array);
```

相反的操作也是可能的:

```java
Integer[] objectArray = { 3, 5, 2, 5, 14, 4 };
int[] array = ArrayUtils.toPrimitive(objectArray);
```

## 11.从数组中删除重复项

**删除重复的最简单的方法是将数组转换成一个`Set`实现。**

我们可能知道，`Collection` s 使用泛型，因此不支持基本类型。

出于这个原因，如果我们不像在我们的例子中那样处理基于对象的数组，我们将首先需要装箱我们的值:

```java
// Box
Integer[] list = ArrayUtils.toObject(array);
// Remove duplicates
Set<Integer> set = new HashSet<Integer>(Arrays.asList(list));
// Create array and unbox
return ArrayUtils.toPrimitive(set.toArray(new Integer[set.size()]));
```

注意:我们也可以使用[其他技术在数组和`Set`对象](/web/20220630132756/https://www.baeldung.com/convert-array-to-set-and-set-to-array)之间进行转换。

同样，如果我们需要保持元素的顺序，我们必须使用不同的`Set`实现，比如`LinkedHashSet`。

## 12.如何打印数组

与`equals`方法相同，数组的`toString`函数使用由`Object`类提供的默认实现，这不是很有用。

`Arrays`和`ArrayUtils `类都附带了将数据结构转换成可读的`String`的实现。

除了它们使用的格式略有不同，最重要的区别是它们如何处理多维对象。

Java Util 的类提供了两个我们可以使用的静态方法:

*   `toString`:不适用于交错数组
*   `deepToString`:支持任何基于`Object`的数组，但不使用原始数组参数编译

另一方面， **Apache 的实现提供了一个在任何情况下都能正确工作的`toString`方法:**

```java
String arrayAsString = ArrayUtils.toString(array);
```

## 13.将数组映射到另一种类型

对所有数组项应用操作通常很有用，可能会将它们转换为另一种类型的对象。

考虑到这个目标，**我们将尝试使用泛型创建一个灵活的助手方法:**

```java
public static <T, U> U[] mapObjectArray(
  T[] array, Function<T, U> function,
  Class<U> targetClazz) {
    U[] newArray = (U[]) Array.newInstance(targetClazz, array.length);
    for (int i = 0; i < array.length; i++) {
        newArray[i] = function.apply(array[i]);
    }
    return newArray;
}
```

如果我们在项目中不使用 Java 8，我们可以丢弃`Function`参数，并为我们需要执行的每个映射创建一个方法。

我们现在可以为不同的操作重用我们的泛型方法。让我们创建两个测试用例来说明这一点:

```java
@Test
public void whenMapArrayMultiplyingValues_thenReturnMultipliedArray() {
    Integer[] multipliedExpectedArray = new Integer[] { 6, 10, 4, 10, 28, 8 };
    Integer[] output = 
      MyHelperClass.mapObjectArray(array, value -> value * 2, Integer.class);

    assertThat(output).containsExactly(multipliedExpectedArray);
}

@Test
public void whenMapDividingObjectArray_thenReturnMultipliedArray() {
    Double[] multipliedExpectedArray = new Double[] { 1.5, 2.5, 1.0, 2.5, 7.0, 2.0 };
    Double[] output =
      MyHelperClass.mapObjectArray(array, value -> value / 2.0, Double.class);

    assertThat(output).containsExactly(multipliedExpectedArray);
}
```

对于原始类型，我们必须首先对我们的值进行装箱。

**作为替代，我们可以求助于 [Java 8 的流](/web/20220630132756/https://www.baeldung.com/java-8-streams-introduction)来为我们进行映射。**

我们需要首先将数组转换成一个由`Object`组成的`Stream`。我们可以用`Arrays.stream`方法做到这一点。

例如，如果我们想要将我们的`int`值映射到一个自定义的`String`表示，我们将实现这个:

```java
String[] stringArray = Arrays.stream(array)
  .mapToObj(value -> String.format("Value: %s", value))
  .toArray(String[]::new);
```

## 14.筛选数组中的值

从集合中筛选出值是一项常见的任务，我们可能需要在不止一种情况下执行这项任务。

这是因为在我们创建接收值的数组时，我们不能确定它的最终大小。因此，**我们将再次依赖于`Stream` s 方法。**

假设我们想从数组中删除所有奇数:

```java
int[] evenArray = Arrays.stream(array)
  .filter(value -> value % 2 == 0)
  .toArray();
```

## 15.其他常见的数组操作

当然，我们可能需要执行许多其他的数组操作。

除了本教程中显示的操作，我们还在专门的帖子中广泛讨论了其他操作:

*   [检查 Java 数组是否包含值](/web/20220630132756/https://www.baeldung.com/java-array-contains-value)
*   [如何在 Java 中复制数组](/web/20220630132756/https://www.baeldung.com/java-array-copy)
*   [删除数组的第一个元素](/web/20220630132756/https://www.baeldung.com/java-array-remove-first-element)
*   [用 Java 寻找数组中的最小值和最大值](/web/20220630132756/https://www.baeldung.com/java-array-min-max)
*   [在 Java 数组中求总和与平均值](/web/20220630132756/https://www.baeldung.com/java-array-sum-average)
*   [如何在 Java 中反转数组](/web/20220630132756/https://www.baeldung.com/java-invert-array)
*   [在 Java 中连接和拆分数组和集合](/web/20220630132756/https://www.baeldung.com/java-join-and-split)
*   [在 Java 中组合不同类型的集合](/web/20220630132756/https://www.baeldung.com/java-combine-collections)
*   [找出一个数组中所有加起来等于给定总和的数字对](/web/20220630132756/https://www.baeldung.com/java-algorithm-number-pairs-sum)
*   [Java 中的排序](/web/20220630132756/https://www.baeldung.com/java-sorting)
*   [Java 中高效的词频计算器](/web/20220630132756/https://www.baeldung.com/java-word-frequency)
*   [Java 中的插入排序](/web/20220630132756/https://www.baeldung.com/java-insertion-sort)

## 16.结论

数组是 Java 的核心功能之一，因此理解它们是如何工作的，以及知道我们能用它们做什么和不能做什么是非常重要的。

在本教程中，我们学习了如何在常见场景中恰当地处理数组操作。

和往常一样，工作示例的完整源代码可以在我们的 Github repo 上获得。