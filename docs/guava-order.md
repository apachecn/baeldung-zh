# 番石榴订购食谱

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/guava-order>

## 1。简介

这本烹饪书展示了如何使用番石榴排序和比较器。它延续了我在[开始的关于番石榴收集的前一篇文章](/web/20220628125433/https://www.baeldung.com/guava-collections "Guava Collections Cookbook")中的食谱和范例焦点格式。

## 2。烹饪书

**处理集合中的空值**

**空值优先**

```java
List<Integer> toSort = Arrays.asList(3, 5, 4, null, 1, 2);
Collections.sort(toSort, Ordering.natural().nullsFirst());
assertThat(toSort.get(0), nullValue());
```

**空值最后一个**

```java
List<Integer> toSort = Arrays.asList(3, 5, 4, null, 1, 2);
Collections.sort(toSort, Ordering.natural().nullsLast());
assertThat(toSort.get(toSort.size() - 1), nullValue());
```

**自然排序**

```java
List<Integer> toSort = Arrays.asList(3, 5, 4, 1, 2);
Collections.sort(toSort, Ordering.natural());

assertTrue(Ordering.natural().isOrdered(toSort));
```

**链接 2 个订单**

```java
List<Integer> toSort = Arrays.asList(3, 5, 4, 1, 2);
Collections.sort(toSort, Ordering.natural().reverse());
```

**反向排序**

```java
List<Integer> toSort = Arrays.asList(3, 5, 4, null, 1, 2);
Collections.sort(toSort, Ordering.natural().nullsLast().reverse());
assertThat(toSort.get(0), nullValue());
```

**自定义顺序——字符串按长度**

```java
private class OrderingByLenght extends Ordering<String> {
    @Override
    public int compare(String s1, String s2) {
        return Ints.compare(s1.length(), s2.length());
    }
}
List<String> toSort = Arrays.asList("zz", "aa", "b", "ccc");
Ordering<String> byLength = new OrderingByLenght();
Collections.sort(toSort, byLength);

Ordering<String> expectedOrder = Ordering.explicit(Lists.newArrayList("b", "zz", "aa", "ccc"));
assertTrue(expectedOrder.isOrdered(toSort))
```

**检查显式顺序**

```java
List<String> toSort = Arrays.asList("zz", "aa", "b", "ccc");
Ordering<String> byLength = new OrderingByLenght();
Collections.sort(toSort, byLength);

Ordering<String> expectedOrder = Ordering.explicit(Lists.newArrayList("b", "zz", "aa", "ccc"));
assertTrue(expectedOrder.isOrdered(toSort));
```

**检查字符串排序**

```java
List<Integer> toSort = Arrays.asList(3, 5, 4, 2, 1, 2);
Collections.sort(toSort, Ordering.natural());

assertFalse(Ordering.natural().isStrictlyOrdered(toSort));
```

**二次订购**

```java
List<String> toSort = Arrays.asList("zz", "aa", "b", "ccc");
Ordering<String> byLength = new OrderingByLenght();
Collections.sort(toSort, byLength.compound(Ordering.natural()));

Ordering<String> expectedOrder = Ordering.explicit(Lists.newArrayList("b", "aa", "zz", "ccc"));
assertTrue(expectedOrder.isOrdered(toSort));
```

**复杂定制订购示例——带链接** 

```java
List<String> toSort = Arrays.asList("zz", "aa", null, "b", "ccc");
Collections.sort(toSort, 
    new OrderingByLenght().reverse().compound(Ordering.natural()).nullsLast());
System.out.println(toSort);
```

**使用`toString`表示法** 排序

```java
List<Integer> toSort = Arrays.asList(1, 2, 11);
Collections.sort(toSort, Ordering.usingToString());

Ordering<Integer> expectedOrder = Ordering.explicit(Lists.newArrayList(1, 11, 2));
assertTrue(expectedOrder.isOrdered(toSort));
```

**排序，然后找到【二分搜索法】**

```java
List<Integer> toSort = Arrays.asList(1, 2, 11);
Collections.sort(toSort, Ordering.usingToString());
int found = Ordering.usingToString().binarySearch(toSort, 2);
System.out.println(found);
```

**不用排序就找到最小值/最大值(更快)**

```java
List<Integer> toSort = Arrays.asList(2, 1, 11, 100, 8, 14);
int found = Ordering.usingToString().min(toSort);
assertThat(found, equalTo(1));
```

**根据排序创建列表的排序副本**

```java
List<String> toSort = Arrays.asList("aa", "b", "ccc");
List<String> sortedCopy = new OrderingByLenght().sortedCopy(toSort);

Ordering<String> expectedOrder = Ordering.explicit(Lists.newArrayList("b", "aa", "ccc"));
assertFalse(expectedOrder.isOrdered(toSort));
assertTrue(expectedOrder.isOrdered(sortedCopy));
```

**创建排序后的部分副本——最少元素**

```java
List<Integer> toSort = Arrays.asList(2, 1, 11, 100, 8, 14);
List<Integer> leastOf = Ordering.natural().leastOf(toSort, 3);
List<Integer> expected = Lists.newArrayList(1, 2, 8);
assertThat(expected, equalTo(leastOf));
```

**通过中介功能订购**

```java
List<Integer> toSort = Arrays.asList(2, 1, 11, 100, 8, 14);
Ordering<Object> ordering = Ordering.natural().onResultOf(Functions.toStringFunction());
List<Integer> sortedCopy = ordering.sortedCopy(toSort);

List<Integer> expected = Lists.newArrayList(1, 100, 11, 14, 2, 8);
assertThat(expected, equalTo(sortedCopy));
```

–**注意**:排序逻辑将首先通过函数运行数字，将它们转换成字符串，然后按照字符串的自然顺序进行排序

## 3。更多番石榴食谱

番石榴是一个全面且非常有用的库——以下是烹饪书形式的更多 API:

*   #### [Guava functional recipe](/web/20220628125433/https://www.baeldung.com/guava-functions-predicates "Guava Functional Cookbook")

*   #### [Guava series recipes](/web/20220628125433/https://www.baeldung.com/guava-collections "Guava Collections Cookbook")

享受吧。

## 4。结论

这种实验性的格式——食谱——有一个明确的重点——简单和快速，所以大多数食谱除了代码示例本身之外没有其他解释。

正如我之前提到的，这是一份活的文档，欢迎在评论中加入新的例子和用例，我会继续添加我自己的例子和用例。

所有这些例子和代码片段的实现**可以在 GitHub** 上找到[——这是一个基于 Maven 的项目，所以它应该很容易导入和运行。](https://web.archive.org/web/20220628125433/https://github.com/eugenp/tutorials/tree/master/guava-modules/guava-collections)