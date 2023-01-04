# Java 列表不支持操作异常

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-list-unsupported-operation-exception>

## 1.概观

在这个快速教程中，我们将讨论在使用大多数`List`实现的一些 API 时可能出现的一个常见的`Exception`—`UnsupportedOperationException`。

一个`java.util.List`比普通的一个`rray`能支持的功能更多。例如，只需要一个内置的方法调用，就可以检查特定的元素是否在结构内部。这就是为什么我们有时需要将`array`转换成`List`或`Collection`的原因。

关于核心 Java `List` 实现的介绍——`ArrayList`—请参考[这篇文章](/web/20220814123140/https://www.baeldung.com/java-arraylist)。

## 2.`UnsupportedOperationException`

当我们使用来自`java.util.Arrays:`的`asList()`方法时，经常会出现这种错误

```
public static List asList(T... a)
```

它返回:

*   **一个固定大小的`List`作为给定大小的`array`**
*   与原始`array`中的元素类型相同的元素，并且它必须是一个`Object`
*   元素的顺序与原始数组中的顺序相同
*   一个为`serializable`并实现`[RandomAccess](https://web.archive.org/web/20220814123140/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/RandomAccess.html)`的列表

由于 T 是一个 [`varargs`](/web/20220814123140/https://www.baeldung.com/java-varargs) ，我们可以直接将一个数组或者项作为参数传递，方法会创建一个固定大小的初始化列表:

```
List<String> flowers = Arrays.asList("Ageratum", "Allium", "Poppy", "Catmint");
```

我们也可以通过一个实际的`array`:

```
String[] flowers = { "Ageratum", "Allium", "Poppy", "Catmint" };
List<String> flowerList = Arrays.asList(flowers);
```

**由于返回的`List`是固定大小的`List`，我们不能添加/删除元素**。

试图添加更多元素会导致`UnsupportedOperationException`:

```
String[] flowers = { "Ageratum", "Allium", "Poppy", "Catmint" }; 
List<String> flowerList = Arrays.asList(flowers); 
flowerList.add("Celosia");
```

这个`Exception`的根源是返回的对象没有实现`add() `操作，因为它与`java.util.ArrayList.` 不同

**是一个`ArrayList`，从`java.util.Arrays.`到**

获得相同异常的另一种方法是尝试从获得的列表中移除一个元素。

另一方面，如果我们需要的话，有一些方法可以获得一个可变的`List`。

其中之一是直接从`asList()`的结果中创建一个`ArrayList`或任何类型的列表:

```
String[] flowers = { "Ageratum", "Allium", "Poppy", "Catmint" }; 
List<String> flowerList = new ArrayList<>(Arrays.asList(flowers));
```

## 3.结论

总之，重要的是要理解，向列表中添加更多的元素不仅对于不可变列表来说是有问题的。

与往常一样，GitHub 上的[提供了示例的完整源代码。](https://web.archive.org/web/20220814123140/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-list-2)