# Arrays.asList()和 List.of()之间的区别

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-arrays-aslist-vs-list-of>

## 1.概观

有时候在 Java 中，为了方便起见，我们需要创建一个小列表，或者将一个数组转换成一个列表。Java 为此提供了一些助手方法。

在本教程中，我们将比较初始化小型特设数组的两种主要方式:`List.of()`和`Array.asList().`

## 2.使用`Arrays.asList()`

Java 1.2 中引入的 [`Arrays.asList()`](/web/20220824220104/https://www.baeldung.com/java-arraylist) ，简化了`List`对象的创建，是`Java` `Collections Framework`的一部分。它可以接受一个数组作为输入，并创建所提供数组的`List`对象:

```java
Integer[] array = new Integer[]{1, 2, 3, 4};
List<Integer> list = Arrays.asList(array);
assertThat(list).containsExactly(1,2,3,4);
```

正如我们所见，创建一个简单的`Integers`的`List`非常容易。

### 2.1.返回列表上不支持的操作

方法`asList()`返回一个固定大小的列表。因此，添加和删除新元素会抛出一个`UnsupportedOperationException`:

```java
List<Integer> list = Arrays.asList(1, 2, 3, 4, 5);
assertThrows(UnsupportedOperationException.class, () -> list.add(6));

List<Integer> list = Arrays.asList(1, 2, 3, 4, 5);
assertThrows(UnsupportedOperationException.class, () -> list.remove(1));
```

### 2.2.使用数组

我们应该注意到这个列表并没有创建输入数组的副本。相反，它用`List`接口包装原始数组。因此，对数组的更改也会反映在列表中:

```java
Integer[] array = new Integer[]{1,2,3};
List<Integer> list = Arrays.asList(array);
array[0] = 1000;
assertThat(list.get(0)).isEqualTo(1000);
```

### 2.3.更改返回的列表

另外，`Arrays.asList()`返回的列表**是可变的**。也就是说，我们可以更改列表中的单个元素:

```java
List<Integer> list = Arrays.asList(1, 2, 3, 4);
list.set(1, 1000);
assertThat(list.get(1)).isEqualTo(1000);
```

最终，这会导致不希望的副作用，导致难以发现的错误。当数组作为输入提供时，列表上的更改也将反映在数组上:

```java
Integer[] array = new Integer[]{1, 2, 3};
List<Integer> list = Arrays.asList(array);
list.set(0,1000);
assertThat(array[0]).isEqualTo(1000);
```

让我们看看创建列表的另一种方法。

## 3.使用`List.of()`

与`Arrays.` `asList()`相反，Java 9 引入了一个更方便的方法， [`List.of()`](/web/20220824220104/https://www.baeldung.com/java-init-list-one-line#factory-methods-java-9) 。这将创建不可修改的`List`对象的实例:

```java
String[] array = new String[]{"one", "two", "three"};
List<String> list = List.of(array);
assertThat(list).containsExactly("two", "two", "three");
```

### 3.1.与`Arrays.asList()`的差异

与`Arrays.asList()`的主要区别在于 **`List.of()`返回一个不可变的** **列表，它是所提供的输入** **数组的副本。**因此，对原始数组的更改不会反映在返回的列表中:

```java
String[] array = new String[]{"one", "two", "three"};
List<String> list = List.of(array);
array[0] = "thousand";
assertThat(list.get(0)).isEqualTo("one");
```

此外，我们不能修改列表的元素。如果我们尝试，它会抛出`UnsupportedOperationException`:

```java
List<String> list = List.of("one", "two", "three");
assertThrows(UnsupportedOperationException.class, () -> list.set(1, "four"));
```

### 3.2.空值

我们还应该注意到，`List.of()`不允许将`null`值作为输入，并将抛出一个`NullPointerException`:

```java
assertThrows(NullPointerException.class, () -> List.of("one", null, "two"));
```

## 4.结论

这篇短文探索了使用`List.of()`和`Arrays.asList()`在 Java 中创建列表。

和往常一样，完整的示例代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20220824220104/https://github.com/harry9656/tutorials/tree/master/core-java-modules/core-java-collections-list-4)