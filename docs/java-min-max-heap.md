# 如何在 Java 中实现最小最大堆

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-min-max-heap>

## 1.概观

在本教程中，我们将看看如何在 Java 中实现 min-max [heap](/web/20220628062856/https://www.baeldung.com/cs/heap-vs-binary-search-tree) 。

## 2.最小-最大堆

首先，我们来看看 heap 的定义和特点。最小-最大堆是一个完整的[二叉树](/web/20220628062856/https://www.baeldung.com/cs/binary-tree-intro)，同时具有最小堆和最大堆的特征:

[![](img/4effdd936ec614ae2f34b2df695f4c64.png)](/web/20220628062856/https://www.baeldung.com/wp-content/uploads/2021/06/Screenshot-from-2021-06-14-22-37-11.png)

正如我们在上面看到的，**树中偶数层的每个节点都小于它的所有后代，而奇数层的每个节点都大于它的所有后代，其中根在零层。**

最小-最大堆中的每个节点都有一个数据成员，通常称为键。在 min-max 堆中，`root`的`key`最小，第二层的两个节点中有一个是最大的`key`。对于最小-最大堆中的每个节点，如`X`:

*   如果`X`在最小(或偶数)层，那么`X.key`是根为`X`的子树中所有键中最小的键
*   如果`X`在最大(或奇数)级别，那么`X.key`是根为`X`的子树中所有关键字中最大的关键字

像最小堆或最大堆一样，插入和删除可以发生在`O(logN)`的[时间复杂度](/web/20220628062856/https://www.baeldung.com/cs/time-vs-space-complexity)内。

## 3.用 Java 实现

让我们从一个简单的表示最小-最大堆的类开始:

```java
public class MinMaxHeap<T extends Comparable<T>> {
    private List<T> array;
    private int capacity;
    private int indicator;
}
```

正如我们在上面看到的，我们使用一个`indicator`来计算添加到数组中的最后一个项目索引。但是在我们继续之前，我们需要记住**数组索引从零开始，但是我们假设索引从堆中的一开始。**

我们可以使用以下方法找到左右孩子的索引:

```java
private int getLeftChildIndex(int i) {
   return 2 * i;
}

private int getRightChildIndex(int i) {
    return ((2 * i) + 1);
}
```

同样，我们可以通过以下代码找到数组中项目的父项和祖父项的索引:

```java
private int getParentIndex(int i) {
   return i / 2;
}

private int getGrandparentIndex(int i) {
   return i / 4;
}
```

现在，让我们继续完成我们简单的最小-最大堆类:

```java
public class MinMaxHeap<T extends Comparable<T>> {
    private List<T> array;
    private int capacity;
    private int indicator;

    MinMaxHeap(int capacity) {
        array = new ArrayList<>();
        this.capacity = capacity;
        indicator = 1;
    }

    MinMaxHeap(List<T> array) {
        this.array = array;
        this.capacity = array.size();
        this.indicator = array.size() + 1;
    }
}
```

我们可以用两种方法创建最小-最大堆的实例。首先，我们用一个 [`ArrayList`](/web/20220628062856/https://www.baeldung.com/java-arraylist) 和特定的容量初始化一个数组，其次，我们从现有的数组中创建一个最小-最大堆。

现在，让我们讨论堆上的操作。

### 3.1.创造

让我们首先看看如何从一个现有的数组构建一个最小-最大堆。这里我们使用 Floyd 算法，并做了一些修改，如 [Heapify 算法](/web/20220628062856/https://www.baeldung.com/cs/binary-tree-max-heapify):

```java
public List<T> create() {
    for (int i = Math.floorDiv(array.size(), 2); i >= 1; i--) {
        pushDown(array, i);
    }
    return array;
}
```

让我们仔细看看下面的代码中的`pushDown`,看看上面的代码到底发生了什么:

```java
private void pushDown(List<T> array, int i) {
    if (isEvenLevel(i)) {
        pushDownMin(array, i);
    } else {
        pushDownMax(array, i);
    }
}
```

正如我们所见，对于所有偶数级，我们用`pushDownMin.` 检查数组项。这种算法类似于我们将用于`removeMin`和`removeMax`的 heapify-down:

```java
private void pushDownMin(List<T> h, int i) {
    while (getLeftChildIndex(i) < indicator) {
       int indexOfSmallest = getIndexOfSmallestChildOrGrandChild(h, i);
          //...
          i = indexOfSmallest;
    }
 }
```

首先，我们找到'`i'`元素的最小子元素或孙元素的索引。那么我们按照以下条件进行。

如果最小的子元素或孙元素不小于当前元素，则我们中断。换句话说，元素的当前排列类似于最小堆:

```java
if (h.get(indexOfSmallest - 1).compareTo(h.get(i - 1)) < 0) {
    //...
} else {
    break;
}
```

**如果最小的子元素或孙元素小于当前元素，我们将其与其父元素或祖父元素交换:**

```java
if (getParentIndex(getParentIndex(indexOfSmallest)) == i) {
       if (h.get(indexOfSmallest - 1).compareTo(h.get(i - 1)) < 0) {
          swap(indexOfSmallest - 1, i - 1, h);
          if (h.get(indexOfSmallest - 1)
            .compareTo(h.get(getParentIndex(indexOfSmallest) - 1)) > 0) {
             swap(indexOfSmallest - 1, getParentIndex(indexOfSmallest) - 1, h);
           }
        }
  } else if (h.get(indexOfSmallest - 1).compareTo(h.get(i - 1)) < 0) {
      swap(indexOfSmallest - 1, i - 1, h);
 }
```

我们将继续上述操作，直到找到元素“I”的子元素。

现在，让我们看看`getIndexOfSmallestChildOrGrandChild `是如何工作的。这很简单！首先，我们假设左边的孩子具有最小值，然后将其与其他孩子进行比较:

```java
private int getIndexOfSmallestChildOrGrandChild(List<T> h, int i) {
    int minIndex = getLeftChildIndex(i);
    T minValue = h.get(minIndex - 1);
    // rest of the implementation
}
```

**在每一步中，如果指数大于指标，最后找到的最小值就是答案。**

例如，让我们比较一下`min-value`和正确的孩子:

```java
if (getRightChildIndex(i) < indicator) {
    if (h.get(getRightChildIndex(i) - 1).compareTo(minValue) < 0) {
        minValue = h.get(getRightChildIndex(i));
        minIndex = getRightChildIndex(i);
    }
} else {
     return minIndex;
}
```

现在，让我们创建一个测试来验证从一个无序数组创建一个最小-最大堆是否可以正常工作:

```java
@Test
public void givenUnOrderedArray_WhenCreateMinMaxHeap_ThenIsEqualWithMinMaxHeapOrdered() {
    List<Integer> list = Arrays.asList(34, 12, 28, 9, 30, 19, 1, 40);
    MinMaxHeap<Integer> minMaxHeap = new MinMaxHeap<>(list);
    minMaxHeap.create();
    Assert.assertEquals(List.of(1, 40, 34, 9, 30, 19, 28, 12), list);
}
```

**`pushDownMax`的算法与`pushDownMin`的算法相同，但是所有的比较都是相反的。**

### 3.2.插入

让我们看看如何向最小-最大堆添加元素:

```java
public void insert(T item) {
    if (isEmpty()) {
        array.add(item);
        indicator++;
    } else if (!isFull()) {
        array.add(item);
        pushUp(array, indicator);
        indicator++;
    } else {
        throw new RuntimeException("invalid operation !!!");
    }
 }
```

首先，我们检查堆是否为空。如果堆是空的，我们添加新元素并增加指示器。否则，添加的新元素可能会改变 min-max 堆的顺序，所以我们需要用`pushUp`来调整堆:

```java
private void pushUp(List<T>h,int i) {
    if (i != 1) {
        if (isEvenLevel(i)) {
            if (h.get(i - 1).compareTo(h.get(getParentIndex(i) - 1)) < 0) {
                pushUpMin(h, i);
            } else {
                swap(i - 1, getParentIndex(i) - 1, h);
                i = getParentIndex(i);
                pushUpMax(h, i);
            }
        } else if (h.get(i - 1).compareTo(h.get(getParentIndex(i) - 1)) > 0) {
            pushUpMax(h, i);
        } else {
            swap(i - 1, getParentIndex(i) - 1, h);
            i = getParentIndex(i);
            pushUpMin(h, i);
        }
    }
}
```

正如我们在上面看到的，新元素比较它的父元素，然后:

*   如果发现它小于(大于)父元素，那么它肯定小于(大于)堆根路径上最大(最小)层上的所有其他元素
*   从新元素到根的路径(只考虑最小/最大级别)应该像插入前一样按降序(升序)排列。所以，我们需要在这个序列中插入一个新元素

现在，让我们看看下面的`pushUpMin`:

```java
private void pushUpMin(List<T> h , int i) {
    while(hasGrandparent(i) && h.get(i - 1)
      .compareTo(h.get(getGrandparentIndex(i) - 1)) < 0) {
        swap(i - 1, getGrandparentIndex(i) - 1, h);
        i = getGrandparentIndex(i);
    }
}
```

**从技术上来说，当父元素更强大时，用它的父元素替换新元素更简单。另外，`pushUpMax`与`pushUpMin`相同，但是所有的比较都是相反的。**

现在，让我们创建一个测试来验证在最小-最大堆中插入一个新元素是否正常:

```java
@Test
public void givenNewElement_WhenInserted_ThenIsEqualWithMinMaxHeapOrdered() {
    MinMaxHeap<Integer> minMaxHeap = new MinMaxHeap(8);
    minMaxHeap.insert(34);
    minMaxHeap.insert(12);
    minMaxHeap.insert(28);
    minMaxHeap.insert(9);
    minMaxHeap.insert(30);
    minMaxHeap.insert(19);
    minMaxHeap.insert(1);
    minMaxHeap.insert(40);
    Assert.assertEquals(List.of(1, 40, 28, 12, 30, 19, 9, 34),
      minMaxHeap.getMinMaxHeap());
}
```

### 3.3.查找最小值

最小-最大堆中的主元素总是位于根，所以我们可以在时间复杂度 O(1)中找到它:

```java
public T min() {
    if (!isEmpty()) {
        return array.get(0);
    }
    return null;
}
```

### 3.4.找到麦克斯

最小-最大堆中的最大元素总是位于第一个奇数层，因此我们可以通过简单的比较在时间复杂度 O(1)中找到它:

```java
public T max() {
    if (!isEmpty()) {
        if (indicator == 2) {
            return array.get(0);
        }
        if (indicator == 3) {
            return array.get(1);
        }
        return array.get(1).compareTo(array.get(2)) < 0 ? array.get(2) : array.get(1);
    }
    return null;
}
```

### 3.5.移除最小值

在这种情况下，我们将找到 min 元素，然后用数组的最后一个元素替换它:

```java
public T removeMin() {
    T min = min();
    if (min != null) {
       if (indicator == 2) {
         array.remove(indicator--);
         return min;
       }
       array.set(0, array.get(--indicator - 1));
       array.remove(indicator - 1);
       pushDown(array, 1);
    }
    return min;
}
```

3.6。移除最大值

移除 max 元素与移除 min 元素是一样的，唯一的变化是我们找到 max 元素的索引，然后调用`pushDown`:

```java
public T removeMax() {
    T max = max();
    if (max != null) {
        int maxIndex;
        if (indicator == 2) {
            maxIndex = 0;
            array.remove(--indicator - 1);
            return max;
        } else if (indicator == 3) {
            maxIndex = 1;
            array.remove(--indicator - 1);
            return max;
        } else {
            maxIndex = array.get(1).compareTo(array.get(2)) < 0 ? 2 : 1;
        }
        array.set(maxIndex, array.get(--indicator - 1));
        array.remove(indicator - 1);
        pushDown(array, maxIndex + 1);
    }
    return max;
}
```

## 4.结论

在本教程中，我们看到了在 Java 中实现最小最大堆，并探索了一些最常见的操作。

首先，我们了解了最小-最大堆到底是什么，包括一些最常见的特性。然后，我们看到了如何在 min-max 堆实现中创建、插入、查找-min、查找-max、删除-min 和删除-max 项。

像往常一样，本文中使用的所有例子都可以在 GitHub 上找到[。](https://web.archive.org/web/20220628062856/https://github.com/eugenp/tutorials/tree/master/data-structures)