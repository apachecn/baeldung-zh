# Cassandra 中的数据建模

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/cassandra-data-modeling>

## 1。概述

Cassandra 是一个 NoSQL 数据库，它在不影响性能的情况下提供高可用性和水平可伸缩性。

为了获得 Cassandra 的最佳性能，我们需要围绕特定于当前业务问题的查询模式仔细设计模式。

在本文中，我们将回顾一些关于如何在 Cassandra 中进行数据建模的关键概念。

在继续之前，您可以浏览我们的 [Cassandra with Java](/web/20221126215334/https://www.baeldung.com/cassandra-with-java) 文章，了解基本知识以及如何使用 Java 连接 Cassandra。

## 延伸阅读:

## [使用 Cassandra、Astra 和 Stargate 构建仪表板](/web/20221126215334/https://www.baeldung.com/cassandra-astra-stargate-dashboard)

了解如何使用 DataStax Astra 构建仪表板，这是一种由 Apache Cassandra 和 Stargate APIs 支持的数据库即服务。[阅读更多](/web/20221126215334/https://www.baeldung.com/cassandra-astra-stargate-dashboard) →

## [用 Cassandra、Astra、REST&graph QL——记录状态更新](/web/20221126215334/https://www.baeldung.com/cassandra-astra-rest-dashboard-updates)

用 Cassandra 存储时序数据的例子。[阅读更多信息](/web/20221126215334/https://www.baeldung.com/cassandra-astra-rest-dashboard-updates) →

## [用卡珊德拉、阿斯特拉和 CQL 构建一个仪表盘——绘制事件数据](/web/20221126215334/https://www.baeldung.com/cassandra-astra-rest-dashboard-map)

了解如何根据存储在阿斯特拉数据库中的数据在交互式地图上显示事件。[阅读更多](/web/20221126215334/https://www.baeldung.com/cassandra-astra-rest-dashboard-map) →

## 2。分区键

Cassandra 是一个分布式数据库，数据在集群中的多个节点上进行分区和存储。

分区键由一个或多个数据字段组成，并由分区器使用**通过哈希生成令牌，以便在集群**中均匀分布数据。

## 3。聚类键

聚集键由一个或多个字段组成，有助于将具有相同分区键的行聚集或分组在一起，并按排序顺序存储它们。

假设我们将时间序列数据存储在 Cassandra 中，并希望按时间顺序检索数据。包含时间序列数据字段的聚集键对于高效检索这个用例的数据非常有帮助。

**注意:分区键和聚类键的组合构成了主键，并且唯一地标识 Cassandra 聚类中的任何记录。**

## 4。关于查询模式的指南

在开始 Cassandra 中的数据建模之前，我们应该确定查询模式，并确保它们遵循以下准则:

1.  每个查询应该从单个分区获取数据
2.  我们应该跟踪一个分区中存储了多少数据，因为 Cassandra 对单个分区中可以存储的列数有限制
3.  可以对数据进行反规范化和复制，以支持对相同数据的不同类型的查询模式

基于上面的指导方针，让我们看看一些真实世界的用例，以及我们将如何为它们建模 Cassandra 数据模型。

## 5。真实世界数据建模示例

### 5.1。脸书邮报

假设我们在 Cassandra 中存储了不同用户的脸书帖子。一种常见的查询模式是获取给定用户发布的排名靠前的'`N`'帖子。

因此，**我们需要** **按照上面的指导方针，在单个分区**上存储特定用户的所有数据。

此外，使用帖子时间戳作为聚类关键字将有助于更有效地检索顶部的'`N`'帖子。

让我们为这个用例定义 Cassandra 表模式:

```java
CREATE TABLE posts_facebook (
  user_id uuid,
  post_id timeuuid, 
  content text,
  PRIMARY KEY (user_id, post_id) )
WITH CLUSTERING ORDER BY (post_id DESC);
```

现在，让我们编写一个查询来查找用户`Anna`的前 20 篇帖子:

```java
SELECT content FROM posts_facebook WHERE user_id = "Anna_id" LIMIT 20
```

### 5.2。全国各地的健身房

假设我们存储了许多国家不同城市和州的不同合作伙伴健身房的详细信息，并且我们希望获取给定城市的健身房。

此外，假设我们需要返回按开业日期排序的健身房的结果。

基于上述准则，我们应该将位于特定州和国家的给定城市的健身房存储在单个分区上，并使用开业日期和健身房名称作为聚类键。

让我们为这个例子定义 Cassandra 表模式:

```java
CREATE TABLE gyms_by_city (
 country_code text,
 state text,
 city text,
 gym_name text,
 opening_date timestamp,
 PRIMARY KEY (
   (country_code, state_province, city), 
   (opening_date, gym_name)) 
 WITH CLUSTERING ORDER BY (opening_date ASC, gym_name ASC);
```

现在，让我们来看一个查询，该查询根据美国亚利桑那州凤凰城的开业日期获取前十家健身房:

```java
SELECT * FROM gyms_by_city
  WHERE country_code = "us" AND state = "Arizona" AND city = "Phoenix"
  LIMIT 10
```

接下来，我们来看一个查询，它获取了美国亚利桑那州凤凰城最近开业的 10 家健身房:

```java
SELECT * FROM gyms_by_city
  WHERE country_code = "us" and state = "Arizona" and city = "Phoenix"
  ORDER BY opening_date DESC 
  LIMIT 10
```

注意:由于最后一个查询的排序顺序与表创建期间定义的排序顺序相反，因此该查询将运行得较慢，因为 Cassandra 将首先获取数据，然后在内存中对其进行排序。

### 5.3。电子商务客户和产品

假设我们正在经营一家电子商务商店，我们将`Customer`和`Product`信息存储在 Cassandra 中。让我们围绕这个用例来看一些常见的查询模式:

1.  获取`Customer`信息
2.  获取`Product`信息
3.  得到所有喜欢给定`Product`的`Customers`
4.  获得所有给定的`Customer`个赞

我们将从使用单独的表来存储`Customer`和`Product`信息开始。然而，我们需要引入相当数量的非规范化来支持上面显示的第三和第四个查询。

我们将创建另外两个表来实现这一点——“T0”和“T1”。

让我们看看这个例子的 Cassandra 表模式:

```java
CREATE TABLE Customer (
  cust_id text,
  first_name text, 
  last_name text,
  registered_on timestamp, 
  PRIMARY KEY (cust_id));

CREATE TABLE Product (
  prdt_id text,
  title text,
  PRIMARY KEY (prdt_id));

CREATE TABLE Customer_By_Liked_Product (
  liked_prdt_id text,
  liked_on timestamp,
  title text,
  cust_id text,
  first_name text, 
  last_name text, 
  PRIMARY KEY (prdt_id, liked_on));

CREATE TABLE Product_Liked_By_Customer (
  cust_id text, 
  first_name text,
  last_name text,
  liked_prdt_id text, 
  liked_on timestamp,
  title text,
  PRIMARY KEY (cust_id, liked_on));
```

注意:为了支持查询、给定客户最近喜欢的产品和最近喜欢给定产品的客户，我们使用了“`liked_on`”列作为聚类键。

让我们看看这个查询，找出最近最喜欢产品“`Pepsi`”的十个客户:

```java
SELECT * FROM Customer_By_Liked_Product WHERE title = "Pepsi" LIMIT 10
```

让我们来看一个查询，该查询查找名为“`Anna`”的客户最近喜欢的产品(最多十个):

```java
SELECT * FROM Product_Liked_By_Customer 
  WHERE first_name = "Anna" LIMIT 10
```

## 6。低效的查询模式

由于 Cassandra 存储数据的方式，一些查询模式根本没有效率，包括:

*   **从多个分区获取数据**–这将需要一个协调器从多个节点获取数据，将其临时存储在堆中，然后在将结果返回给用户之前聚合数据
*   **基于连接的查询**–由于其分布式特性，Cassandra 不像关系数据库那样支持查询中的表连接，因此，使用 **连接的**查询会更慢，还会导致不一致和可用性问题****

## 7。结论

在本教程中，我们介绍了一些关于如何在 Cassandra 中进行数据建模的最佳实践。

理解核心概念并提前识别查询模式对于设计正确的数据模型以从 Cassandra 集群获得最佳性能是必要的。