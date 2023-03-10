# Java 中的排序

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-sorting>

## 1。概述

本文将举例说明如何在 Java 7 和 Java 8 中对`Array`、`List`、`Set`和`Map`应用排序。

## 2。用`Array`排序

我们先用`[Arrays.sort()](https://web.archive.org/web/20220805115431/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Arrays.html#sort(byte%5B%5D))` 方法对整数数组进行排序。

我们将在一个`@Before` jUnit 方法中定义下面的`int`数组:

```java
@Before
public void initVariables () {
    toSort = new int[] 
      { 5, 1, 89, 255, 7, 88, 200, 123, 66 }; 
    sortedInts = new int[] 
      {1, 5, 7, 66, 88, 89, 123, 200, 255};
    sortedRangeInts = new int[] 
      {5, 1, 89, 7, 88, 200, 255, 123, 66};
    ...
}
```

### 2.1。排序完整数组

现在让我们使用简单的`Array.sort()` API:

```java
@Test
public void givenIntArray_whenUsingSort_thenSortedArray() {
    Arrays.sort(toSort);

    assertTrue(Arrays.equals(toSort, sortedInts));
}
```

未排序的数组现在已完全排序:

```java
[1, 5, 7, 66, 88, 89, 123, 200, 255]
```

[官方 JavaDoc](https://web.archive.org/web/20220805115431/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Arrays.html#sort(int%5B%5D)) 中提到， `Arrays.sort`对**原语**使用双支点快速排序。它提供 O(n log(n))性能，并且通常比传统的(单支点)快速排序实现更快。然而，它对对象的`Array`使用了 [mergesort 算法的稳定、自适应、迭代实现。](https://web.archive.org/web/20220805115431/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Arrays.html#sort(java.lang.Object%5B%5D))

### 2.2。对数组的一部分进行排序

`Arrays.sort`还有一个`sort`API——我们将在这里讨论:

```java
Arrays.sort(int[] a, int fromIndex, int toIndex)
```

这只会对数组中两个索引之间的一部分进行排序。

让我们看一个简单的例子:

```java
@Test
public void givenIntArray_whenUsingRangeSort_thenRangeSortedArray() {
    Arrays.sort(toSort, 3, 7);

    assertTrue(Arrays.equals(toSort, sortedRangeInts));
}
```

将仅对以下子数组元素进行排序(`toIndex`除外):

```java
[255, 7, 88, 200]
```

包括主数组在内的排序后的子数组将是:

```java
[5, 1, 89, 7, 88, 200, 255, 123, 66]
```

### 2.3。Java 8`Arrays.sort`vs`Arrays.parallelSort`

Java 8 附带了一个新的 API—`parallelSort`,其签名与`Arrays.sort()` API 相似:

```java
@Test 
public void givenIntArray_whenUsingParallelSort_thenArraySorted() {
    Arrays.parallelSort(toSort);

    assertTrue(Arrays.equals(toSort, sortedInts));
}
```

在`parallelSort(),` 的幕后，它将数组分成不同的子数组(根据`parallelSort`算法中的粒度)。每个子数组在不同的线程中用`Arrays.sort()`排序，这样`sort`可以并行执行，最后合并为一个排序后的数组。

请注意， [ForJoin 公共池](https://web.archive.org/web/20220805115431/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/ForkJoinPool.html#commonPool())用于执行这些并行任务，然后合并结果。

**`Arrays.parallelSort`的结果将与`Array.sort`相同。当然，这只是利用多线程的问题。**

最后，在`Arrays.parallelSort`中也有类似的 API `Arrays.sort`变体:

```java
Arrays.parallelSort (int [] a, int fromIndex, int toIndex);
```

## 3。`List`排序一

现在让我们使用`[java.utils.Collections](https://web.archive.org/web/20220805115431/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Collections.html)` 中的`[Collections.sort()](https://web.archive.org/web/20220805115431/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Collections.html#sort(java.util.List))` API 对整数的`List`进行排序:

```java
@Test
public void givenList_whenUsingSort_thenSortedList() {
    List<Integer> toSortList = Ints.asList(toSort);
    Collections.sort(toSortList);

    assertTrue(Arrays.equals(toSortList.toArray(), 
    ArrayUtils.toObject(sortedInts)));
}
```

排序前的`List`将包含以下元素:

```java
[5, 1, 89, 255, 7, 88, 200, 123, 66]
```

自然，排序后:

```java
[1, 5, 7, 66, 88, 89, 123, 200, 255]
```

[正如在 Oracle JavaDoc](https://web.archive.org/web/20220805115431/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Collections.html#sort(java.util.List)) 中提到的`Collections.Sort`，它使用了一种改进的合并排序，并提供有保证的`n log(n)`性能。

## 4。`Set`排序一

接下来，我们用`[Collections.sort()](https://web.archive.org/web/20220805115431/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Collections.html#sort(java.util.List))`对一个`LinkedHashSet`进行排序。

我们使用`LinkedHashSet`是因为它保持了插入顺序。

注意，为了在`Collections`–**中使用`sort` API，我们首先将集合包装在一个列表**中:

```java
@Test
public void givenSet_whenUsingSort_thenSortedSet() {
    Set<Integer> integersSet = new LinkedHashSet<>(Ints.asList(toSort));
    Set<Integer> descSortedIntegersSet = new LinkedHashSet<>(
      Arrays.asList(new Integer[] 
        {255, 200, 123, 89, 88, 66, 7, 5, 1}));

    List<Integer> list = new ArrayList<Integer>(integersSet);
    Collections.sort(Comparator.reverseOrder());
    integersSet = new LinkedHashSet<>(list);

    assertTrue(Arrays.equals(
      integersSet.toArray(), descSortedIntegersSet.toArray()));
}
```

`[Comparator.reverseOrder()](https://web.archive.org/web/20220805115431/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Comparator.html#reverseOrder()) `方法颠倒了自然排序的顺序。

## 5。`Map`排序

在这一节中，我们将开始看一下**对映射进行排序——既通过键也通过值。**

让我们首先定义要排序的地图:

```java
@Before
public void initVariables () {
    ....
    HashMap<Integer, String> map = new HashMap<>();
    map.put(55, "John");
    map.put(22, "Apple");
    map.put(66, "Earl");
    map.put(77, "Pearl");
    map.put(12, "George");
    map.put(6, "Rocky");
    ....
}
```

### 5.1。按键排序`Map`

我们现在将从`HashMap`中提取**关键字**和**值**条目，并根据本例中关键字的值对其进行排序:

```java
@Test
public void givenMap_whenSortingByKeys_thenSortedMap() {
    Integer[] sortedKeys = new Integer[] { 6, 12, 22, 55, 66, 77 };

    List<Map.Entry<Integer, String>> entries 
      = new ArrayList<>(map.entrySet());
    Collections.sort(entries, new Comparator<Entry<Integer, String>>() {
        @Override
        public int compare(
          Entry<Integer, String> o1, Entry<Integer, String> o2) {
            return o1.getKey().compareTo(o2.getKey());
        }
    });
    Map<Integer, String> sortedMap = new LinkedHashMap<>();
    for (Map.Entry<Integer, String> entry : entries) {
        sortedMap.put(entry.getKey(), entry.getValue());
    }

    assertTrue(Arrays.equals(sortedMap.keySet().toArray(), sortedKeys));
}
```

注意我们在复制基于键排序的`Entries`时是如何使用`LinkedHashMap` 的(因为`HashSet`不能保证键的顺序)。

排序前的 `Map`:

```java
[Key: 66 , Value: Earl] 
[Key: 22 , Value: Apple] 
[Key: 6 , Value: Rocky] 
[Key: 55 , Value: John] 
[Key: 12 , Value: George] 
[Key: 77 , Value: Pearl]
```

通过键对**排序后的 `Map`:**

```java
[Key: 6 , Value: Rocky] 
[Key: 12 , Value: George] 
[Key: 22 , Value: Apple] 
[Key: 55 , Value: John] 
[Key: 66 , Value: Earl] 
[Key: 77 , Value: Pearl] 
```

### 5.2。按值排序`Map`

这里我们将比较`HashMap`条目的值，以便根据`HashMap`的值进行排序:

```java
@Test
public void givenMap_whenSortingByValues_thenSortedMap() {
    String[] sortedValues = new String[] 
      { "Apple", "Earl", "George", "John", "Pearl", "Rocky" };

    List<Map.Entry<Integer, String>> entries 
      = new ArrayList<>(map.entrySet());
    Collections.sort(entries, new Comparator<Entry<Integer, String>>() {
        @Override
        public int compare(
          Entry<Integer, String> o1, Entry<Integer, String> o2) {
            return o1.getValue().compareTo(o2.getValue());
        }
    });
    Map<Integer, String> sortedMap = new LinkedHashMap<>();
    for (Map.Entry<Integer, String> entry : entries) {
        sortedMap.put(entry.getKey(), entry.getValue());
    }

    assertTrue(Arrays.equals(sortedMap.values().toArray(), sortedValues));
}
```

排序前的`Map`:

```java
[Key: 66 , Value: Earl] 
[Key: 22 , Value: Apple] 
[Key: 6 , Value: Rocky] 
[Key: 55 , Value: John] 
[Key: 12 , Value: George] 
[Key: 77 , Value: Pearl]
```

按值对**排序后的 `Map`:**

```java
[Key: 22 , Value: Apple] 
[Key: 66 , Value: Earl] 
[Key: 12 , Value: George] 
[Key: 55 , Value: John] 
[Key: 77 , Value: Pearl] 
[Key: 6 , Value: Rocky]
```

## 6。分类自定义对象

现在让我们使用一个自定义对象:

```java
public class Employee implements Comparable {
    private String name;
    private int age;
    private double salary;

    public Employee(String name, int age, double salary) {
        ...
    }

    // standard getters, setters and toString
}
```

我们将使用下面的`Employee`数组作为下面几节中的排序示例:

```java
@Before
public void initVariables () {
    ....    
    employees = new Employee[] { 
      new Employee("John", 23, 5000), new Employee("Steve", 26, 6000), 
      new Employee("Frank", 33, 7000), new Employee("Earl", 43, 10000), 
      new Employee("Jessica", 23, 4000), new Employee("Pearl", 33, 6000)};

    employeesSorted = new Employee[] {
      new Employee("Earl", 43, 10000), new Employee("Frank", 33, 70000),
      new Employee("Jessica", 23, 4000), new Employee("John", 23, 5000), 
      new Employee("Pearl", 33, 4000), new Employee("Steve", 26, 6000)};

    employeesSortedByAge = new Employee[] { 
      new Employee("John", 23, 5000), new Employee("Jessica", 23, 4000), 
      new Employee("Steve", 26, 6000), new Employee("Frank", 33, 70000), 
      new Employee("Pearl", 33, 4000), new Employee("Earl", 43, 10000)};
}
```

我们可以对自定义对象的数组或集合进行排序:

1.  以自然顺序(使用`Comparable`界面)或
2.  按`Comparator` `Interface`提供的顺序

### 6.1。★T2`sing Comparable`

[Java 中的自然顺序](https://web.archive.org/web/20220805115431/https://docs.oracle.com/javase/tutorial/collections/interfaces/order.html)是指原语或对象在给定数组或集合中应该有序排序的顺序。

`[java.util.Arrays](https://web.archive.org/web/20220805115431/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Arrays.html)` 和 [`java.util.Collections`](https://web.archive.org/web/20220805115431/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Collections.html) 都有一个`sort()`方法，**强烈建议自然命令要和`equals`的语义一致。**

在本例中，我们将认为具有相同`name`的员工是平等的:

```java
@Test
public void givenEmpArray_SortEmpArray_thenSortedArrayinNaturalOrder() {
    Arrays.sort(employees);

    assertTrue(Arrays.equals(employees, employeesSorted));
}
```

您可以通过实现一个`Comparable` 接口来定义元素的自然顺序，该接口具有用于比较当前对象和作为参数传递的对象的`compareTo()`方法。

为了清楚地理解这一点，让我们看一个实现了`Comparable` 接口的示例`Employee`类:

```java
public class Employee implements Comparable {
    ...

    @Override
    public boolean equals(Object obj) {
        return ((Employee) obj).getName().equals(getName());
    }

    @Override
    public int compareTo(Object o) {
        Employee e = (Employee) o;
        return getName().compareTo(e.getName());
    }
}
```

一般来说，用于比较的逻辑将被写成方法`compareTo`。这里我们比较雇员订单或雇员字段的`name`。两个员工同名就平等了。

现在，当在上面的代码中调用`Arrays.sort(employees);`时，我们现在知道按照年龄对雇员进行排序的逻辑和顺序是什么:

```java
[("Earl", 43, 10000),("Frank", 33, 70000), ("Jessica", 23, 4000),
 ("John", 23, 5000),("Pearl", 33, 4000), ("Steve", 26, 6000)]
```

我们可以看到这个数组是按照雇员的名字排序的——这现在成为了`Employee`类的自然顺序。

### 6.2。使用`Comparator`

现在，让我们使用一个`Comparator`接口实现对元素进行排序——在这里，我们动态地将匿名内部类传递给`Arrays.sort()` API:

```java
@Test
public void givenIntegerArray_whenUsingSort_thenSortedArray() {
    Integer [] integers = ArrayUtils.toObject(toSort);
    Arrays.sort(integers, new Comparator<Integer>() {
        @Override
        public int compare(Integer a, Integer b) {
            return Integer.compare(a, b);
        }
    });

    assertTrue(Arrays.equals(integers, ArrayUtils.toObject(sortedInts)));
}
```

现在让我们根据`salary` 对雇员进行排序，并传入另一个比较器实现:

```java
Arrays.sort(employees, new Comparator<Employee>() {
    @Override
    public int compare(Employee o1, Employee o2) {
       return Double.compare(o1.getSalary(), o2.getSalary());
    }
 });
```

基于`salary`排序的雇员数组将是:

```java
[(Jessica,23,4000.0), (John,23,5000.0), (Pearl,33,6000.0), (Steve,26,6000.0), 
(Frank,33,7000.0), (Earl,43,10000.0)] 
```

注意，我们可以以类似的方式使用`Collections.sort()`来按照自然或定制的顺序对对象的`List`和`Set`进行排序，就像上面针对数组所描述的那样。

## 7。用 Lambdas 排序

从 Java 8 开始，我们可以用 Lambdas 实现`Comparator`功能接口。

你可以看看 Java 8 中的 [Lambdas 来复习一下语法。](/web/20220805115431/https://www.baeldung.com/java-streams#lambda)

让我们替换旧的比较器:

```java
Comparator<Integer> c  = new Comparator<>() {
    @Override
    public int compare(Integer a, Integer b) {
        return Integer.compare(a, b);
    }
}
```

对于等效的实现，使用 Lambda 表达式:

```java
Comparator<Integer> c = (a, b) -> Integer.compare(a, b);
```

最后，让我们编写测试:

```java
@Test
public void givenArray_whenUsingSortWithLambdas_thenSortedArray() {
    Integer [] integersToSort = ArrayUtils.toObject(toSort);
    Arrays.sort(integersToSort, (a, b) -> {
        return Integer.compare(a, b);
    });

    assertTrue(Arrays.equals(integersToSort, 
      ArrayUtils.toObject(sortedInts)));
}
```

如你所见，这里有一个更清晰、更简洁的逻辑。

## 8。使用`Comparator.comparing`和`Comparator.thenComparing`和

Java 8 附带了两个用于排序的新 API——`Comparator`接口中的`[comparing()](https://web.archive.org/web/20220805115431/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Comparator.html#comparing(java.util.function.Function))` 和`[thenComparing()](https://web.archive.org/web/20220805115431/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Comparator.html#thenComparing(java.util.Comparator))` 。

这些对于链接`Comparator`的多个条件非常方便。

让我们考虑这样一个场景，我们可能想用`age`和`name`来比较`Employee` :

```java
@Test
public void givenArrayObjects_whenUsingComparing_thenSortedArrayObjects() {
    List<Employee> employeesList = Arrays.asList(employees);
    employees.sort(Comparator.comparing(Employee::getAge));

    assertTrue(Arrays.toString(employees.toArray())
      .equals(sortedArrayString));
}
```

在本例中，`Employee::getAge`是`Comparator`接口的排序关键字，该接口实现了具有比较功能的功能接口。

以下是排序后的员工数组:

```java
[(John,23,5000.0), (Jessica,23,4000.0), (Steve,26,6000.0), (Frank,33,7000.0), 
(Pearl,33,6000.0), (Earl,43,10000.0)]
```

这里根据`age`对员工进行排序。

我们可以看到`John`和`Jessica`年龄相同——这意味着顺序逻辑现在应该考虑他们的名字——这可以通过`thenComparing()`实现:

```java
... 
employees.sort(Comparator.comparing(Employee::getAge)
  .thenComparing(Employee::getName)); 
... 
```

使用上面的代码片段排序后，employee 数组中的元素将排序如下:

```java
[(Jessica,23,4000.0), 
 (John,23,5000.0), 
 (Steve,26,6000.0), 
 (Frank,33,7000.0), 
 (Pearl,33,6000.0), 
 (Earl,43,10000.0)
]
```

因此 `comparing()`和`thenComparing()`无疑使得更复杂的排序场景更容易实现。

## 9.结论

在本文中，我们看到了如何对`Array`、`List`、`Set`和`Map`进行排序。

我们还看到了关于 Java 8 的特性如何在排序中有用的简要介绍，比如 Lambdas、`comparing()`、`thenComparing()`和`parallelSort()`的用法。

文章中使用的所有例子都可以在 GitHub 的[上找到。](https://web.archive.org/web/20220805115431/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-2)