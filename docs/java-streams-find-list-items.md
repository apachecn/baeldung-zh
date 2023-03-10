# Java 8 Streams:根据一个列表中的值从另一个列表中查找项目

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-streams-find-list-items>

## 1。概述

在这个快速教程中，我们将学习如何使用[Java 8 Streams](/web/20220524010610/https://www.baeldung.com/java-8-streams-introduction)T3**根据一个列表中的值从另一个列表中找到项目。**

## 2。使用 Java 8 流

让我们从两个实体类开始——*雇员*和`Department`:

```java
class Employee {
    Integer employeeId;
    String employeeName;

    // getters and setters
}

class Department {
    Integer employeeId;
    String department;

    // getters and setters
} 
```

**这里的想法是基于一列`Department`对象过滤一列`Employee`对象。**更确切地说，我们想从一个列表中找到所有的`Employees `:

*   将“销售”作为他们的部门
*   在`Department`列表中有相应的`employeeId`

为了实现这一点，我们实际上是将一个过滤到另一个中:

```java
@Test
public void givenDepartmentList_thenEmployeeListIsFilteredCorrectly() {
    Integer expectedId = 1002;

    populate(emplList, deptList);

    List<Employee> filteredList = emplList.stream()
      .filter(empl -> deptList.stream()
        .anyMatch(dept -> 
          dept.getDepartment().equals("sales") && 
          empl.getEmployeeId().equals(dept.getEmployeeId())))
        .collect(Collectors.toList());

    assertEquals(1, filteredList.size());
    assertEquals(expectedId, filteredList.get(0)
      .getEmployeeId());
}
```

在填充了这两个列表之后，我们简单地将一个*雇员*对象的流传递给一个`Department`对象的流。

**接下来，为了根据我们的两个条件过滤记录，我们使用了`anyMatch`谓词**，在其中我们组合了所有给定的条件。

最后，我们将结果`collect`转化为`filteredList`。

## 3。结论

在本文中，我们学习了如何:

*   使用`Collection#s` `tream `将一个列表的值流入另一个列表，并
*   使用`anyMatch() `谓词组合多个过滤条件

这个例子的完整源代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220524010610/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-list-2)