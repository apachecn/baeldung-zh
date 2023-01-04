# 卡珊德拉冻结关键字

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/cassandra-frozen-keyword>

## 1.概观

在本教程中，我们将讨论 [Apache Cassandra 数据库](https://web.archive.org/web/20220529024638/https://cassandra.apache.org/_/index.html)中的`frozen`关键字。首先，我们将展示如何声明`frozen` 集合或用户定义类型(udt)*。*接下来*，*我们将讨论使用示例以及它如何影响持久存储的基本操作。

## 延伸阅读:

## [使用 Cassandra、Astra 和 Stargate 构建仪表板](/web/20220529024638/https://www.baeldung.com/cassandra-astra-stargate-dashboard)

了解如何使用 DataStax Astra 构建仪表板，这是一种由 Apache Cassandra 和 Stargate APIs 支持的数据库即服务。[阅读更多](/web/20220529024638/https://www.baeldung.com/cassandra-astra-stargate-dashboard) →

## [用 Cassandra、Astra、REST&graph QL——记录状态更新](/web/20220529024638/https://www.baeldung.com/cassandra-astra-rest-dashboard-updates)

用 Cassandra 存储时序数据的例子。[阅读更多信息](/web/20220529024638/https://www.baeldung.com/cassandra-astra-rest-dashboard-updates) →

## [使用 Cassandra、Astra 和 CQL 构建仪表板——绘制事件数据](/web/20220529024638/https://www.baeldung.com/cassandra-astra-rest-dashboard-map)

了解如何根据 Astra 数据库中存储的数据在交互式地图上显示事件。[阅读更多](/web/20220529024638/https://www.baeldung.com/cassandra-astra-rest-dashboard-map) →

## 2.Cassandra 数据库配置

让我们使用 [docker 映像](https://web.archive.org/web/20220529024638/https://github.com/bitnami/bitnami-docker-cassandra)创建一个数据库，并使用`cqlsh`将其连接到数据库。接下来，我们应该创建一个`keyspace`:

```
CREATE KEYSPACE mykeyspace WITH replication = {'class':'SimpleStrategy', 'replication_factor' : 1};
```

对于本教程，我们创建了一个只有一个数据副本的`keyspace `。现在，让我们将客户端会话连接到一个`keyspace`:

```
USE mykeyspace;
```

## 3.冻结集合类型

**类型为`frozen`集合(`set`、`map`或`list`)的列只能整体替换其值。**换句话说，我们不能像在非冻结集合类型中那样添加、更新或删除集合中的单个元素。因此，`frozen`关键字会很有用，例如，当我们想要保护集合免受单值更新时。

此外，**t**需要冻结，我们可以使用冻结的集合作为表中的主键。我们可以通过使用像`set`、`list`或`map`这样的集合类型来声明集合列。然后我们添加集合的类型。

要声明一个`frozen`集合，我们必须在集合定义前添加关键字:

```
CREATE TABLE mykeyspace.users
(
    id         uuid PRIMARY KEY,
    ip_numbers frozen<set<inet>>,
    addresses  frozen<map<text, tuple<text>>>,
    emails     frozen<list<varchar>>,
);
```

让我们插入一些数据:

```
INSERT INTO mykeyspace.users (id, ip_numbers)
VALUES (6ab09bec-e68e-48d9-a5f8-97e6fb4c9b47, {'10.10.11.1', '10.10.10.1', '10.10.12.1'});
```

重要的是，如上所述，一个 **`frozen`集合只能作为一个整体**来替换。这意味着我们不能添加或删除元素。让我们尝试向`ip_numbers`集合添加一个新元素:

```
UPDATE mykeyspace.users
SET ip_numbers = ip_numbers + {'10.10.14.1'}
WHERE id = 6ab09bec-e68e-48d9-a5f8-97e6fb4c9b47; 
```

执行更新后，我们将得到错误:

```
InvalidRequest: Error from server: code=2200 [Invalid query] message="Invalid operation (ip_numbers = ip_numbers + {'10.10.14.1'}) for frozen collection column ip_numbers"
```

如果我们想要更新集合中的数据，我们需要更新整个集合:

```
UPDATE mykeyspace.users
SET ip_numbers = {'11.10.11.1', '11.10.10.1', '11.10.12.1'}
WHERE id = 6ab09bec-e68e-48d9-a5f8-97e6fb4c9b47; 
```

### 3.1.嵌套集合

有时我们不得不在 Cassandra 数据库中使用嵌套集合。**只有当我们将嵌套集合标记为`frozen`** 时，它们才是可能的。这意味着这个集合将是不可变的。我们可以冻结`frozen`和 `non-frozen`集合中的嵌套集合。让我们看一个例子:

```
CREATE TABLE mykeyspace.users_score
(
    id    uuid PRIMARY KEY,
    score set<frozen<set<int>>>
);
```

## 4.冻结用户定义的类型

用户定义类型(udt)可以将多个数据字段(每个字段都有名称和类型)附加到一个列。用于创建用户定义类型的字段可以是任何有效的数据类型，包括集合或其他 udt。让我们创造我们的 UDT:

```
CREATE TYPE mykeyspace.address (
    city text,
    street text,
    streetNo int,
    zipcode text
); 
```

让我们看看一个`frozen `用户定义类型的声明:

```
CREATE TABLE mykeyspace.building
(
    id      uuid PRIMARY KEY,
    address frozen<address>
);
```

当我们在用户定义的类型上使用`frozen`时，Cassandra 将该值视为 blob。这个 blob 是通过将我们的 UDT 序列化为单个值获得的。所以，**我们不能更新用户定义类型值**的一部分。我们必须覆盖整个值。

首先，让我们插入一些数据:

```
INSERT INTO mykeyspace.building (id, address)
VALUES (6ab09bec-e68e-48d9-a5f8-97e6fb4c9b48,
  {city: 'City', street: 'Street', streetNo: 2,zipcode: '02-212'});
```

让我们看看当我们试图只更新一个字段时会发生什么:

```
UPDATE mykeyspace.building
SET address.city = 'City2'
WHERE id = 6ab09bec-e68e-48d9-a5f8-97e6fb4c9b48; 
```

我们将再次得到错误:

```
InvalidRequest: Error from server: code=2200 [Invalid query] message="Invalid operation (address.city = 'City2') for frozen UDT column address"
```

因此，让我们更新整个值:

```
UPDATE mykeyspace.building
SET address = {city : 'City2', street : 'Street2'}
WHERE id = 6ab09bec-e68e-48d9-a5f8-97e6fb4c9b48;
```

这一次，地址将被更新。查询中不包括的字段用`null`值来完成。

## 5.元组

与其他组合类型不同，**一个元组总是被冻结**。因此，我们不必用`frozen`关键字来标记元组。因此，不可能只更新元组的某些元素。对于冻结的集合或 udt，我们必须覆盖整个值。

## 6.结论

在这个快速教程中，我们探索了在 Cassandra 数据库中冻结组件的基本概念。接下来，我们创建了冻结的集合和用户定义的类型。然后，我们检查了这些数据结构的行为。之后，我们讨论了元组数据类型。和往常一样，GitHub 上的[提供了这篇文章的完整源代码。](https://web.archive.org/web/20220529024638/https://github.com/eugenp/tutorials/tree/master/persistence-modules/java-cassandra)