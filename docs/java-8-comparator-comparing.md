# Java 8 比较器指南.比较()

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-8-comparator-comparing>

## 1。概述

Java 8 引入了对`Comparator`接口的一些增强，包括一些静态函数，这些函数在为集合排序时非常有用。

`Comparator `接口也可以有效地利用 Java 8 lambdas。关于 lambdas 和`Comparator`的详细解释可以在这里找到[，关于`Comparator`和排序的应用的编年史可以在](/web/20221026100005/https://www.baeldung.com/java-8-sort-lambda)[这里找到](/web/20221026100005/https://www.baeldung.com/java-sorting)。

在本教程中，**我们将探索 Java 8** 中为`Comparator`接口引入的几个函数。

## 2。入门

### 2.1。示例 Bean 类

对于本教程中的示例，让我们创建一个`Employee` bean，并使用它的字段进行比较和排序:

```java
public class Employee {
    String name;
    int age;
    double salary;
    long mobile;

    // constructors, getters & setters
}
```

### 2.2。我们的测试数据

我们还将创建一个雇员数组，我们将使用它来存储整个教程中各种测试用例的结果:

```java
employees = new Employee[] { ... };
```

`employees`元素的初始顺序是:

```java
[Employee(name=John, age=25, salary=3000.0, mobile=9922001), 
Employee(name=Ace, age=22, salary=2000.0, mobile=5924001), 
Employee(name=Keith, age=35, salary=4000.0, mobile=3924401)]
```

在整个教程中，我们将使用不同的函数对上面的`Employee`数组进行排序。

对于测试断言，我们将使用一组预先排序的数组，我们将把它们与不同场景下的排序结果(即`employees`数组)进行比较。

让我们声明几个这样的数组:

```java
@Before
public void initData() {
    sortedEmployeesByName = new Employee[] {...};
    sortedEmployeesByNameDesc = new Employee[] {...};
    sortedEmployeesByAge = new Employee[] {...};

    // ...
}
```

一如既往，请随意参考我们的 [GitHub 链接](https://web.archive.org/web/20221026100005/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java/)获取完整代码。

## 3。使用`Comparator.comparing`

在这一节中，我们将介绍`Comparator.comparing`静态函数的变体。

### 3.1。按键选择器变体

`Comparator.comparing`静态函数接受一个排序关键字`Function`并返回一个包含排序关键字的类型的`Comparator`:

```java
static <T,U extends Comparable<? super U>> Comparator<T> comparing(
   Function<? super T,? extends U> keyExtractor)
```

为了看到这一点，我们将使用`Employee`中的`name`字段作为排序键，并将它的方法引用作为类型为`Function.`的参数进行传递。从相同类型返回的`Comparator`用于排序:

```java
@Test
public void whenComparing_thenSortedByName() {
    Comparator<Employee> employeeNameComparator
      = Comparator.comparing(Employee::getName);

    Arrays.sort(employees, employeeNameComparator);

    assertTrue(Arrays.equals(employees, sortedEmployeesByName));
}
```

排序的结果是，`employees `数组值按名称排序:

```java
[Employee(name=Ace, age=22, salary=2000.0, mobile=5924001), 
Employee(name=John, age=25, salary=3000.0, mobile=9922001), 
Employee(name=Keith, age=35, salary=4000.0, mobile=3924401)] 
```

### 3.2。按键选择器和`Comparator`变体

还有另一个选项，通过提供一个为排序键创建自定义排序的`Comparator`,可以方便地覆盖排序键的自然排序:

```java
static <T,U> Comparator<T> comparing(
  Function<? super T,? extends U> keyExtractor,
    Comparator<? super U> keyComparator)
```

所以我们来修改一下上面的测试。我们将通过提供一个用于按降序排列姓名的`Comparator`作为`Comparator.comparing`的第二个参数，来覆盖按`name`字段排序的自然顺序:

```java
@Test
public void whenComparingWithComparator_thenSortedByNameDesc() {
    Comparator<Employee> employeeNameComparator
      = Comparator.comparing(
        Employee::getName, (s1, s2) -> {
            return s2.compareTo(s1);
        });

    Arrays.sort(employees, employeeNameComparator);

    assertTrue(Arrays.equals(employees, sortedEmployeesByNameDesc));
}
```

如我们所见，结果按照`name`降序排列:

```java
[Employee(name=Keith, age=35, salary=4000.0, mobile=3924401), 
Employee(name=John, age=25, salary=3000.0, mobile=9922001), 
Employee(name=Ace, age=22, salary=2000.0, mobile=5924001)]
```

### 3.3。使用`Comparator.reversed`

当在一个现有的`Comparator`上调用时，实例方法`Comparator.reversed`返回一个新的`Comparator`，该方法颠倒了原来的排序顺序。

我们将使用按照`name`和`reverse` it 对员工进行排序的`Comparator`，以便按照`name`的降序对员工进行排序:

```java
@Test
public void whenReversed_thenSortedByNameDesc() {
    Comparator<Employee> employeeNameComparator
      = Comparator.comparing(Employee::getName);
    Comparator<Employee> employeeNameComparatorReversed 
      = employeeNameComparator.reversed();
    Arrays.sort(employees, employeeNameComparatorReversed);
    assertTrue(Arrays.equals(employees, sortedEmployeesByNameDesc));
}
```

现在结果按`name`降序排列:

```java
[Employee(name=Keith, age=35, salary=4000.0, mobile=3924401), 
Employee(name=John, age=25, salary=3000.0, mobile=9922001), 
Employee(name=Ace, age=22, salary=2000.0, mobile=5924001)]
```

### 3.4。使用`Comparator.comparingInt`

还有一个函数，`Comparator.comparingInt,`与`Comparator.comparing`做同样的事情，但是它只使用了`int`选择器。让我们用一个例子来试试，我们通过`age`订购`employees`:

```java
@Test
public void whenComparingInt_thenSortedByAge() {
    Comparator<Employee> employeeAgeComparator 
      = Comparator.comparingInt(Employee::getAge);

    Arrays.sort(employees, employeeAgeComparator);

    assertTrue(Arrays.equals(employees, sortedEmployeesByAge));
}
```

排序后，`employees `数组值的顺序如下:

```java
[Employee(name=Ace, age=22, salary=2000.0, mobile=5924001), 
Employee(name=John, age=25, salary=3000.0, mobile=9922001), 
Employee(name=Keith, age=35, salary=4000.0, mobile=3924401)]
```

### 3.5。使用`Comparator.comparingLong`

类似于我们对`int`键所做的，让我们看一个使用`Comparator.comparingLong`的例子，通过按照`mobile`字段对`employees`数组排序来考虑`long`类型的排序键:

```java
@Test
public void whenComparingLong_thenSortedByMobile() {
    Comparator<Employee> employeeMobileComparator 
      = Comparator.comparingLong(Employee::getMobile);

    Arrays.sort(employees, employeeMobileComparator);

    assertTrue(Arrays.equals(employees, sortedEmployeesByMobile));
}
```

排序后，`employees `数组值以`mobile`为关键字，顺序如下:

```java
[Employee(name=Keith, age=35, salary=4000.0, mobile=3924401), 
Employee(name=Ace, age=22, salary=2000.0, mobile=5924001), 
Employee(name=John, age=25, salary=3000.0, mobile=9922001)]
```

### 3.6。使用`Comparator.comparingDouble`

同样，正如我们对`int`和`long`键所做的那样，让我们看一个使用`Comparator.comparingDouble`的例子，通过按照`salary`字段对`employees`数组排序来考虑`double`类型的排序键:

```java
@Test
public void whenComparingDouble_thenSortedBySalary() {
    Comparator<Employee> employeeSalaryComparator
      = Comparator.comparingDouble(Employee::getSalary);

    Arrays.sort(employees, employeeSalaryComparator);

    assertTrue(Arrays.equals(employees, sortedEmployeesBySalary));
}
```

排序后，`employees `数组值的顺序如下，其中`salary`为排序关键字:

```java
[Employee(name=Ace, age=22, salary=2000.0, mobile=5924001), 
Employee(name=John, age=25, salary=3000.0, mobile=9922001), 
Employee(name=Keith, age=35, salary=4000.0, mobile=3924401)]
```

## 4。`Comparator`在考虑自然秩序

我们可以通过`Comparable`接口实现的行为来定义自然顺序。关于`Comparator`和`Comparable`接口用法之间的差异的更多信息可以在本文的[中找到。](/web/20221026100005/https://www.baeldung.com/java-sorting)

让我们在我们的`Employee`类中实现`Comparable`，这样我们就可以尝试`Comparator`接口的`naturalOrder`和`reverseOrder`功能:

```java
public class Employee implements Comparable<Employee>{
    // ...

    @Override
    public int compareTo(Employee argEmployee) {
        return name.compareTo(argEmployee.getName());
    }
}
```

### 4.1。使用自然顺序

`naturalOrder`函数返回签名中提到的返回类型的`Comparator`:

```java
static <T extends Comparable<? super T>> Comparator<T> naturalOrder()
```

给定上述基于`name`字段比较雇员的逻辑，让我们使用这个函数获得一个`Comparator`，它以自然顺序对`employees`数组进行排序:

```java
@Test
public void whenNaturalOrder_thenSortedByName() {
    Comparator<Employee> employeeNameComparator 
      = Comparator.<Employee> naturalOrder();

    Arrays.sort(employees, employeeNameComparator);

    assertTrue(Arrays.equals(employees, sortedEmployeesByName));
}
```

排序后，`employees `数组值的顺序如下:

```java
[Employee(name=Ace, age=22, salary=2000.0, mobile=5924001), 
Employee(name=John, age=25, salary=3000.0, mobile=9922001), 
Employee(name=Keith, age=35, salary=4000.0, mobile=3924401)]
```

### 4.2。使用反向自然顺序

类似于我们使用`naturalOrder`的方式，我们将使用`reverseOrder`方法生成一个`Comparator`，它将产生一个与`naturalOrder`例子中的`employees`相反的顺序:

```java
@Test
public void whenReverseOrder_thenSortedByNameDesc() {
    Comparator<Employee> employeeNameComparator 
      = Comparator.<Employee> reverseOrder();

    Arrays.sort(employees, employeeNameComparator);

    assertTrue(Arrays.equals(employees, sortedEmployeesByNameDesc));
}
```

排序后，`employees `数组值的顺序如下:

```java
[Employee(name=Keith, age=35, salary=4000.0, mobile=3924401), 
Employee(name=John, age=25, salary=3000.0, mobile=9922001), 
Employee(name=Ace, age=22, salary=2000.0, mobile=5924001)]
```

## 5。考虑比较器中的空值

在本节中，我们将介绍`nullsFirst`和`nullsLast` 函数，它们在排序时考虑`null`值，并将`null`值保持在排序序列的开头或结尾。

### 5.1。首先考虑空值

让我们在`employees`数组中随机插入`null`值:

```java
[Employee(name=John, age=25, salary=3000.0, mobile=9922001), 
null, 
Employee(name=Ace, age=22, salary=2000.0, mobile=5924001), 
null, 
Employee(name=Keith, age=35, salary=4000.0, mobile=3924401)]
```

`nullsFirst`函数将返回一个`Comparator`,将所有的`nulls`保持在排序序列的开头:

```java
@Test
public void whenNullsFirst_thenSortedByNameWithNullsFirst() {
    Comparator<Employee> employeeNameComparator
      = Comparator.comparing(Employee::getName);
    Comparator<Employee> employeeNameComparator_nullFirst
      = Comparator.nullsFirst(employeeNameComparator);

    Arrays.sort(employeesArrayWithNulls, 
      employeeNameComparator_nullFirst);

    assertTrue(Arrays.equals(
      employeesArrayWithNulls,
      sortedEmployeesArray_WithNullsFirst));
}
```

排序后，`employees `数组值的顺序如下:

```java
[null, 
null, 
Employee(name=Ace, age=22, salary=2000.0, mobile=5924001), 
Employee(name=John, age=25, salary=3000.0, mobile=9922001), 
Employee(name=Keith, age=35, salary=4000.0, mobile=3924401)]
```

### 5.2。考虑到最后一个为空

`nullsLast`函数将返回一个`Comparator`，它将所有的`nulls`保持在排序序列的末尾:

```java
@Test
public void whenNullsLast_thenSortedByNameWithNullsLast() {
    Comparator<Employee> employeeNameComparator
      = Comparator.comparing(Employee::getName);
    Comparator<Employee> employeeNameComparator_nullLast
      = Comparator.nullsLast(employeeNameComparator);

    Arrays.sort(employeesArrayWithNulls, employeeNameComparator_nullLast);

    assertTrue(Arrays.equals(
      employeesArrayWithNulls, sortedEmployeesArray_WithNullsLast));
}
```

排序后，`employees `数组值的顺序如下:

```java
[Employee(name=Ace, age=22, salary=2000.0, mobile=5924001), 
Employee(name=John, age=25, salary=3000.0, mobile=9922001), 
Employee(name=Keith, age=35, salary=4000.0, mobile=3924401), 
null, 
null]
```

## 6。使用`Comparator.thenComparing`

`thenComparing`函数让我们通过以特定的顺序提供多个排序键来设置值的字典顺序。

让我们看看`Employee`类的另一个数组:

```java
someMoreEmployees = new Employee[] { ... };
```

我们将考虑上述数组中的以下元素序列:

```java
[Employee(name=Jake, age=25, salary=3000.0, mobile=9922001), 
Employee(name=Jake, age=22, salary=2000.0, mobile=5924001), 
Employee(name=Ace, age=22, salary=3000.0, mobile=6423001), 
Employee(name=Keith, age=35, salary=4000.0, mobile=3924401)]
```

然后，我们将编写一个比较序列，如`age`后跟`name,`，并查看该数组的排序:

```java
@Test
public void whenThenComparing_thenSortedByAgeName(){
    Comparator<Employee> employee_Age_Name_Comparator
      = Comparator.comparing(Employee::getAge)
        .thenComparing(Employee::getName);

    Arrays.sort(someMoreEmployees, employee_Age_Name_Comparator);

    assertTrue(Arrays.equals(someMoreEmployees, sortedEmployeesByAgeName));
}
```

这里排序将由`age`完成，对于具有相同`age`的值，排序将由`name`完成。我们可以在排序后收到的序列中看到这一点:

```java
[Employee(name=Ace, age=22, salary=3000.0, mobile=6423001), 
Employee(name=Jake, age=22, salary=2000.0, mobile=5924001), 
Employee(name=Jake, age=25, salary=3000.0, mobile=9922001), 
Employee(name=Keith, age=35, salary=4000.0, mobile=3924401)]
```

现在我们可以使用另一个版本的`thenComparing`、`thenComparingInt`，将字典顺序改为`name`后接`age`:

```java
@Test
public void whenThenComparing_thenSortedByNameAge() {
    Comparator<Employee> employee_Name_Age_Comparator
      = Comparator.comparing(Employee::getName)
        .thenComparingInt(Employee::getAge);

    Arrays.sort(someMoreEmployees, employee_Name_Age_Comparator);

    assertTrue(Arrays.equals(someMoreEmployees, 
      sortedEmployeesByNameAge));
}
```

排序后，`employees `数组值的顺序如下:

```java
[Employee(name=Ace, age=22, salary=3000.0, mobile=6423001), 
Employee(name=Jake, age=22, salary=2000.0, mobile=5924001), 
Employee(name=Jake, age=25, salary=3000.0, mobile=9922001), 
Employee(name=Keith, age=35, salary=4000.0, mobile=3924401)]
```

同样，功能`thenComparingLong`和`thenComparingDouble` 分别用于使用`long`和`double`分类键`,` 。

## 7 .**。结论**

本文是 Java 8 中为`Comparator`接口引入的几个特性的指南。

和往常一样，源代码可以在 Github 上找到[。](https://web.archive.org/web/20221026100005/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-8)