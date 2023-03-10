# arrays . aslist()vs collections . singletonlist()

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-aslist-vs-singletonlist>

## 1.概观

`[List](/web/20220921120138/https://www.baeldung.com/tag/java-list/)`是我们使用 Java 时常用的集合类型。

我们知道，我们可以很容易地用一行初始化一个`List`[。例如，当我们想要初始化一个只有一个元素的`List`时，我们可以使用](/web/20220921120138/https://www.baeldung.com/java-init-list-one-line) [`Arrays.asList()`](https://web.archive.org/web/20220921120138/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Arrays.html#asList(T...)) 方法或者 [`Collections.singletonList()`](https://web.archive.org/web/20220921120138/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Collections.html#singletonList(T)) 方法。

在本教程中，我们将讨论这两种方法之间的区别。然后，为了简单起见，我们将使用单元测试断言来验证一些操作是否如预期的那样运行。

## 2.`Arrays.asList()`法

首先，**`Arrays.asList()`方法返回一个固定大小的列表**。

任何结构上的改变都会抛出`UnsupportedOperationException`，比如向列表中添加新元素或者从列表中移除元素。现在，让我们用一个测试来检验它:

```java
List<String> arraysAsList = Arrays.asList("ONE");
assertThatExceptionOfType(UnsupportedOperationException.class).isThrownBy(
    () -> arraysAsList.add("TWO")
); 
```

如果我们试一试，测试就会通过。在上面的代码中，我们使用了 [Assertj 的异常断言](/web/20220921120138/https://www.baeldung.com/assertj-exception-assertion)来验证当我们试图向列表中添加新元素时`UnsupportedOperationException`是否被抛出。

尽管我们不能在列表上调用`add()`或`remove()`操作，但是**我们可以使用`set()`方法**改变列表中的元素:

```java
arraysAsList.set(0, "A brand new string");
assertThat(arraysAsList.get(0)).isEqualTo("A brand new string");
```

这一次，我们用一个新的`String`对象来设置列表中的元素。如果我们执行测试，它就通过了。

最后，我们来讨论一下`Arrays.asList()`方法的数组和返回列表的关系。

顾名思义，该方法使数组作为`List`工作。让我们来理解“让一个数组像一个`List`一样工作”是什么意思。

**`Arrays.asList()`方法返回一个`List`对象，该对象由给定的数组**支持。也就是说，该方法不会将数组中的元素复制到新的`List`对象中。相反，该方法在给定的数组上提供了一个`List`视图。因此，我们对数组所做的任何更改都将在返回的列表中可见。类似地，对列表所做的更改也将在数组中可见:

```java
String[] theArray = new String[] { "ONE", "TWO" };
List<String> theList = Arrays.asList(theArray);
//changing the list, the array is changed too
theList.set(0, "ONE [changed in list]");
assertThat(theArray[0]).isEqualTo("ONE [changed in list]");

//changing the array, the list is changed too
theArray[1] = "TWO [changed in array]";
assertThat(theList.get(1)).isEqualTo("TWO [changed in array]"); 
```

测试通过。所以对于数组和返回的列表，如果我们在一边做了一些改变，另一边也会改变。

## 3.`Collections.singletonList()`法

首先，`singletonList()`方法返回的列表只有一个元素。与`Arrays.asList()`方法不同， **`singletonList()`返回一个不可变的列表**。

换句话说，不允许对由`singletonList().` 返回的列表进行结构性和非结构性的更改。一个测试可以快速说明这一点:

```java
List<String> singletonList = Collections.singletonList("ONE");
assertThatExceptionOfType(UnsupportedOperationException.class).isThrownBy(
    () -> singletonList.add("TWO")
);
assertThatExceptionOfType(UnsupportedOperationException.class).isThrownBy(
    () -> singletonList.set(0, "A brand new string")
); 
```

如果我们进行测试，就会通过。因此，无论我们是向列表中添加元素还是改变列表中的元素，它都会抛出`UnsupportedOperationException.`

值得一提的是，如果我们看一下返回列表类的源代码，与其他`List`实现不同，返回列表中的单个元素没有存储在数组或任何其他复杂的数据结构中。相反，**list 直接保存元素对象**:

```java
private static class SingletonList<E> extends AbstractList<E> implements RandomAccess, Serializable {
    ...
    private final E element;

    SingletonList(E obj) {element = obj;}
    ...
}
```

因此，它将占用更少的内存。

## 4.简短的摘要

最后，让我们用表格总结一下`Arrays.asList()`方法和`Collections.singletonList()`方法的特点，以便更好地了解:

|  | `Arrays.asList()` | `Collections.singletonList()` |
| **结构变化** | 不允许 | 不允许 |
| **非结构性变化** | 允许 | 不允许 |
| **数据结构** | 由阵列支持 | 直接握住元素 |

## 5.结论

在这篇简短的文章中，我们讨论了`Arrays.asList()`方法和`Collections.singletonList()`方法。

当我们想要初始化一个只有一个元素的固定大小的列表时，我们可以考虑使用`Collections.singletonList()`方法。但是，如果需要改变返回列表中的元素，我们可以选择`Arrays.asList()`方法。

与往常一样，GitHub 上的[提供了示例的完整源代码。](https://web.archive.org/web/20220921120138/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-list-4)