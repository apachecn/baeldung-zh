# 通过列表过滤 Java 集合

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-filter-collection-by-list>

## 1。概述

通过`List`过滤`Collection`是一种常见的业务逻辑场景。有很多方法可以实现这一点。但是，如果处理不当，有些可能会导致解决方案表现不佳。

在本教程中，**我们将比较一些过滤实现，并讨论它们的优缺点**。

## 2.使用`For-Each`循环

我们将从最经典的语法开始，for-each 循环。

对于本文中的这个例子和所有其他例子，我们将使用下面的类:

```java
public class Employee {

    private Integer employeeNumber;
    private String name;
    private Integer departmentId;
    //Standard constructor, getters and setters.
}
```

为了简单起见，我们还将对所有示例使用以下方法:

```java
private List<Employee> buildEmployeeList() {
    return Arrays.asList(
      new Employee(1, "Mike", 1),
      new Employee(2, "John", 1),
      new Employee(3, "Mary", 1),
      new Employee(4, "Joe", 2),
      new Employee(5, "Nicole", 2),
      new Employee(6, "Alice", 2),
      new Employee(7, "Bob", 3),
      new Employee(8, "Scarlett", 3));
}

private List<String> employeeNameFilter() {
    return Arrays.asList("Alice", "Mike", "Bob");
}
```

在我们的例子中，我们将基于第二个有`Employee`名字的列表来过滤第一个有`Employees`的列表，以便只找到有那些特定名字的`Employees`。

现在，让我们看看传统的方法——**循环遍历两个列表寻找匹配:**

```java
@Test
public void givenEmployeeList_andNameFilterList_thenObtainFilteredEmployeeList_usingForEachLoop() {
    List<Employee> filteredList = new ArrayList<>();
    List<Employee> originalList = buildEmployeeList();
    List<String> nameFilter = employeeNameFilter();

    for (Employee employee : originalList) {
        for (String name : nameFilter) {
            if (employee.getName().equals(name)) {
                filteredList.add(employee);
                // break;
            }
        }
    }

    assertThat(filteredList.size(), is(nameFilter.size()));
}
```

这是一个简单的语法，但是它非常冗长，而且实际上非常低效。简单来说，它**迭代两个集合**的笛卡尔积，以得到我们的答案。

即使添加一个`break`来提前退出，在一般情况下仍然会以与笛卡尔积相同的顺序进行迭代。

如果我们称雇员列表的大小为`n, `，那么`nameFilter`也同样大，给我们**一个`O(n²) `分类。**

## 3.使用流和`List#contains`

**我们现在将通过使用 lambdas 来重构前面的方法，以简化语法并提高可读性**。让我们也使用`List#contains`方法作为`[lambda filter](/web/20221208143941/https://www.baeldung.com/java-stream-filter-lambda)`:

```java
@Test
public void givenEmployeeList_andNameFilterList_thenObtainFilteredEmployeeList_usingLambda() {
    List<Employee> filteredList;
    List<Employee> originalList = buildEmployeeList();
    List<String> nameFilter = employeeNameFilter();

    filteredList = originalList.stream()
      .filter(employee -> nameFilter.contains(employee.getName()))
      .collect(Collectors.toList());

    assertThat(filteredList.size(), is(nameFilter.size()));
}
```

通过使用`[Stream API](/web/20221208143941/https://www.baeldung.com/java-8-streams-introduction)`，可读性得到了极大的提高，但是我们的代码仍然像我们之前的方法一样低效，因为它的**仍然在内部迭代笛卡尔积**。因此，我们有相同的`O(n²) `分类。

## 4.通过`HashSet`使用流

为了提高性能，我们必须使用`HashSet#contains`方法。**这个方法与`List#contains`不同，因为它执行了一个`hash code` 查找，给我们一个常数时间的操作数:**

```java
@Test
public void givenEmployeeList_andNameFilterList_thenObtainFilteredEmployeeList_usingLambdaAndHashSet() {
    List<Employee> filteredList;
    List<Employee> originalList = buildEmployeeList();
    Set<String> nameFilterSet = employeeNameFilter().stream().collect(Collectors.toSet());

    filteredList = originalList.stream()
      .filter(employee -> nameFilterSet.contains(employee.getName()))
      .collect(Collectors.toList());

    assertThat(filteredList.size(), is(nameFilterSet.size()));
}
```

通过使用`[HashSet](/web/20221208143941/https://www.baeldung.com/java-hashset),`,我们的代码效率大大提高了，同时不影响可读性。**由于`HashSet#contains`以恒定时间运行，我们已经将我们的分类改进为`O(n).`**

## 5。结论

在这个快速教程中，我们学习了如何通过值的`List`来过滤`Collection`，以及使用看似最简单的方法的缺点。

我们必须始终考虑效率，因为我们的代码最终可能会在庞大的数据集中运行，而在这样的环境中，性能问题可能会带来灾难性的后果。

本文中的所有代码都可以在 GitHub 上的[处获得。](https://web.archive.org/web/20221208143941/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-list-3)