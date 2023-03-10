# 在流过滤器上计数匹配

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-stream-filter-count>

## 1。概述

在本教程中，我们将探索`Stream.count()`方法的使用。具体来说，**我们将看看如何将`count()`方法与`filter()`方法结合起来，来计算我们所应用的`Predicate`** 的匹配数。

## 2。使用`Stream.count()`

`count()`方法本身提供了一个很小但非常有用的功能。我们还可以将它与其他工具很好地结合起来，例如与`Stream.filter()`。

让我们使用我们在[教程中为`Stream.filter()`](/web/20220930105629/https://www.baeldung.com/java-stream-filter-lambda) 定义的同一个`Customer`类:

```java
public class Customer {
    private String name;
    private int points;
    //Constructor and standard getters
}
```

此外，我们还创建了相同的客户群:

```java
Customer john = new Customer("John P.", 15);
Customer sarah = new Customer("Sarah M.", 200);
Customer charles = new Customer("Charles B.", 150);
Customer mary = new Customer("Mary T.", 1);

List<Customer> customers = Arrays.asList(john, sarah, charles, mary);
```

接下来，我们将在列表上应用`Stream`方法来过滤它，并确定我们的过滤器得到多少匹配。

### 2.1。计数元素

让我们来看看`count()`的基本用法:

```java
long count = customers.stream().count();

assertThat(count).isEqualTo(4L);
```

注意，`count()`返回一个`long`值。

### 2.2。使用`count()`与`filter()`和

上一小节中的例子并没有给人留下深刻的印象。我们可以用`List.size()`方法得出同样的结果。

当我们将 **`Stream.count()`与其他`Stream`方法结合时，它真的大放异彩——最常见的是与`filter()` :** 结合

```java
long countBigCustomers = customers
  .stream()
  .filter(c -> c.getPoints() > 100)
  .count();

assertThat(countBigCustomers).isEqualTo(2L);
```

在这个例子中，我们对客户列表应用了一个过滤器，并且我们还获得了满足条件的客户数量。在这种情况下，我们有两个超过 100 分的客户。

当然，也可能没有元素与我们的过滤器匹配:

```java
long count = customers
  .stream()
  .filter(c -> c.getPoints() > 500)
  .count();

assertThat(count).isEqualTo(0L); 
```

### 2.3。使用带高级过滤器的`count()`

在我们关于`filter()`的教程中，我们看到了该方法的一些更高级的用例。当然，我们还是可以统计出这样的`filter()`操作的结果。

**我们可以根据多个标准过滤收藏:**

```java
long count = customers
  .stream()
  .filter(c -> c.getPoints() > 10 && c.getName().startsWith("Charles"))
  .count();

assertThat(count).isEqualTo(1L);
```

在这里，我们筛选并统计了姓名以“Charles”开头且积分超过 10 的客户的数量。

**我们也可以将标准提取到自己的方法中，并使用方法引用:**

```java
long count = customers
  .stream()
  .filter(Customer::hasOverHundredPoints)
  .count();

assertThat(count).isEqualTo(2L);
```

## 3。结论

在本文中，我们看到了一些如何结合使用`count()`方法和`filter()`方法来处理流的例子。对于`count()`的进一步用例，查看返回一个`Stream`的其他方法，比如在[我们关于用`concat()`](/web/20220930105629/https://www.baeldung.com/java-merge-streams) 合并流的教程中显示的那些方法。

和往常一样，完整的代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220930105629/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-streams)