# Cassandra 是面向列的数据库还是列族数据库？

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/cassandra-column-family-data-model>

## 1.介绍

[Apache Cassandra](/web/20220627181546/https://www.baeldung.com/cassandra-with-java) 是一个开源的分布式 NoSQL 数据库，旨在处理多个数据中心的大量数据。Cassandra 的数据模型是多个文档和论文的讨论主题，经常导致混乱或矛盾的信息。这是因为 Cassandra 能够分别存储和访问列族，这导致了错误的分类，即分类为`column-oriented`而不是`column-family.`

在本教程中，我们将看看数据模型之间的差异，并建立 Cassandra 的`partitioned row store`数据模型的性质。

## 2.数据库数据模型

[Apache Cassandra git repo](https://web.archive.org/web/20220627181546/https://github.com/apache/cassandra) 上的`README`声明:

```
Cassandra is a partitioned row store. Rows are organized into tables with a required primary key.

Partitioning means that Cassandra can distribute your data across multiple machines in an application-transparent matter. Cassandra will automatically repartition as machines are added and removed from the cluster.

Row store means that like relational databases, Cassandra organizes data by rows and columns.
```

由此，我们可以断定**卡珊德拉是一个`partitioned row`** `**store**`。然而，`column-family`或`wide-column`也是合适的名字，我们将在下面找到。

**一个`column-family` 数据模型不同于一个`column-oriented`模型**。一个`column-family`数据库存储一行及其所有的列族，而一个`column-oriented`数据库只是按列而不是按行存储数据表。

### 2.1.面向行和面向列的数据存储

让我们以一个`Employees`表为例:

```
 ID         Last    First   Age
  1          Cooper  James   32
  2          Bell    Lisa    57
  3          Young   Joseph  45
```

一个`row-oriented`数据库将上述数据存储为:

```
1,Cooper,James,32;2,Bell,Lisa,57;3,Young,Joseph,45;
```

而`column-oriented` 数据库将数据存储为:

```
1,2,3;Cooper,Bell,Young;James,Lisa,Joseph;32,57,45;
```

**Cassandra 不像`row-oriented`或`column-oriented`数据库那样存储数据。**

### 2.2.分区行存储

**Cassandra 用了一个`partitioned row store`** ，意思是行包含列。一个`column-family`数据库用映射到值的键和分组到多个列族的值存储数据。

在`partitioned row store`中，`Employees`数据看起来像这样:

```
"Employees" : {
           row1 : { "ID":1, "Last":"Cooper", "First":"James", "Age":32},
           row2 : { "ID":2, "Last":"Bell", "First":"Lisa", "Age":57},
           row3 : { "ID":3, "Last":"Young", "First":"Jospeh", "Age":45},
           ...
     }
```

**A `partitioned row store`有包含列的行，但是每一行的列数不必相同**(像`big-table`)。一些行可能有数千列，而一些行可能仅限于一列。

我们可以把一个`partitioned row store`看作一个`two-dimensional key-value store`，其中一个行键和一个列键用于访问数据。**要访问数据的最小单位(一列)，我们必须首先指定行名(键)，然后指定列名。**

## 3.结论

在这篇文章中，我们了解到 Cassandra 的`partitioned row store`意味着它是 **`column-family`而不是`column-oriented`** `. `定义`column-family`的主要特征是**列信息是** **数据**的一部分。这是`column-family` 型号与`row-oriented`和`column-oriented`型号的主要区别。术语`wide-column`来源于这样一种想法，即拥有无限列数的表本质上是宽的。

我们还探索了`column-family`数据存储中的行如何不需要共享列名或列号。这将启用`schema-free`或`semi-structured`表。