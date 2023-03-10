# 向 Java 数组和 ArrayList 添加元素

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-add-element-to-array-vs-list>

## 1.概观

在本教程中，我们将简要地看一下 Java 数组和标准的`ArrayList`[在内存分配方面的异同。此外，我们将看到如何在数组和`ArrayList`中追加和插入元素。](/web/20221205184906/https://www.baeldung.com/java-arraylist)

## 2.Java 数组和数组列表

一个 [Java 数组](/web/20221205184906/https://www.baeldung.com/java-common-array-operations)是该语言提供的基本数据结构。相比之下，`ArrayList`是由数组支持的`List`接口的实现，在 Java 集合框架中提供。

### 2.1.访问和修改元素

我们可以使用方括号符号来访问和修改数组元素:

```java
System.out.println(anArray[1]);
anArray[1] = 4;
```

另一方面，`ArrayList`有一套访问和修改元素的方法:

```java
int n = anArrayList.get(1);
anArrayList.set(1, 4);
```

### 2.2.固定与动态大小

数组和`ArrayList`都以相似的方式分配堆内存，但是不同的是**数组的大小是固定的，而`ArrayList`的大小是动态增加的。**

由于 Java 数组是固定大小的，我们需要在实例化它时提供大小。一旦被实例化，就不可能再增加数组的大小。相反，我们需要用调整后的大小创建一个新数组，并复制前一个数组中的所有元素。

`ArrayList`是`List`接口的可调整大小的数组实现——也就是说，`ArrayList`随着元素的添加而动态增长。当当前元素的数量(包括要添加到`ArrayList`的新元素)大于其底层数组的最大大小时，`ArrayList`增加底层数组的大小。

底层阵列的增长策略取决于`ArrayList`的实施。但是，由于基础数组的大小不能动态增加，因此会创建一个新数组，并将旧数组元素复制到新数组中。

加法运算有一个固定的摊销时间成本。**换句话说，将`n`元素添加到`ArrayList`需要`O(n)`** 时间。

### 2.3.元素类型

根据数组的定义，数组可以包含基元和非基元数据类型。但是， **an `ArrayList`只能包含非原始数据类型**。

当我们将具有原始数据类型的元素插入到一个`ArrayList`中时，Java 编译器会自动将原始数据类型转换成其对应的对象包装类。

现在让我们看看如何在 Java 数组和`ArrayList`中追加和插入元素。

## 3.追加元素

正如我们已经看到的，**数组是固定大小的。**

因此，要追加一个元素，首先，我们需要声明一个比旧数组大的新数组，并将旧数组中的元素复制到新创建的数组中。之后，我们可以将新元素添加到这个新创建的数组中。

让我们看看它在 Java 中的实现，不使用任何实用程序类:

```java
public Integer[] addElementUsingPureJava(Integer[] srcArray, int elementToAdd) {
    Integer[] destArray = new Integer[srcArray.length+1];

    for(int i = 0; i < srcArray.length; i++) {
        destArray[i] = srcArray[i];
    }

    destArray[destArray.length - 1] = elementToAdd;
    return destArray;
}
```

或者， [`Arrays`](/web/20221205184906/https://www.baeldung.com/java-util-arrays) 类提供了一个实用方法`copyOf()`，它帮助创建一个更大的新数组，并从旧数组中复制所有元素:

```java
int[] destArray = Arrays.copyOf(srcArray, srcArray.length + 1);
```

一旦我们创建了一个新的数组，我们可以很容易地将新元素添加到数组中:

```java
destArray[destArray.length - 1] = elementToAdd;
```

另一方面，**在`ArrayList`中添加一个元素是非常容易的**:

```java
anArrayList.add(newElement);
```

## 4.在索引处插入元素

在数组中，在给定索引处插入元素而不丢失之前添加的元素并不是一件简单的任务。

首先，如果数组已经包含了与其大小相等的元素，那么我们首先需要创建一个更大的新数组，并将元素复制到新数组中。

此外，我们需要将指定索引之后的所有元素向右移动一个位置:

```java
public static int[] insertAnElementAtAGivenIndex(final int[] srcArray, int index, int newElement) {
    int[] destArray = new int[srcArray.length+1];
    int j = 0;
    for(int i = 0; i < destArray.length-1; i++) {

        if(i == index) {
            destArray[i] = newElement;
        } else {
            destArray[i] = srcArray[j];
            j++;
        }
    }
    return destArray;
}
```

然而， **[`ArrayUtils`](/web/20221205184906/https://www.baeldung.com/array-processing-commons-lang) 类给了我们一个更简单的解决方案来将项目插入到数组**中:

```java
int[] destArray = ArrayUtils.insert(2, srcArray, 77);
```

我们必须指定要插入值的索引、源数组和要插入的值。

`insert()`方法返回一个包含大量元素的新数组，新元素位于指定的索引处，所有剩余的元素都向右移动一个位置。

注意，`insert()`方法的最后一个参数是一个可变参数，所以我们可以在一个数组中插入任意数量的项。

让我们用它从索引二开始在`srcArray`中插入三个元素:

```java
int[] destArray = ArrayUtils.insert(2, srcArray, 77, 88, 99);
```

并且剩余的元素将向右移动三个位置。

此外，这对于`ArrayList`来说可以很容易地实现:

```java
anArrayList.add(index, newElement);
```

`ArrayList`移动元素并在所需位置插入元素。

## 5.结论

在本文中，我们看了 Java 数组和`ArrayList`。此外，我们还研究了两者之间的异同。最后，我们看到了如何在数组和`ArrayList`中追加和插入元素。

和往常一样，GitHub 上的[提供了工作示例的完整源代码。](https://web.archive.org/web/20221205184906/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-arrays-operations-basic)