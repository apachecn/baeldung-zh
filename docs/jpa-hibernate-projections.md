# JPA/Hibernate 预测

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jpa-hibernate-projections>

## 1。概述

在本教程中，我们将学习**如何使用 JPA 和 Hibernate** 来投影实体属性。

## 2。实体

首先，让我们看一下我们将在本文中使用的实体:

```java
@Entity
public class Product {
    @Id
    private long id;

    private String name;

    private String description;

    private String category;

    private BigDecimal unitPrice;

    // setters and getters
}
```

这是一个简单的实体类，表示具有各种属性的产品。

## 3。JPA 预测

尽管 JPA 规范没有明确提到投影，但是在很多情况下，我们在概念上发现了它们。

通常，JPQL 查询有一个候选实体类。该查询在执行时创建候选类的对象——使用检索到的数据填充它们的所有属性。

但是，**可以检索实体属性的一个子集，**或者列数据的一个`projection`。

除了列数据，**我们还可以投影分组函数的结果。**

### 3.1。单列投影

假设我们想列出所有产品的名称。在 JPQL 中，我们可以通过在`select`子句中只包含`name`来做到这一点:

```java
Query query = entityManager.createQuery("select name from Product");
List<Object> resultList = query.getResultList();
```

或者，我们可以对`CriteriaBuilder`做同样的事情:

```java
CriteriaBuilder builder = entityManager.getCriteriaBuilder();
CriteriaQuery<String> query = builder.createQuery(String.class);
Root<Product> product = query.from(Product.class);
query.select(product.get("name"));
List<String> resultList = entityManager.createQuery(query).getResultList();
```

因为**我们正在投影一个类型为`String`的单个列，我们期望在结果中得到一个`String` s** 的列表。因此，我们在`createQuery()`方法中将候选类指定为`String`。

由于我们想要对单个属性进行投影，**我们使用了`Query.select()`方法。**这里输入的是我们想要的属性，所以在我们的例子中，我们将使用来自`Product` 实体的`name` 属性。

现在，让我们来看看上面两个查询生成的示例输出:

```java
Product Name 1
Product Name 2
Product Name 3
Product Name 4
```

注意，**如果我们在投影中使用了`id`属性而不是`name`，查询将返回一个`Long`对象的列表。**

### 3.2。多栏投影

要使用 JPQL 在多个列上进行投影，我们只需将所有必需的列添加到`select`子句中:

```java
Query query = session.createQuery("select id, name, unitPrice from Product");
List resultList = query.getResultList();
```

但是，当使用`CriteriaBuilder`时，我们必须做一些不同的事情:

```java
CriteriaBuilder builder = session.getCriteriaBuilder();
CriteriaQuery<Object[]> query = builder.createQuery(Object[].class);
Root<Product> product = query.from(Product.class);
query.multiselect(product.get("id"), product.get("name"), product.get("unitPrice"));
List<Object[]> resultList = entityManager.createQuery(query).getResultList();
```

在这里，**我们用的是`multiselect()`而不是`select()`** 。使用这种方法，我们可以指定多个要选择的项目。

另一个显著的变化是使用了`Object[]`。**当我们选择多个项目时，查询返回一个对象数组**，其中包含每个项目的投影值。JPQL 也是如此。

让我们看看打印出来的数据是什么样的:

```java
[1, Product Name 1, 1.40]
[2, Product Name 2, 4.30]
[3, Product Name 3, 14.00]
[4, Product Name 4, 3.90]
```

正如我们所看到的，返回的数据处理起来有点麻烦。但是，幸运的是，**我们可以让 JPA 将这些数据填充到一个自定义类中。**

同样，我们可以使用`CriteriaBuilder.tuple()`或`CriteriaBuilder.construct()`来分别获得作为`Tuple`对象或自定义类对象列表的结果。

### 3.3。投影聚合函数

除了列数据之外，我们有时可能想要对数据进行分组并使用聚合函数，比如`count`和`average.`

假设我们想要找到每个类别中产品的数量。我们可以使用 JPQL 中的`count()`聚合函数来做到这一点:

```java
Query query = entityManager.createQuery("select p.category, count(p) from Product p group by p.category");
```

或者我们可以使用`CriteriaBuilder`:

```java
CriteriaBuilder builder = entityManager.getCriteriaBuilder();
CriteriaQuery<Object[]> query = builder.createQuery(Object[].class);
Root<Product> product = query.from(Product.class);
query.multiselect(product.get("category"), builder.count(product));
query.groupBy(product.get("category"));
```

这里，我们使用了`CriteriaBuilder`的`count()`方法。

使用上述任何一种方法都会产生一个对象数组列表:

```java
[category1, 2]
[category2, 1]
[category3, 1]
```

除了`count()`，`CriteriaBuilder` 还提供了其他各种聚合函数:

*   `avg` –计算组中一列的平均值
*   `max` –计算组中一列的最大值
*   `min` –计算组中一列的最小值
*   `least` –查找最少的列值(例如，按字母顺序或日期)
*   `sum` –计算组中列值的总和

## 4。冬眠预测

与 JPA 不同，Hibernate 提供了 **`org.hibernate.criterion.Projection` ，用于通过 [`Criteria`查询](/web/20221127013945/https://www.baeldung.com/hibernate-criteria-queries)** 进行投影。它还为`Projection`实例提供了一个名为`org.hibernate.criterion.Projections,` 的类和一个工厂。

### 4.1。单列投影

首先，让我们看看如何投影一个单独的列。我们将使用之前看到的示例:

```java
Criteria criteria = session.createCriteria(Product.class);
criteria = criteria.setProjection(Projections.property("name")); 
```

我们已经使用了`Criteria.setProjection()`方法来指定我们希望在查询结果中出现的属性。 **`Projections.property()`在指示要选择的列时，为我们做了与`Root.get()`和**相同的工作。

### 4.2。多栏投影

要投影多列，我们必须首先创建一个*项目列表。 **`ProjectionList`*** **是一种特殊的`Projection`包裹其他投影**以允许选择多个值*。*

我们可以使用`Projections.projectionList()` 方法创建一个*项目列表* **，比如显示`Product`的`id` 和`name`:**

```java
Criteria criteria = session.createCriteria(Product.class);
criteria = criteria.setProjection(
  Projections.projectionList()
    .add(Projections.id())
    .add(Projections.property("name")));
```

### 4.3。投影聚合函数

就像`CriteriaBuilder`一样，`Projections`类也为聚合函数提供方法。

让我们看看如何实现前面看到的 count 示例:

```java
Criteria criteria = session.createCriteria(Product.class);
criteria = criteria.setProjection(
  Projections.projectionList()
    .add(Projections.groupProperty("category"))
    .add(Projections.rowCount()));
```

需要注意的是，**我们没有在`Criteria` 对象**中直接指定 GROUP BY。调用`groupProperty` 为我们触发这个。

除了`rowCount()`函数， **`Projections`还提供了我们前面看到的聚合函数。**

### 4.4。为投影使用别名

Hibernate Criteria API 的一个有趣特性是为投影使用别名。

这在使用聚合函数时特别有用，因为我们可以在`Criterion`和`Order`实例中引用别名:

```java
Criteria criteria = session.createCriteria(Product.class);
criteria = criteria.setProjection(Projections.projectionList()
             .add(Projections.groupProperty("category"))
             .add(Projections.alias(Projections.rowCount(), "count")));
criteria.addOrder(Order.asc("count"));
```

## 5。结论

在本文中，我们看到了如何使用 JPA 和 Hibernate 来投影实体属性。

值得注意的是 **Hibernate 从 5.2 版本开始就不再支持它的 Criteria API，而支持 JPA CriteriaQuery API** 。但是，这仅仅是因为 Hibernate 团队没有时间保持两个不同的 API 同步，这两个 API 几乎做同样的事情。

当然，本文中使用的代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221127013945/https://github.com/eugenp/tutorials/tree/master/persistence-modules/java-jpa-2)