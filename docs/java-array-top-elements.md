# 在 Java 数组中查找前 K 个元素

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-array-top-elements>

## 1.概观

在本教程中，我们将用 Java 实现不同的解决方案来解决**在数组中寻找`k`最大元素**的问题。为了描述时间复杂度，我们将使用 [Big-O](/web/20221208143956/https://www.baeldung.com/cs/big-o-notation) 符号。

## 2.强力解决方案

这个问题的强力解决方案是**遍历给定的数组`k`次**。**在每次迭代中，我们将** **找到最大值**。然后，我们将从数组中删除该值，并将其放入输出列表:

```java
public List findTopK(List input, int k) {
    List array = new ArrayList<>(input);
    List topKList = new ArrayList<>();

    for (int i = 0; i < k; i++) {
        int maxIndex = 0;

        for (int j = 1; j < array.size(); j++) {
            if (array.get(j) > array.get(maxIndex)) {
                maxIndex = j;
            }
        }

        topKList.add(array.remove(maxIndex));
    }

    return topKList;
}
```

如果我们假设`n`是给定数组的大小，**这个解的时间复杂度是`O(n * k)`** 。此外，这是最低效的解决方案。

## 3.Java 集合方法

然而，对于这个问题存在更有效的解决方案。在这一节中，我们将使用 Java 集合来解释其中的两个。

### 3.1.`TreeSet`

`TreeSet`有一个 [**红黑树**](/web/20221208143956/https://www.baeldung.com/cs/red-black-trees) **数据结构**作为主干。因此，给这个集合赋值需要花费`O(log n)`。`TreeSet`是一个经过整理的集合。因此，我们可以将`TreeSet`****中的所有值提取出其中的第一个`k`** :**

```java
public List<Integer> findTopK(List<Integer> input, int k) {
    Set<Integer> sortedSet = new TreeSet<>(Comparator.reverseOrder());
    sortedSet.addAll(input);

    return sortedSet.stream().limit(k).collect(Collectors.toList());
}
```

这个解的**时间复杂度是`O(n * log n)`** 。最重要的是，如果`k ≥ log n`，这应该比蛮力方法更有效。

记住`TreeSet`不包含重复项是很重要的。因此，该解决方案仅适用于具有不同值的输入数组。

### 3.2.`PriorityQueue`

`PriorityQueue` **是 Java 中的一种** **堆** **数据结构**。在它的帮助下，**我们可以实现一个`O(n * log k)`解决方案**。此外，这将是一个比前一个更快的解决方案。由于上述问题，`k`总是小于数组的大小。所以，这意味着`O(n * log k) ≤ O(n * log n).`

算法**通过给定的数组**迭代一次。在每次迭代中，我们将向堆中添加一个新元素。此外，我们将保持堆的大小小于或等于`k`。因此，我们必须从堆中移除多余的元素，并添加新的元素。因此，在遍历数组之后，堆将包含`k`个最大值:

```java
public List<Integer> findTopK(List<Integer> input, int k) {
    PriorityQueue<Integer> maxHeap = new PriorityQueue<>();

    input.forEach(number -> {
        maxHeap.add(number);

        if (maxHeap.size() > k) {
            maxHeap.poll();
        }
    });

    List<Integer> topKList = new ArrayList<>(maxHeap);
    Collections.reverse(topKList);

    return topKList;
}
```

## 4.选择算法

有许多方法可以解决给定的问题。尽管这超出了本教程的范围，使用[选择算法](https://web.archive.org/web/20221208143956/https://en.wikipedia.org/wiki/Selection_algorithm)方法 **的**将是最好的**，因为它产生了线性时间复杂度。**

## 5.结论

在本教程中，我们已经描述了寻找数组中最大元素的几种解决方案。

像往常一样，GitHub 上的示例代码[可用。](https://web.archive.org/web/20221208143956/https://github.com/eugenp/tutorials/tree/master/algorithms-modules/algorithms-miscellaneous-6)**