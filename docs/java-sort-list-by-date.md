# 按日期对列表中的对象进行排序

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-sort-list-by-date>

## 1.概观

在本教程中，我们将讨论按日期对 [`List`](/web/20220706232727/https://www.baeldung.com/java-collections) 中的对象进行排序。大多数排序技术或示例都让用户按字母顺序对列表进行排序，但在本文中，我们将讨论如何用 [`Date`](/web/20220706232727/https://www.baeldung.com/java-8-date-time-intro) 对象来实现这一点。

我们将看看如何使用 Java 的`Comparator`类对列表的值进行自定义排序。

## 2.设置

让我们来看看我们将在本文中使用的`Employee`实体:

```java
public class Employee implements Comparable<Employee> {

    private String name;
    private Date joiningDate;

    public Employee(String name, Date joiningDate) {
        // ...
    }

    // standard getters and setters
}
```

我们可以注意到，我们在`Employee`类中实现了一个 [`Comparable`](/web/20220706232727/https://www.baeldung.com/java-comparator-comparable) 接口。这个接口让我们定义一个策略，用于将对象与同类型的其他对象进行比较。这用于**按照自然排序形式或由`compareTo()`方法定义对对象进行排序。**

## 3.使用`Comparable`进行分类

在 Java 中，自然顺序指的是我们应该如何对数组或集合中的原语或对象进行排序。`java.util.Arrays`和`java.util.Collections`中的 **`sort()`方法应该是一致的，并且反映了相等的语义。**

我们将使用这个方法来比较当前对象和作为参数传递的对象:

```java
public class Employee implements Comparable<Employee> {

    // ...

    @Override
    public boolean equals(Object obj) {
        return ((Employee) obj).getName().equals(getName());
    }

    @Override
    public int compareTo(Employee employee) {
        return getJoiningDate().compareTo(employee.getJoiningDate());
    }
}
```

这个`compareTo()`方法将**比较当前对象和作为参数发送的对象。**在上面的例子中，我们将当前对象的加入日期与传递过来的雇员对象进行比较。

### 3.1.按升序排序

在大多数情况下，`compareTo()`方法**描述了自然排序对象之间的比较逻辑。**这里，我们将员工的入职日期字段与同类型的其他对象进行比较。如果任何两个雇员有相同的加入日期，他们将返回 0:

```java
@Test
public void givenEmpList_SortEmpList_thenSortedListinNaturalOrder() {
    Collections.sort(employees);
    assertEquals(employees, employeesSortedByDateAsc);
}
```

现在，`Collections.sort(employees)`将根据它的`joiningDate`而不是它的主键或名字对雇员列表进行排序。我们可以看到，该列表是按照员工的`joiningDate`排序的，这现在变成了`Employee`类的自然顺序:

```java
[(Pearl,Tue Apr 27 23:30:47 IST 2021),
(Earl,Sun Feb 27 23:30:47 IST 2022),
(Steve,Sun Apr 17 23:30:47 IST 2022),
(John,Wed Apr 27 23:30:47 IST 2022)]
```

### 3.2.按降序排序

[`Collections.reverseOrder()`](https://web.archive.org/web/20220706232727/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Comparator.html#reverseOrder()) 方法**对对象进行排序，但顺序与自然排序相反。**这将返回一个比较器，该比较器将反向执行排序。当对象在比较中返回`null`时，它会抛出一个`NullPointerException`:

```java
@Test
public void givenEmpList_SortEmpList_thenSortedListinDescOrder() {
    Collections.sort(employees, Collections.reverseOrder());
    assertEquals(employees, employeesSortedByDateDesc);
}
```

## 4.使用`Comparator`进行分类

### 4.1.按升序排序

现在让我们使用`Comparator`接口实现对我们的员工列表进行排序。这里，我们将动态地将一个匿名内部类参数传递给`Collections.sort()` API:

```java
@Test
public void givenEmpList_SortEmpList_thenCheckSortedList() {

    Collections.sort(employees, new Comparator<Employee>() {
        public int compare(Employee o1, Employee o2) {
            return o1.getJoiningDate().compareTo(o2.getJoiningDate());
        }
    });

    assertEquals(employees, employeesSortedByDateAsc);
}
```

我们也可以用 Java 8 Lambda 语法替换这个语法，这样可以使我们的代码更小，如下所示:

```java
@Test
public void givenEmpList_SortEmpList_thenCheckSortedListAscLambda() {

    Collections.sort(employees, Comparator.comparing(Employee::getJoiningDate));

    assertEquals(employees, employeesSortedByDateAsc);
}
```

`compare(arg1, arg2)`方法**接受泛型类型的两个参数并返回一个整数。**由于它是从类定义中分离出来的，我们可以定义一个基于不同变量和实体的自定义比较。当我们想要定义不同的自定义排序来比较参数对象时，这很有用。

### 4.2.按降序排序

我们可以通过反转 employee 对象比较，即比较`Employee2`和`Employee1`，对给定的`Employee`列表进行降序排序。这将反转比较，从而以降序返回结果:

```java
@Test
public void givenEmpList_SortEmpList_thenCheckSortedListDescV1() {

    Collections.sort(employees, new Comparator<Employee>() {
        public int compare(Employee emp1, Employee emp2) {
            return emp2.getJoiningDate().compareTo(emp1.getJoiningDate());
        }
    });

    assertEquals(employees, employeesSortedByDateDesc);
}
```

我们还可以使用 Java 8 Lambda 表达式将上述方法转换成更简洁的形式。这将执行与上述函数相同的功能，唯一的区别是与上面的代码相比，该代码包含更少的代码行。尽管这也降低了代码的可读性。在使用 Comparator 时，我们为`Collections.sort()` API 动态传递一个匿名内部类:

```java
@Test
public void givenEmpList_SortEmpList_thenCheckSortedListDescLambda() {

    Collections.sort(employees, (emp1, emp2) -> emp2.getJoiningDate().compareTo(emp1.getJoiningDate()));
    assertEquals(employees, employeesSortedByDateDesc);
}
```

## 5.结论

在本文中，我们探索了如何在升序和降序两种模式下通过`Date`对象对 Java 集合进行排序。

我们还简要地看到了 Java 8 lambda 的一些特性，这些特性有助于排序和简化代码。

和往常一样，本文中使用的完整代码示例可以在 GitHub 上找到[。](https://web.archive.org/web/20220706232727/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-4)