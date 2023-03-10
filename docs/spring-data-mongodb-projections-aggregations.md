# Spring Data MongoDB:投影和聚合

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-data-mongodb-projections-aggregations>

## 1。概述

Spring Data MongoDB 为 MongoDB 原生查询语言提供了简单的高级抽象。在本文中，**我们将探讨对预测和聚合框架的支持。**

如果您对这个主题不熟悉，请参考我们的介绍性文章[Spring Data MongoDB](/web/20220926182342/https://www.baeldung.com/spring-data-mongodb-tutorial)简介。

## 2。投影

在 MongoDB 中，投影是一种从数据库中只获取文档所需字段的方法。这减少了必须从数据库服务器传输到客户机的数据量，从而提高了性能。

使用 Spring Data MongDB，投影可以与 *MongoTemplate* 和 *MongoRepository 一起使用。*

在我们继续下一步之前，让我们看看我们将使用的数据模型:

```java
@Document
public class User {
    @Id
    private String id;
    private String name;
    private Integer age;

    // standard getters and setters
}
```

### 2.1。使用`MongoTemplate` 进行预测

`Field`类上的`include()`和`exclude()`方法分别用于包含和排除字段:

```java
Query query = new Query();
query.fields().include("name").exclude("id");
List<User> john = mongoTemplate.find(query, User.class);
```

这些方法可以链接在一起，以包含或排除多个字段。除非明确排除，否则总是提取标记为`@Id`(数据库中的`_id`)的字段。

当使用投影提取记录时，排除的字段在模型类实例中是`null`。在字段属于基本类型或其包装类的情况下，被排除字段的值是基本类型的默认值。

例如，`String`会是`null`，`int` / `Integer`会是`0`，`boolean` / `Boolean`会是`false`。

因此，在上面的例子中，`name`字段将是`John`，`id`将是`null`，`age`将是`0.`

### 2.2。使用`MongoRepository` 进行预测

使用 MongoRepositories 时，`@Query`注释的`fields`可以用 JSON 格式定义:

```java
@Query(value="{}", fields="{name : 1, _id : 0}")
List<User> findNameAndExcludeId();
```

结果将与使用 MongoTemplate 一样。`value=”{}”`表示没有过滤器，因此将获取所有文档。

## 3。`Aggregation`

MongoDB 中的聚合是为了处理数据和返回计算结果而构建的。数据分阶段处理，一个阶段的输出作为输入提供给下一个阶段。这种分阶段应用转换和对数据进行计算的能力使聚合成为一种非常强大的分析工具。

Spring Data MongoDB 使用三个类为原生聚合查询提供了一个抽象:封装聚合查询的`Aggregation`、封装各个管道阶段的`AggregationOperation`以及聚合产生的结果的容器`AggregationResults`。

要执行聚合，首先使用静态构建器方法在`Aggregation`类上创建聚合管道，然后使用`Aggregation`类上的`newAggregation()`方法创建`Aggregation`的实例，最后使用`MongoTemplate`运行聚合:

```java
MatchOperation matchStage = Aggregation.match(new Criteria("foo").is("bar"));
ProjectionOperation projectStage = Aggregation.project("foo", "bar.baz");

Aggregation aggregation 
  = Aggregation.newAggregation(matchStage, projectStage);

AggregationResults<OutType> output 
  = mongoTemplate.aggregate(aggregation, "foobar", OutType.class);
```

请注意，`MatchOperation`和`ProjectionOperation`都实现了`AggregationOperation`。其他聚合管道也有类似的实现。`OutType`是预期输出的数据模型。

现在，我们将查看几个示例及其解释，以涵盖主要的聚合管道和操作符。

我们将在本文中使用的数据集列出了美国所有邮政编码的详细信息，可以从 [MongoDB 存储库](https://web.archive.org/web/20220926182342/http://media.mongodb.org/zips.json)下载。

让我们在将一个样本文档导入到数据库`test`中一个名为`zips`的集合中之后，来看看这个样本文档。

```java
{
    "_id" : "01001",
    "city" : "AGAWAM",
    "loc" : [
        -72.622739,
        42.070206
    ],
    "pop" : 15338,
    "state" : "MA"
}
```

为了简单起见，也为了让代码简洁，在接下来的代码片段中，我们将假设`Aggregation`类的所有`static`方法都是静态导入的。

### 3.1。获取所有人口超过 1000 万的州，按人口降序排列

这里我们将有三个管道:

1.  `$group`阶段汇总所有邮政编码的人口
2.  `$match`阶段过滤出人口超过 1000 万的州
3.  `$sort` stage 将所有文档按人口降序排序

预期输出将有一个字段`_id`作为 state，还有一个字段`statePop`包含总的 state 人口。让我们为此创建一个数据模型并运行聚合:

```java
public class StatePoulation {

    @Id
    private String state;
    private Integer statePop;

    // standard getters and setters
}
```

`@Id`注释将把`_id`字段从输出映射到模型中的`state`:

```java
GroupOperation groupByStateAndSumPop = group("state")
  .sum("pop").as("statePop");
MatchOperation filterStates = match(new Criteria("statePop").gt(10000000));
SortOperation sortByPopDesc = sort(Sort.by(Direction.DESC, "statePop"));

Aggregation aggregation = newAggregation(
  groupByStateAndSumPop, filterStates, sortByPopDesc);
AggregationResults<StatePopulation> result = mongoTemplate.aggregate(
  aggregation, "zips", StatePopulation.class);
```

`AggregationResults`类实现了`Iterable`,因此我们可以迭代它并打印结果。

如果输出数据模型未知，可以使用标准的 MongoDB 类`Document`。

### 3.2。根据平均城市人口得到最小的州

对于这个问题，我们将需要四个阶段:

1.  `$group`对每个城市的总人口进行求和
2.  `$group`计算每个州的平均人口
3.  `$sort`按城市平均人口升序对各州进行排序
4.  `$limit`获得平均城市人口最低的第一个州

虽然这不是必须的，但是我们将使用一个额外的`$project`阶段按照 out `StatePopulation`数据模型重新格式化文档。

```java
GroupOperation sumTotalCityPop = group("state", "city")
  .sum("pop").as("cityPop");
GroupOperation averageStatePop = group("_id.state")
  .avg("cityPop").as("avgCityPop");
SortOperation sortByAvgPopAsc = sort(Sort.by(Direction.ASC, "avgCityPop"));
LimitOperation limitToOnlyFirstDoc = limit(1);
ProjectionOperation projectToMatchModel = project()
  .andExpression("_id").as("state")
  .andExpression("avgCityPop").as("statePop");

Aggregation aggregation = newAggregation(
  sumTotalCityPop, averageStatePop, sortByAvgPopAsc,
  limitToOnlyFirstDoc, projectToMatchModel);

AggregationResults<StatePopulation> result = mongoTemplate
  .aggregate(aggregation, "zips", StatePopulation.class);
StatePopulation smallestState = result.getUniqueMappedResult();
```

在这个例子中，我们已经知道结果中只有一个文档，因为我们在最后阶段将输出文档的数量限制为 1。因此，我们可以调用`getUniqueMappedResult()`来获得所需的`StatePopulation`实例。

另一件要注意的事情是，我们没有依赖于`@Id`注释来将`_id`映射到 state，而是在投影阶段显式地完成了它。

### 3.3。获取最大和最小邮政编码的州名

对于这个例子，我们需要三个阶段:

1.  `$group`统计每个州的邮政编码数量
2.  `$sort`按照邮政编码的数量对各州进行排序
3.  `$group`使用`$first`和`$last`操作符找到带有最大和最小邮政编码的州

```java
GroupOperation sumZips = group("state").count().as("zipCount");
SortOperation sortByCount = sort(Direction.ASC, "zipCount");
GroupOperation groupFirstAndLast = group().first("_id").as("minZipState")
  .first("zipCount").as("minZipCount").last("_id").as("maxZipState")
  .last("zipCount").as("maxZipCount");

Aggregation aggregation = newAggregation(sumZips, sortByCount, groupFirstAndLast);

AggregationResults<Document> result = mongoTemplate
  .aggregate(aggregation, "zips", Document.class);
Document document= result.getUniqueMappedResult();
```

这里我们没有使用任何模型，而是使用了 MongoDB 驱动程序已经提供的`Document`。

## 4。结论

在本文中，我们学习了如何使用 Spring Data MongoDB 中的投影来获取 MongoDB 中文档的指定字段。

我们还了解了 Spring 数据中对 MongoDB 聚合框架的支持。我们讨论了主要的聚合阶段——分组、项目、排序、限制和匹配，并查看了一些实际应用的例子。完整的源代码可以在 GitHub 上找到。