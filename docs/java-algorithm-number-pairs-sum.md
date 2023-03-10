# 在 Java 中找出数组中所有加起来等于给定总和的数字对

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-algorithm-number-pairs-sum>

## 1。概述

在这个快速教程中，我们将展示如何实现一个算法来寻找数组中所有和等于给定数字的数字对。我们将关注解决问题的两种方法。

在第一种方法中，我们将找到所有这样的对，而不考虑唯一性。在第二个例子中，我们将只找到唯一的数字组合，去掉多余的数字对。

对于每种方法，我们将给出两种实现——一种使用`for `循环的传统实现，另一种使用 Java 8 Stream API。

## 2。返回所有匹配对

我们将遍历一个整数数组，使用一种强力的、嵌套循环的方法找到所有加起来等于给定数字(`sum`)的对(`i`和`j`)。**该算法的运行时间复杂度为`O(n²)`。**

在我们的演示中，我们将使用下面的`input`数组来寻找总和等于`6`的所有数字对:

```java
int[] input = { 2, 4, 3, 3 }; 
```

在这种方法中，我们的算法应该返回:

```java
{2,4}, {4,2}, {3,3}, {3,3}
```

在每一个算法中，当我们找到一对加起来等于目标数的目标数时，我们将使用一个实用方法`addPairs(i, j)`来收集这对数。

**我们可能想到的实现解决方案的第一种方法是使用传统的`for`循环:**

```java
for (int i = 0; i < input.length; i++) {
    for (int j = 0; j < input.length; j++) {
        if (j != i && (input[i] + input[j]) == sum) {
            addPairs(input[i], sum-input[i]));
        }
    }
}
```

这可能有点初级，所以**让我们也使用 Java 8 Stream API** 编写一个实现。

这里，我们使用方法`IntStream.range `来生成一个连续的数字流。然后，我们根据我们的条件过滤它们: `number 1 + number 2 = sum`:

```java
IntStream.range(0,  input.length)
    .forEach(i -> IntStream.range(0,  input.length)
        .filter(j -> i != j && input[i] + input[j] == sum)
        .forEach(j -> addPairs(input[i], input[j]))
); 
```

## 3。返回所有唯一匹配对

对于这个例子，我们必须**开发一个更智能的算法，只返回唯一的数字组合，忽略多余的数字对**。

为了实现这一点，我们将把每个元素添加到一个 hash map 中(不进行排序)，首先检查该元素对是否已经显示。如果没有，我们将检索并标记它，如图所示(将*值*字段设置为`null`)。

因此，使用与之前相同的`input`数组，以及`6`的目标和，我们的算法应该只返回不同的数字组合:

```java
{2,4}, {3,3}
```

**如果我们使用传统的`for `循环，我们将有:**

```java
Map<Integer, Integer> pairs = new HashMap();
for (int i : input) {
    if (pairs.containsKey(i)) {
        if (pairs.get(i) != null) {            
            addPairs(i, sum-i);
        }                
        pairs.put(sum - i, null);
    } else if (!pairs.containsValue(i)) {        
        pairs.put(sum-i, i);
    }
}
```

注意，这个实现改进了之前的复杂性，因为**我们只使用了一个`for `循环，所以我们将有`O(n)`** 。

现在让我们使用 Java 8 和 Stream API 来解决这个问题:

```java
Map<Integer, Integer> pairs = new HashMap();
IntStream.range(0, input.length).forEach(i -> {
    if (pairs.containsKey(input[i])) {
        if (pairs.get(input[i]) != null) {
            addPairs(input[i], sum - input[i]);
        }
        pairs.put(sum - input[i], null);
    } else if (!pairs.containsValue(input[i])) {
        pairs.put(sum - input[i], input[i]);
    }
});
```

## 4。结论

在本文中，我们解释了几种不同的方法来寻找 Java 中所有对给定数字求和的对。我们看到了两个不同的解决方案，每个都使用了两个 Java 核心方法。

像往常一样，本文中显示的所有代码示例都可以在 GitHub 上找到[——这是一个 Maven 项目，因此应该很容易编译和运行它。](https://web.archive.org/web/20221129002330/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-numbers)