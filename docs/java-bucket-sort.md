# Java 中的桶排序

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-bucket-sort>

## 1.介绍

在本文中，我们将深入研究 **[桶排序](https://web.archive.org/web/20221205122937/https://en.wikipedia.org/wiki/Bucket_sort)算法。**在进行 Java 实现和单元测试我们的解决方案之前，我们将从简单的**理论开始。最后，我们将**看看桶排序的时间复杂度**。**

## 2.桶排序理论

桶排序，有时也称为箱排序，是一种特定的排序算法。排序的工作方式是将我们想要排序的元素分布到几个单独排序的桶中。通过这样做，我们可以减少元素之间的比较次数，并有助于减少排序时间。

让我们快速看一下执行桶排序所需的**步骤:**

1.  建立一个最初空桶的数组
2.  将我们的元素分配到适当的桶中
3.  对每个存储桶进行排序
4.  将排序后的桶连接在一起以重新创建完整的列表

## 3.Java 实现

虽然这个算法不是特定于语言的，但是我们将在 Java 中实现排序。让我们一步一步地浏览上面的列表，并编写代码对一个整数列表进行排序。

### 3.1.铲斗设置

首先，我们**需要确定一个散列算法**来决定我们的哪些元素被放入哪个桶:

```java
private int hash(int i, int max, int numberOfBuckets) {
    return (int) ((double) i / max * (numberOfBuckets - 1));
}
```

定义了散列方法后，我们现在可以用输入列表大小的平方根来指定容器的数量:

```java
final int numberOfBuckets = (int) Math.sqrt(initialList.size());
List<List<Integer>> buckets = new ArrayList<>(numberOfBuckets);
for(int i = 0; i < numberOfBuckets; i++) {
    buckets.add(new ArrayList<>());
}
```

最后，我们需要一个简单的方法来确定输入列表中的最大整数:

```java
private int findMax(List<Integer> input) {
    int m = Integer.MIN_VALUE;
    for (int i : input) {
        m = Math.max(i, m);
    }
    return m;
}
```

### 3.2.分发元素

现在我们已经定义了我们的桶，我们可以使用`hash `方法将输入列表中的每个元素分配到相应的桶中:

```java
int max = findMax(initialList);

for (int i : initialList) {
    buckets.get(hash(i, max, numberOfBuckets)).add(i);
} 
```

### 3.3.对单个存储桶进行排序

定义好桶并装满整数后，**让我们用一个`Comparator` 对它们进行排序**:

```java
Comparator<Integer> comparator = Comparator.naturalOrder();

for(List<Integer> bucket  : buckets){
    bucket.sort(comparator);
}
```

### 3.4.串联我们的桶

最后，我们需要齐心协力重新创建单一列表。因为我们的存储桶已经排序，所以我们只需要遍历每个存储桶一次，并将元素添加到主列表中:

```java
List<Integer> sortedArray = new LinkedList<>();

for(List<Integer> bucket : buckets) {
    sortedArray.addAll(bucket);
} 

return sortedArray;
```

## 4.测试我们的代码

实现完成后，让我们编写一个快速的单元测试来确保它按预期工作:

```java
BucketSorter sorter = new IntegerBucketSorter();

List<Integer> unsorted = Arrays.asList(80,50,60,30,20,10,70,0,40,500,600,602,200,15);
List<Integer> expected = Arrays.asList(0,10,15,20,30,40,50,60,70,80,200,500,600,602);

List<Integer> sorted = sorter.sort(unsorted);

assertEquals(expected, sorted);
```

## 5.时间复杂度

接下来，让我们快速看一下执行桶排序的时间复杂度。

### 5.1.最坏的情况

在最坏的情况下，我们会在同一个桶中以相反的顺序找到所有的元素。当这种情况发生时，我们将我们的桶排序减少到一个简单的排序，其中每个元素与每个其他元素进行比较，**产生 O(n )** 的时间复杂度。

### 5.2.平均案例情景

在我们的平均情况下，我们发现**元素相对均匀地分布在我们的输入桶中。**因为我们的每一步只需要迭代一次我们的输入桶，我们发现我们的桶排序**在 O(n)时间**内完成。

## 6.结论

在本文中，我们看到了如何用 Java 实现桶排序。我们还研究了桶排序算法的时间复杂度。

和往常一样，本文中显示的代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20221205122937/https://github.com/eugenp/tutorials/tree/master/algorithms-modules/algorithms-sorting)