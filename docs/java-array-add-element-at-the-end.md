# 扩展数组的长度

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-array-add-element-at-the-end>

## 1。概述

在本教程中，我们将看看扩展 Java [数组](/web/20221126222844/https://www.baeldung.com/java-arrays-guide)的不同方法。

因为数组是一个连续的内存块，所以答案可能不是很明显，但是现在让我们来解开这个谜团。

## 2。使用`Arrays.copyOf`

首先，我们来看看`Arrays.copyOf`。我们将复制数组并向副本中添加一个新元素:

```java
public Integer[] addElementUsingArraysCopyOf(Integer[] srcArray, int elementToAdd) {
    Integer[] destArray = Arrays.copyOf(srcArray, srcArray.length + 1);
    destArray[destArray.length - 1] = elementToAdd;
    return destArray;
}
```

`Arrays.copyOf`的工作方式是，它接受`srcArray`，**将长度参数中指定的元素数量复制到内部创建的新数组**中。新数组的大小是我们提供的参数。

需要注意的一点是，当 length 参数大于源数组的大小时，`Arrays.copyOf`会用`null`填充目的数组中多余的元素。

**根据不同的数据类型，填充的行为会有所不同。**例如，如果我们使用原始数据类型来代替`Integer`，那么额外的元素会用零填充。在`char`的情况下，`Arrays.copyOf`会用`null`填充多余的元素，`boolean,` 会用`false`填充多余的元素。

## 3。使用`ArrayList`

我们接下来要看的是使用`ArrayList.`

我们将首先**将数组转换为** `**ArrayList**` ，然后添加元素。然后我们将**将`ArrayList`转换回数组**:

```java
public Integer[] addElementUsingArrayList(Integer[] srcArray, int elementToAdd) {
    Integer[] destArray = new Integer[srcArray.length + 1];
    ArrayList<Integer> arrayList = new ArrayList<>(Arrays.asList(srcArray));
    arrayList.add(elementToAdd);
    return arrayList.toArray(destArray);
}
```

请注意，我们已经通过将`srcArray `转换为`Collection. `来传递它，`srcArray`**将在** `**ArrayList**.`中填充底层数组

另外，另一件要注意的事情是，我们已经将目标数组作为参数传递给了`toArray`。这个方法将**的底层数组复制到`destArray`的**中。

## 4。使用`System.arraycopy`

最后，我们来看看`System.arraycopy`，它与`Arrays.copyOf`非常相似:

```java
public Integer[] addElementUsingSystemArrayCopy(Integer[] srcArray, int elementToAdd) {
    Integer[] destArray = new Integer[srcArray.length + 1];
    System.arraycopy(srcArray, 0, destArray, 0, srcArray.length);
    destArray[destArray.length - 1] = elementToAdd;
    return destArray;
}
```

一个有趣的事实是 **`Arrays.copyOf `内部使用了这种方法。**

这里我们可以注意到，我们**将元素从`srcArray`复制到`destArray`** ，然后**将新元素**添加到`destArray`。

## 5.表演

所有解决方案的一个共同点是，我们必须以某种方式创建一个新的数组。其原因在于数组在内存中的分配方式。一个数组包含一个**连续的内存块**用于超快速查找，这就是为什么我们不能简单地调整它的大小。

这当然会对性能产生影响，尤其是对于大型阵列。这就是 [`ArrayList`](/web/20221126222844/https://www.baeldung.com/java-arraylist) 过度分配的原因，有效减少了 JVM 需要重新分配内存的次数。

但是，如果我们正在做大量的插入，数组可能不是正确的数据结构，我们应该考虑一个 [`LinkedList`](/web/20221126222844/https://www.baeldung.com/java-linkedlist) 。

## 6。结论

在本文中，我们探讨了在数组末尾添加元素的不同方法。

和往常一样，完整的代码可以在 GitHub 上获得[。](https://web.archive.org/web/20221126222844/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-arrays-operations-basic)