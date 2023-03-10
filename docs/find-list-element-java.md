# 如何用 Java 找到列表中的元素

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/find-list-element-java>

## 1。概述

作为开发人员，在列表中查找元素是我们经常遇到的任务。

在这个快速教程中，我们将介绍用 Java 实现这一点的不同方法。

## 延伸阅读:

## [在 Java 中检查列表是否排序](/web/20221118235115/https://www.baeldung.com/java-check-if-list-sorted)

Learn several algorithms for checking whether a list is sorted in Java.[Read more](/web/20221118235115/https://www.baeldung.com/java-check-if-list-sorted) →

## [一行 Java 列表初始化](/web/20221118235115/https://www.baeldung.com/java-init-list-one-line)

In this quick tutorial, we'll investigate how can we initialize a List using one-liners.[Read more](/web/20221118235115/https://www.baeldung.com/java-init-list-one-line) →

## 2。设置

首先让我们从定义一个`Customer` POJO 开始:

```java
public class Customer {

    private int id;
    private String name;

    // getters/setters, custom hashcode/equals
}
```

然后是一个 [`ArrayList`](/web/20221118235115/https://www.baeldung.com/java-arraylist) 的客户:

```java
List<Customer> customers = new ArrayList<>();
customers.add(new Customer(1, "Jack"));
customers.add(new Customer(2, "James"));
customers.add(new Customer(3, "Kelly")); 
```

注意，我们已经覆盖了`Customer`类中的`hashCode`和`equals`。

基于我们当前对`equals`的实现，具有相同`id`的两个`Customer`对象将被认为是相等的。

我们将在此过程中使用这个`customers`列表。

## 3。使用 Java API

Java 本身提供了几种在列表中查找项目的方法:

*   **`contains`法**
*   **`indexOf `法**
*   **临时 for 循环**
*   **`Stream `API**

### 3.1。`contains()`

`List`公开了一个名为`contains`的方法:

```java
boolean contains(Object element)
```

顾名思义，如果列表包含指定的`element,`，这个方法返回`true`，否则返回 `false`。

因此，当我们需要检查列表中是否存在某个特定项目时，我们可以:

```java
Customer james = new Customer(2, "James");
if (customers.contains(james)) {
    // ...
}
```

### 3.2。`indexOf()`

`indexOf`是另一种查找元素的有用方法:

```java
int indexOf(Object element)
```

该方法返回指定的`element`在给定列表中第一次出现的索引，如果列表不包含`element`，则返回-1。

所以从逻辑上讲，如果这个方法返回-1 以外的任何值，我们知道列表包含元素:

```java
if(customers.indexOf(james) != -1) {
    // ...
}
```

使用这种方法的主要优点是它可以告诉我们指定元素在给定列表中的位置。

### 3.3。基本循环

现在，如果我们想对一个元素进行基于字段的搜索呢？例如，假设我们正在宣布一次抽奖，我们需要宣布一个带有特定`name` 的`Customer`为获胜者。

对于这种基于字段的搜索，我们可以转向迭代。

遍历列表的传统方式是使用 Java 的循环结构之一。在每次迭代中，我们将列表中的当前项目与我们正在寻找的元素进行比较，以查看它是否匹配:

```java
public Customer findUsingEnhancedForLoop(
  String name, List<Customer> customers) {

    for (Customer customer : customers) {
        if (customer.getName().equals(name)) {
            return customer;
        }
    }
    return null;
}
```

这里的 `name`指的是我们在给定的`customers`列表中搜索的名字。该方法返回列表中第一个具有匹配的`name`的`Customer`对象，如果不存在这样的`Customer`，则返回`null`。

### 3.4。用`Iterator` 循环

[`Iterator`](/web/20221118235115/https://www.baeldung.com/java-iterator) 是我们遍历条目列表的另一种方式。

我们可以简单地拿我们之前的例子做一点调整:

```java
public Customer findUsingIterator(
  String name, List<Customer> customers) {
    Iterator<Customer> iterator = customers.iterator();
    while (iterator.hasNext()) {
        Customer customer = iterator.next();
        if (customer.getName().equals(name)) {
            return customer;
        }
    }
    return null;
}
```

因此，行为与之前相同。

### 3.5。Java 8 `Stream` API

从 Java 8 开始，我们也可以[使用`Stream` API](/web/20221118235115/https://www.baeldung.com/java-8-streams) 在`List.`中查找元素

为了在给定的列表中找到符合特定标准的元素，我们:

*   调用列表上的`stream()`
*   用适当的 `Predicate`调用`f` `ilter()`方法
*   调用`findAny() `构造，它返回第一个与包装在`Optional` 中的`filter` 谓词匹配的**元素，如果这样的元素存在的话**

```java
Customer james = customers.stream()
  .filter(customer -> "James".equals(customer.getName()))
  .findAny()
  .orElse(null);
```

为了方便起见，在`Optional`为空的情况下，我们默认使用`null`,但是这并不总是每个场景的最佳选择。

## 4。第三方库

现在，虽然 Stream API 已经足够了，但是如果我们还停留在 Java 的早期版本上，我们该怎么办？

幸运的是，有很多像 Google Guava 和 Apache Commons 这样的第三方库可供我们使用。

### 4.1。谷歌番石榴

谷歌番石榴提供的功能类似于我们对流的处理:

```java
Customer james = Iterables.tryFind(customers,
  new Predicate<Customer>() {
      public boolean apply(Customer customer) {
          return "James".equals(customer.getName());
      }
  }).orNull();
```

就像使用`Stream` API 一样，我们可以选择返回一个默认值，而不是`null`:

```java
Customer james = Iterables.tryFind(customers,
  new Predicate<Customer>() {
      public boolean apply(Customer customer) {
          return "James".equals(customer.getName());
      }
  }).or(customers.get(0));
```

如果没有找到匹配，上面的代码将选择列表中的第一个元素。

**另外，不要忘记如果列表或谓词是`null`，那么 Guava 会抛出一个`NullPointerException`。**

### 4.2 .Apache common〔t1〕

我们可以使用 Apache Commons 以几乎完全相同的方式找到一个元素:

```java
Customer james = IterableUtils.find(customers,
  new Predicate<Customer>() {
      public boolean evaluate(Customer customer) {
          return "James".equals(customer.getName());
      }
  });
```

但是有几个重要的区别:

1.  如果我们传递一个`null`列表，Apache Commons 只返回`null `。
2.  **它不像芭乐的`tryFind.`** 一样提供默认值功能

## 5。结论

在本文中，我们学习了在`List, s`中查找元素的不同方法，从快速存在检查开始，到基于字段的搜索结束。

我们还研究了第三方库`Google Guava`和`Apache Commons` 作为 Java 8 `Streams` API 的替代品。

感谢您的来访，记得在 GitHub 上查看这些例子的所有来源。