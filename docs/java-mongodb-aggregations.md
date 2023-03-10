# 使用 Java 的 MongoDB 聚合

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-mongodb-aggregations>

## 1.概观

在本教程中，我们将使用 [MongoDB Java](/web/20220630125052/https://www.baeldung.com/java-mongodb#maven-dependencies) 驱动程序深入研究 **MongoDB 聚合框架。**

我们将首先从概念上了解聚合的含义，然后建立一个数据集。最后，我们将使用[聚合](https://web.archive.org/web/20220630125052/https://mongodb.github.io/mongo-java-driver/3.12/builders/aggregation/)生成器来看看**各种聚合技术的实际应用。**

## 2.什么是聚合？

**MongoDB 中使用聚合来分析数据并从中获取有意义的信息**。

这些通常在不同的阶段中执行，并且这些阶段形成一个管道——这样一个阶段的输出作为输入传递到下一个阶段。

最常用的阶段可以总结为:

| 阶段 | SQL 等价物 | 描述 |
| **项目** | 挑选 | 仅选择必需的字段，也可用于计算派生字段并将其添加到集合中 |
| **匹配** | 在哪里 | 根据指定的条件筛选集合 |
| **组** | 分组依据 | 根据指定的标准(如计数、求和)收集输入，为每个不同的分组返回一个文档 |
| **排序** | 以...排序 | 按给定字段的升序或降序对结果进行排序 |
| **计数** | 数数 | 统计集合中包含的文档数 |
| **极限值** | 限制 | 将结果限制为指定数量的文档，而不是返回整个集合 |
| **out** | 选择到新表中 | 将结果写入命名集合；这个阶段只能作为管道中的最后一个阶段 |

上面包含了每个聚集阶段的
**[SQL 等价物](https://web.archive.org/web/20220630125052/https://docs.mongodb.com/manual/reference/sql-aggregation-comparison/)，让我们了解上述操作在 SQL 世界中的意义。**

我们将很快查看所有这些阶段的 Java 代码示例。但在此之前，我们需要一个数据库。

## 3.数据库设置

### 3.1.资料组

学习任何与数据库相关的东西的首要要求是数据集本身！

出于本教程的目的，我们将使用一个[公开可用的 restful API 端点](https://web.archive.org/web/20220630125052/https://restcountries.com/#api-endpoints-v3)，它提供了关于世界上所有国家的全面信息。**这个 API 以一种方便的 JSON 格式为我们提供了一个国家的大量数据点**。我们将在分析中使用的一些字段是:

*   `name`–国家的名称；例如，`United States of America`
*   `alpha3Code`–国家名称的简称；例如，`IND` (印度)
*   `region`–国家所属的地区；例如，`Europe`
*   `area`–国家的地理区域
*   `languages`–数组格式的国家官方语言；例如，`English`
*   `borders`–一系列邻国的`alpha3Code`

现在让我们看看**如何将这些数据转换成 MongoDB 数据库**中的集合。

### 3.2.导入到 MongoDB

首先，我们需要**点击[API 端点来获取所有国家](https://web.archive.org/web/20220630125052/https://api.countrylayer.com/rest/v2/all)并将响应保存在本地的 JSON 文件**中。下一步是使用`mongoimport`命令将其导入 MongoDB:

```java
mongoimport.exe --db <db_name> --collection <collection_name> --file <path_to_file> --jsonArray
```

成功的导入应该会给我们一个包含 250 个文档的集合。

## 4.Java 中的聚合示例

现在我们已经了解了基础，让我们开始从所有国家的数据中获得一些有意义的见解。为此，我们将使用几个 JUnit 测试。

但在此之前，我们需要连接到数据库:

```java
@BeforeClass
public static void setUpDB() throws IOException {
    mongoClient = MongoClients.create();
    database = mongoClient.getDatabase(DATABASE);
    collection = database.getCollection(COLLECTION);
} 
```

在接下来的所有例子中，**我们将使用由 [MongoDB Java 驱动](https://web.archive.org/web/20220630125052/https://mongodb.github.io/mongo-java-driver/)提供的 [`Aggregates`](https://web.archive.org/web/20220630125052/https://mongodb.github.io/mongo-java-driver/3.10/javadoc/?com/mongodb/client/model/Aggregates.html) 助手类。**

为了提高代码片段的可读性，我们可以添加一个静态导入:

```java
import static com.mongodb.client.model.Aggregates.*;
```

### 4.1.`match`和`count`

首先，让我们从简单的事情开始。之前我们注意到数据集包含关于语言的信息。

现在，假设我们想要**检查世界上有多少国家将英语作为官方语言**:

```java
@Test
public void givenCountryCollection_whenEnglishSpeakingCountriesCounted_thenNinetyOne() {
    Document englishSpeakingCountries = collection.aggregate(Arrays.asList(
      match(Filters.eq("languages.name", "English")),
      count())).first();

    assertEquals(91, englishSpeakingCountries.get("count"));
}
```

**这里我们在聚合管道中使用了两个阶段: [`match`](https://web.archive.org/web/20220630125052/https://docs.mongodb.com/manual/reference/operator/aggregation/match/) 和 [`count`](https://web.archive.org/web/20220630125052/https://docs.mongodb.com/manual/reference/operator/aggregation/count/)** 。

首先，我们过滤掉集合，只匹配那些在`languages` 字段中包含`English`的文档。这些文档可以被想象成一个临时的或中间的集合，成为我们下一阶段的输入，`count.`它统计前一阶段的文档数量。

该示例中需要注意的另一点是方法`first`的使用。因为我们知道最后一个阶段`count`的输出将是一个单独的记录，所以这是提取出单独的结果文档的一种有保证的方法。

### 4.2.`group`(带`sum`)和`sort`

在本例中，我们的目标是**找出包含最多国家的地理区域**:

```java
@Test
public void givenCountryCollection_whenCountedRegionWise_thenMaxInAfrica() {
    Document maxCountriedRegion = collection.aggregate(Arrays.asList(
      group("$region", Accumulators.sum("tally", 1)),
      sort(Sorts.descending("tally")))).first();

    assertTrue(maxCountriedRegion.containsValue("Africa"));
}
```

很明显，**我们使用 [`group`](https://web.archive.org/web/20220630125052/https://docs.mongodb.com/manual/reference/operator/aggregation/group/) 和 [`sort`](https://web.archive.org/web/20220630125052/https://docs.mongodb.com/manual/reference/operator/aggregation/sort/) 来达到我们这里的目的**。

首先，我们通过在变量`tally.`中累加国家出现的`sum`来收集每个地区的国家数量，这给了我们一个中间的文档集合，每个文档包含两个字段:地区和其中国家的计数。然后，我们按降序对其进行排序，并提取第一个文档，以给出拥有最多国家/地区的区域。

### 4.3.`sort,` `limit,`和`out`

现在，让我们使用 **`sort`、`[limit](https://web.archive.org/web/20220630125052/http://docs.mongodb.org/manual/reference/operator/aggregation/limit/)`和 [`out`](https://web.archive.org/web/20220630125052/http://docs.mongodb.org/manual/reference/operator/aggregation/out/) 来提取区域范围内最大的七个国家，并将它们写入新的集合**:

```java
@Test
public void givenCountryCollection_whenAreaSortedDescending_thenSuccess() {
    collection.aggregate(Arrays.asList(
      sort(Sorts.descending("area")), 
      limit(7),
      out("largest_seven"))).toCollection();

    MongoCollection<Document> largestSeven = database.getCollection("largest_seven");

    assertEquals(7, largestSeven.countDocuments());

    Document usa = largestSeven.find(Filters.eq("alpha3Code", "USA")).first();

    assertNotNull(usa);
}
```

这里，我们首先按照`area.`的降序对给定的集合进行排序，然后，我们使用`Aggregates#limit`方法将结果限制为 7 个文档。最后，我们使用 **`out`阶段将这些数据反序列化到一个名为`largest_seven`** 的新集合中。现在，此收藏可以像其他收藏一样使用，例如，如果它包含`USA.`，则可以用于`find`

### 4.4.`project, group (with max), match`

在我们的最后一个示例中，让我们尝试一些更复杂的东西。比方说，我们需要**找出每个国家与其他国家共享多少条边界，最多多少条边界**。

现在在我们的数据集中，我们有一个`borders` 字段，它是一个数组，列出了这个国家**所有接壤国家的`alpha3Code`，但是没有任何字段直接给我们计数。**所以我们需要用 [`project`](https://web.archive.org/web/20220630125052/https://docs.mongodb.com/manual/reference/operator/aggregation/project/) 来推导出`borderingCountries`的数字:

```java
@Test
public void givenCountryCollection_whenNeighborsCalculated_thenMaxIsFifteenInChina() {
    Bson borderingCountriesCollection = project(Projections.fields(Projections.excludeId(), 
      Projections.include("name"), Projections.computed("borderingCountries", 
        Projections.computed("$size", "$borders"))));

    int maxValue = collection.aggregate(Arrays.asList(borderingCountriesCollection, 
      group(null, Accumulators.max("max", "$borderingCountries"))))
      .first().getInteger("max");

    assertEquals(15, maxValue);

    Document maxNeighboredCountry = collection.aggregate(Arrays.asList(borderingCountriesCollection,
      match(Filters.eq("borderingCountries", maxValue)))).first();

    assertTrue(maxNeighboredCountry.containsValue("China"));
}
```

之后，正如我们之前看到的，我们将`group`投影的集合，以找到`borderingCountries`的`max`值。这里需要指出的一点是**`max`累加器以数字**的形式给出最大值，而不是包含最大值的整个`Document`。如果要执行任何进一步的操作，我们需要执行`match`来过滤掉所需的`Document`。

## 5.结论

在本文中，我们看到了**什么是 MongoDB 聚合，以及如何使用示例数据集**在 Java 中应用它们。

我们使用了四个示例来说明各个聚合阶段，以形成对该概念的基本理解。**该框架为数据分析提供了无数可能性，可以进一步探索**。

为了便于阅读， [Spring Data MongoDB](/web/20220630125052/https://www.baeldung.com/spring-data-mongodb-guide) 提供了一种在 Java 中处理[投影和聚合](/web/20220630125052/https://www.baeldung.com/spring-data-mongodb-projections-aggregations)的替代方法。

和往常一样，源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220630125052/https://github.com/eugenp/tutorials/tree/master/persistence-modules/java-mongodb-2)