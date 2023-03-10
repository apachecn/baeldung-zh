# Java 中使用堆的整数流的中值

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-stream-integers-median-using-heap>

## 1.概观

在本教程中，我们将学习如何计算整数流的中值。

我们将通过举例说明问题，然后分析问题，最后用 Java 实现几个解决方案。

## 2.问题陈述

**[中位数](https://web.archive.org/web/20220627165942/http://www.icoachmath.com/math_dictionary/median.html)是有序数据集的中间值。对于一组整数，小于中位数的元素和大于中位数的元素一样多。**

在有序的一组中:

*   奇数个整数，中间的元素是中位数——在有序集合`{ 5, 7, 10 }`中，中位数是`7`
*   偶数个整数，没有中间元素；中值计算为两个中间元素的平均值——在有序集合`{5, 7, 8, 10}`中，中值为`(7 + 8) / 2 = 7.5`

现在，让我们假设我们从数据流中读取整数，而不是有限集。我们可以将一个整数流的**中值定义为** **目前读取的整数集合的中值**。

让我们将问题陈述形式化。给定整数流的输入，我们必须设计一个类，为我们读取的每个整数执行以下两个任务:

1.  将该整数添加到整数集合中
2.  找到目前为止读取的整数的中间值

例如:

```java
add 5         // sorted-set = { 5 }, size = 1
get median -> 5

add 7         // sorted-set = { 5, 7 }, size = 2 
get median -> (5 + 7) / 2 = 6

add 10        // sorted-set = { 5, 7, 10 }, size = 3 
get median -> 7

add 8         // sorted-set = { 5, 7, 8, 10 }, size = 4 
get median -> (7 + 8) / 2 = 7.5
.. 
```

尽管流是非有限的，但我们可以假设我们可以一次在内存中保存流的所有元素。

我们可以将我们的任务表示为代码中的以下操作:

```java
void add(int num);

double getMedian(); 
```

## 3.天真的方法

### 3.1.已排序`List`

让我们从一个简单的想法开始——我们可以通过索引访问`list`的中间元素或中间两个元素来计算整数排序后的`list`的中值。`getMedian`操作的时间复杂度为`O(1)`。

当添加一个新的整数时，我们必须确定它在`list`中的正确位置，以使`list`保持有序。该操作可以在`O(n)`时间内进行，其中`n`是`list`的大小。因此，向`list`添加新元素并计算新的中值的总成本是`O(n)`。

### 3.2.改进简单的方法

`add`操作在线性时间内运行，这不是最佳的。让我们在本节中尝试解决这个问题。

我们可以将`list`分成两个排序后的`lists`–**，较小的一半整数按降序排序，较大的一半整数按升序排序**。我们可以在适当的一半中添加一个新的整数，使得`lists`的大小最多相差 1:

```java
if element is smaller than min. element of larger half:
    insert into smaller half at appropriate index
    if smaller half is much bigger than larger half:
        remove max. element of smaller half and insert at the beginning of larger half (rebalance)
else
    insert into larger half at appropriate index:
    if larger half is much bigger than smaller half:
        remove min. element of larger half and insert at the beginning of smaller half (rebalance) 
```

[![](img/f4be7dc25636faa385a0df1e495e3e4d.png)](/web/20220627165942/https://www.baeldung.com/wp-content/uploads/2019/12/Halves-Median-scaled-1.png)

现在，我们可以计算中位数:

```java
if lists contain equal number of elements:
    median = (max. element of smaller half + min. element of larger half) / 2
else if smaller half contains more elements:
    median = max. element of smaller half
else if larger half contains more elements:
    median = min. element of larger half
```

虽然我们只是通过某个常数因子改进了`add`操作的时间复杂度，但是我们已经取得了进展。

让我们分析一下我们在两个排序的`lists`中访问的元素。在(sorted) `add `操作中，当我们移动元素时，我们可能会访问每个元素。更重要的是，在`add `重新平衡操作和`getMedian `操作期间，我们分别访问较大和较小一半的最小值和最大值(极值)。

我们可以看到**极值是它们各自列表**的第一个元素。因此，我们**必须针对每半个**优化访问索引`0`处的元素，以提高`add`操作的总运行时间。

## 4.基于`Heap`的方法

让我们通过应用我们从幼稚的方法中学到的东西来完善我们对这个问题的理解:

1.  我们必须在`O(1)`时间内获得数据集的最小/最大元素
2.  只要我们能有效地获得最小/最大元素，元素就不必按排序顺序保存
3.  我们需要找到一种向数据集添加元素的方法，这种方法花费的时间少于`O(n)`时间

接下来，我们将看看堆数据结构，它帮助我们高效地实现我们的目标。

### 4.1.堆数据结构

[`Heap`](/web/20220627165942/https://www.baeldung.com/java-heap-sort#heap-data-structure) 是一种**数据结构，通常用数组实现，但也可以认为是二叉树**。

[![](img/1434f5bae945c2ed4c3114ea1e2f672a.png)](/web/20220627165942/https://www.baeldung.com/wp-content/uploads/2019/12/Min-Max-Heap-scaled-1.png)

堆受堆属性的约束:

#### 4.1.1。最大`–`堆属性

(子)节点的值不能大于父节点的值。因此，在`max-heap`中，根节点总是具有最大值。

#### 4.1.2。最小`–`堆属性

(父)节点的值不能大于其子节点的值。因此，在`min-heap`中，根节点总是具有最小值。

在 Java 中，`[PriorityQueue](https://web.archive.org/web/20220627165942/https://algs4.cs.princeton.edu/24pq/)`类代表一个堆。让我们前进到使用堆的第一个解决方案。

### 4.2.第一种解决方案

让我们用两个堆来代替我们简单方法中的列表:

*   包含一半以上元素的最小堆，最小元素位于根
*   包含较小一半元素的最大堆，最大元素位于根

现在，我们可以通过将传入的整数与最小堆的根进行比较，将它添加到相关的一半。接下来，如果在插入之后，一个堆的大小与另一个堆的大小相差超过 1，我们可以重新平衡这些堆，从而将大小差保持在最多 1:

```java
if size(minHeap) > size(maxHeap) + 1:
    remove root element of minHeap, insert into maxHeap
if size(maxHeap) > size(minHeap) + 1:
    remove root element of maxHeap, insert into minHeap
```

使用这种方法，如果两个堆的大小相等，我们可以计算两个堆的根元素的平均值。否则，具有更多元素的堆的**根元素是中间值**。

我们将使用`PriorityQueue`类来表示堆。`PriorityQueue`的默认堆属性是最小堆。我们可以通过使用与自然顺序相反的`Comparator.reverserOrder`来创建 max-heap:

```java
class MedianOfIntegerStream {

    private Queue<Integer> minHeap, maxHeap;

    MedianOfIntegerStream() {
        minHeap = new PriorityQueue<>();
        maxHeap = new PriorityQueue<>(Comparator.reverseOrder());
    }

    void add(int num) {
        if (!minHeap.isEmpty() && num < minHeap.peek()) {
            maxHeap.offer(num);
            if (maxHeap.size() > minHeap.size() + 1) {
                minHeap.offer(maxHeap.poll());
            }
        } else {
            minHeap.offer(num);
            if (minHeap.size() > maxHeap.size() + 1) {
                maxHeap.offer(minHeap.poll());
            }
        }
    }

    double getMedian() {
        int median;
        if (minHeap.size() < maxHeap.size()) {
            median = maxHeap.peek();
        } else if (minHeap.size() > maxHeap.size()) {
            median = minHeap.peek();
        } else {
            median = (minHeap.peek() + maxHeap.peek()) / 2; 
        }
        return median;
    }
}
```

在我们分析代码的运行时间之前，让我们看看我们使用的堆操作的时间复杂性:

```java
find-min/find-max        O(1)    

delete-min/delete-max    O(log n)

insert                   O(log n) 
```

因此，`getMedian `操作可以在`O(1)`时间内执行，因为它只需要 `find-min` 和`find-max`功能。`add`操作的时间复杂度是`O(log n)`–三个`insert` / `delete `调用，每个调用需要`O(log n) `时间。

### 4.3.堆大小不变解决方案

在我们之前的方法中，我们将每个新元素与堆的根元素进行了比较。让我们探索另一种使用 heap 的方法，其中我们可以利用 heap 属性在适当的一半中添加新元素。

正如我们在前面的解决方案中所做的那样，我们从两个堆开始——最小堆和最大堆。接下来，我们引入一个条件:**max-heap 的大小必须始终为`(n / 2)`，而 min-heap 的大小可以是`(n / 2)`也可以是`(n / 2) + 1`，这取决于两个堆中的元素总数**。换句话说，当元素总数为奇数时，我们可以只允许最小堆有一个额外的元素。

在堆大小不变的情况下，如果两个堆的大小都是`(n / 2)`，我们可以计算两个堆的根元素的平均值。否则，最小堆的根元素**就是中间的**。

当我们添加一个新的整数时，我们有两种情况:

```java
1\. Total no. of existing elements is even
   size(min-heap) == size(max-heap) == (n / 2)

2\. Total no. of existing elements is odd
   size(max-heap) == (n / 2)
   size(min-heap) == (n / 2) + 1 
```

我们可以通过向其中一个堆添加新元素并每次重新平衡来保持不变量:

[![](img/d4d4d36e76eb336c47c5dd161cbbac10.png)](/web/20220627165942/https://www.baeldung.com/wp-content/uploads/2019/12/Heap-Solution.png)

重新平衡通过将最大的元素从最大堆移动到最小堆，或者将最小的元素从最小堆移动到最大堆来实现。这样，尽管在把新的整数添加到堆中之前，我们不会对其进行比较，但是随后的重新平衡确保了我们尊重更小和更大的一半的基本不变量。

让我们用 Java 实现我们的解决方案，使用`PriorityQueues`:

```java
class MedianOfIntegerStream {

    private Queue<Integer> minHeap, maxHeap;

    MedianOfIntegerStream() {
        minHeap = new PriorityQueue<>();
        maxHeap = new PriorityQueue<>(Comparator.reverseOrder());
    }

    void add(int num) {
        if (minHeap.size() == maxHeap.size()) {
            maxHeap.offer(num);
            minHeap.offer(maxHeap.poll());
        } else {
            minHeap.offer(num);
            maxHeap.offer(minHeap.poll());
        }
    }

    double getMedian() {
        int median;
        if (minHeap.size() > maxHeap.size()) {
            median = minHeap.peek();
        } else {
            median = (minHeap.peek() + maxHeap.peek()) / 2;
        }
        return median;
    }
}
```

**我们操作的时间复杂度保持不变** : `getMedian`花费`O(1)`时间，而`add`以完全相同的操作数在`O(log n)`时间内运行。

这两种基于堆的解决方案提供了相似的空间和时间复杂性。虽然第二种解决方案很聪明，实现也更简洁，但这种方法并不直观。另一方面，第一种解决方案很自然地遵循我们的直觉，并且更容易推理其`add`操作的正确性。

## 5.**结论**

在本教程中，我们学习了如何计算整数流的中值。我们评估了几种方法，并使用`PriorityQueue`在 Java 中实现了几种不同的解决方案。

像往常一样，所有例子的源代码都可以在 GitHub 的[上找到。](https://web.archive.org/web/20220627165942/https://github.com/eugenp/tutorials/tree/master/algorithms-modules/algorithms-miscellaneous-5)