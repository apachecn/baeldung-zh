# Apache Commons 集合指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/apache-commons-collection-utils>

[This article is part of a series:](javascript:void(0);)[• Apache Commons Collections Bag](/web/20220926153229/https://www.baeldung.com/apache-commons-bag)
[• Apache Commons Collections SetUtils](/web/20220926153229/https://www.baeldung.com/apache-commons-setutils)
[• Apache Commons Collections OrderedMap](/web/20220926153229/https://www.baeldung.com/apache-commons-ordered-map)
[• Apache Commons Collections BidiMap](/web/20220926153229/https://www.baeldung.com/commons-collections-bidi-map)
• A Guide to Apache Commons Collections CollectionUtils (current article)[• Apache Commons Collections MapUtils](/web/20220926153229/https://www.baeldung.com/apache-commons-map-utils)
[• Guide to Apache Commons CircularFifoQueue](/web/20220926153229/https://www.baeldung.com/commons-circular-fifo-queue)

## 1。概述

简单地说， `Apache` `CollectionUtils`为常见操作提供了实用方法，涵盖了广泛的用例，有助于避免编写样板代码。该库面向旧版本的 JVM，因为目前 Java 8 的`Stream` API 提供了类似的功能。

## 2.Maven 依赖性

我们需要添加下面的依赖关系来开始`CollectionUtils:`

```java
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-collections4</artifactId>
    <version>4.1</version>
</dependency>
```

这个库的最新版本可以在[这里](https://web.archive.org/web/20220926153229/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.apache.commons%22%20AND%20a%3A%22commons-collections4%22)找到。

## 3。设置

让我们加上`Customer` 和 `Address classes:`

```java
public class Customer {
    private Integer id;
    private String name;
    private Address address;

    // standard getters and setters
}

public class Address {
    private String locality;
    private String city;

    // standard getters and setters
}
```

我们还将准备好下面的`Customer` 和`List` 实例来测试我们的实现:

```java
Customer customer1 = new Customer(1, "Daniel", "locality1", "city1");
Customer customer2 = new Customer(2, "Fredrik", "locality2", "city2");
Customer customer3 = new Customer(3, "Kyle", "locality3", "city3");
Customer customer4 = new Customer(4, "Bob", "locality4", "city4");
Customer customer5 = new Customer(5, "Cat", "locality5", "city5");
Customer customer6 = new Customer(6, "John", "locality6", "city6");

List<Customer> list1 = Arrays.asList(customer1, customer2, customer3);
List<Customer> list2 = Arrays.asList(customer4, customer5, customer6);
List<Customer> list3 = Arrays.asList(customer1, customer2);

List<Customer> linkedList1 = new LinkedList<>(list1);
```

## 4。`CollectionUtils`

让我们来看看`Apache Commons CollectionUtils`类中一些最常用的方法。

### 4.1。仅添加非空元素

**我们可以使用`CollectionUtils's addIgnoreNull` 方法只将非空元素添加到提供的集合中。**

此方法的第一个参数是我们要添加元素的集合，第二个参数是我们要添加的元素:

```java
@Test
public void givenList_whenAddIgnoreNull_thenNoNullAdded() {
    CollectionUtils.addIgnoreNull(list1, null);

    assertFalse(list1.contains(null));
}
```

请注意，`null`没有被添加到列表中。

### 4.2。整理列表

**我们可以使用`collate` 方法来整理两个已经排序的列表。**这个方法将我们想要合并的两个列表作为参数，并返回一个排序列表:

```java
@Test
public void givenTwoSortedLists_whenCollated_thenSorted() {
    List<Customer> sortedList = CollectionUtils.collate(list1, list2);

    assertEquals(6, sortedList.size()); 
    assertTrue(sortedList.get(0).getName().equals("Bob"));
    assertTrue(sortedList.get(2).getName().equals("Daniel"));
}
```

### 4.3。变换对象

我们可以使用`transform`方法将 A 类的对象转换成 b 类的不同对象，这个方法以 A 类的对象列表和 a `transformer`作为参数。

该操作的结果是 B 类对象的列表:

```java
@Test
public void givenListOfCustomers_whenTransformed_thenListOfAddress() {
    Collection<Address> addressCol = CollectionUtils.collect(list1, 
      new Transformer<Customer, Address>() {
        public Address transform(Customer customer) {
            return customer.getAddress();
        }
    });

    List<Address> addressList = new ArrayList<>(addressCol);
    assertTrue(addressList.size() == 3);
    assertTrue(addressList.get(0).getLocality().equals("locality1"));
}
```

### 4.4。过滤对象

**使用`filter` 我们可以从列表中删除不满足给定条件的对象** `**.**` 该方法将列表作为第一个参数，将`Predicate` 作为第二个参数。

**`filterInverse` 方法正好相反。**当`Predicate` 返回 true 时，它从列表中删除对象。

**如果输入列表被修改，即至少有一个对象从列表中被过滤掉:**，则`filter` 和`filterInverse` 都返回`true`

```java
@Test
public void givenCustomerList_WhenFiltered_thenCorrectSize() {

    boolean isModified = CollectionUtils.filter(linkedList1, 
      new Predicate<Customer>() {
        public boolean evaluate(Customer customer) {
            return Arrays.asList("Daniel","Kyle").contains(customer.getName());
        }
    });

    assertTrue(linkedList1.size() == 2);
}
```

如果我们希望返回结果列表而不是布尔标志，我们可以使用`**select**` 和`**selectRejected**` 。

### 4.5。检查非空

当我们想检查一个列表中是否至少有一个元素时，这个方法非常方便。同样的另一种检查方式是:

```java
boolean isNotEmpty = (list != null && list.size() > 0);
```

虽然上面的代码行做了同样的事情，但是`CollectionUtils.isNotEmpty` 让我们的代码更干净:

```java
@Test
public void givenNonEmptyList_whenCheckedIsNotEmpty_thenTrue() {
    assertTrue(CollectionUtils.isNotEmpty(list1));
}
```

**`isEmpty`则相反。**它检查给定的列表是否为空或者列表中是否有零个元素:

```java
List<Customer> emptyList = new ArrayList<>();
List<Customer> nullList = null;

assertTrue(CollectionUtils.isEmpty(nullList));
assertTrue(CollectionUtils.isEmpty(emptyList));
```

### 4.6。检查内含物

我们可以使用`isSubCollection` 来检查一个集合是否包含在另一个集合中。`isSubCollection` 接受两个集合作为参数，如果第一个集合是第二个集合的子集合，则返回`true` :

```java
@Test
public void givenCustomerListAndASubcollection_whenChecked_thenTrue() {
    assertTrue(CollectionUtils.isSubCollection(list3, list1));
}
```

如果一个对象在第一个集合中出现的次数小于或等于它在第二个集合中出现的次数，则该集合是另一个集合的子集合。

### 4.7。集合的交集

**我们可以用`CollectionUtils.intersection` 的方法得到两个集合的交集。**该方法采用两个集合，并返回两个输入集合中相同元素的集合:

```java
@Test
public void givenTwoLists_whenIntersected_thenCheckSize() {
    Collection<Customer> intersection = CollectionUtils.intersection(list1, list3);
    assertTrue(intersection.size() == 2);
}
```

一个元素在结果集合中出现的次数是它在每个给定集合中出现的次数的最小值。

### 4.8。减去收款

`**CollectionUtils.subtract**` 接受两个集合作为输入，返回一个集合，该集合包含第一个集合中有而第二个集合中没有的元素:

```java
@Test
public void givenTwoLists_whenSubtracted_thenCheckElementNotPresentInA() {
    Collection<Customer> result = CollectionUtils.subtract(list1, list3);
    assertFalse(result.contains(customer1));
}
```

一个集合在结果中出现的次数等于它在第一个集合中出现的次数减去它在第二个集合中出现的次数。

### 4.9。集合的联合

执行两个集合的并集运算，并返回一个包含第一个或第二个集合中所有元素的集合。

```java
@Test
public void givenTwoLists_whenUnioned_thenCheckElementPresentInResult() {
    Collection<Customer> union = CollectionUtils.union(list1, list2);

    assertTrue(union.contains(customer1));
    assertTrue(union.contains(customer4));
}
```

元素在结果集合中出现的次数是它在每个给定集合中出现的次数的最大值。

## 5。结论

我们结束了。

我们学习了一些常用的`CollectionUtils` 方法——当我们在 Java 项目中使用集合时，这对于避免样板文件非常有用。

像往常一样，代码可以在 GitHub 上获得。

Next **»**[Apache Commons Collections MapUtils](/web/20220926153229/https://www.baeldung.com/apache-commons-map-utils)**«** Previous[Apache Commons Collections BidiMap](/web/20220926153229/https://www.baeldung.com/commons-collections-bidi-map)