# 使用 N1QL 查询 Couchbase

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/n1ql-couchbase>

## 1。概述

在本文中，我们将研究用 [N1QL](https://web.archive.org/web/20220629001358/https://www.couchbase.com/products/n1ql) 查询 Couchbase 服务器。简而言之，这就是 SQL for NoSQL 数据库——目标是使从 SQL/关系数据库到 NoSQL 数据库系统的过渡更加容易。

有几种与 Couchbase 服务器交互的方式；这里，我们将使用 Java SDK 与数据库进行交互——这是 Java 应用程序的典型做法。

## 延伸阅读:

## [Spring 数据库简介](/web/20220629001358/https://www.baeldung.com/spring-data-couchbase)

Quick and practical into to using Spring Data Couchbase to interact with a Couchbase DB Server.[Read more](/web/20220629001358/https://www.baeldung.com/spring-data-couchbase) →

## [couch base 中的异步批处理操作](/web/20220629001358/https://www.baeldung.com/async-batch-operations-in-couchbase)

Learn how to perform efficient batch operations in Couchbase using the asynchronous Couchbase Java API.[Read more](/web/20220629001358/https://www.baeldung.com/async-batch-operations-in-couchbase) →

## [Java couch base SDK 简介](/web/20220629001358/https://www.baeldung.com/java-couchbase-sdk)

A quick and practical intro to using the Java Couchbase SDK.[Read more](/web/20220629001358/https://www.baeldung.com/java-couchbase-sdk) →

## 2。Maven 依赖关系

我们假设已经建立了一个本地 Couchbase 服务器；如果不是这样，这个[指南](https://web.archive.org/web/20220629001358/https://docs.couchbase.com/server/current/install/install-intro.html)可以帮你入门。

现在让我们将 Couchbase Java SDK 的依赖项添加到`pom.xml`:

```
<dependency>
    <groupId>com.couchbase.client</groupId>
    <artifactId>java-client</artifactId>
    <version>2.5.0</version>
</dependency>
```

Couchbase Java SDK 的最新版本可以在 [Maven Central](https://web.archive.org/web/20220629001358/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22com.couchbase.client%22%20AND%20a%3A%22java-client%22) 上找到。

我们还将使用 Jackson 库来映射查询返回的结果；让我们将它的依赖项也添加到`pom.xml`中:

```
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.13.0</version>
</dependency>
```

杰克逊图书馆的最新版本可以在 [Maven Central](https://web.archive.org/web/20220629001358/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22com.fasterxml.jackson.core%22%20AND%20a%3A%22jackson-databind%22) 上找到。

## 3。连接到 Couchbase 服务器

现在，项目已经建立了正确的依赖关系，让我们从一个 Java 应用程序连接到 Couchbase Server。

首先，我们需要启动 Couchbase 服务器——如果它还没有运行的话。

启动和停止 Couchbase 服务器的指南可以在[这里](https://web.archive.org/web/20220629001358/https://docs.couchbase.com/server/current/install/startup-shutdown.html)找到。

**让我们连接一个沙发底座`Bucket` :**

```
Cluster cluster = CouchbaseCluster.create("localhost");
Bucket bucket = cluster.openBucket("test");
```

我们所做的是连接到 Couchbase `Cluster`然后获得`Bucket`对象。

Couchbase 集群中的 bucket 的名称是`test`，可以使用 Couchbase Web 控制台创建。当我们完成所有数据库操作后，我们可以关闭已经打开的特定存储桶。

另一方面，我们可以断开与集群的连接，这将最终关闭所有存储桶:

```
bucket.close();
cluster.disconnect();
```

## 4。插入文件

Couchbase 是一个面向文档的数据库系统。让我们向`test` 桶添加一个新文档:

```
JsonObject personObj = JsonObject.create()
  .put("name", "John")
  .put("email", "[[email protected]](/web/20220629001358/https://www.baeldung.com/cdn-cgi/l/email-protection)")
  .put("interests", JsonArray.from("Java", "Nigerian Jollof"));

String id = UUID.randomUUID().toString();
JsonDocument doc = JsonDocument.create(id, personObj);
bucket.insert(doc);
```

首先，我们创建了一个 JSON `personObj`并提供了一些初始数据。键可以被看作是关系数据库系统中的列。

从 person 对象，我们使用`JsonDocument.create(),`创建了一个 JSON 文档，我们将把它插入到 bucket 中。注意，我们使用`java.util.UUID`类生成一个随机的`id`。

插入的文档可以在位于`http://localhost:8091`的 Couchbase Web 控制台中看到，或者通过使用`id`调用`bucket.get()`来看到:

```
System.out.println(bucket.get(id));
```

## 5。基本 N1QL `SELECT`查询

N1QL 是 SQL 的超集，它的语法自然看起来很相似。

例如，用于选择`test bucket`中所有文档的 N1QL 是:

```
SELECT * FROM test
```

让我们在应用程序中执行这个查询:

```
bucket.bucketManager().createN1qlPrimaryIndex(true, false);

N1qlQueryResult result
  = bucket.query(N1qlQuery.simple("SELECT * FROM test"));
```

首先，我们使用`createN1qlPrimaryIndex()`创建一个主索引，如果之前已经创建过，它将被忽略；在执行任何查询之前，必须创建它。

然后我们使用`bucket.query()`来执行 N1QL 查询。

`N1qlQueryResult`是一个`Iterable<N1qlQueryRow>`对象，因此我们可以使用`forEach()`打印出每一行:

```
result.forEach(System.out::println);
```

从返回的`result`中，我们可以通过调用`result.info()`得到`N1qlMetrics` 对象。从 metrics 对象中，我们可以了解返回的结果，例如，结果和错误计数:

```
System.out.println("result count: " + result.info().resultCount());
System.out.println("error count: " + result.info().errorCount());
```

在返回的`result`上，我们可以使用`result.parseSuccess()`来检查查询的语法是否正确，是否解析成功。我们可以使用`result.finalSuccess()`来确定查询的执行是否成功。

## 6。N1QL 查询语句

让我们看看不同的 N1QL 查询语句以及通过 Java SDK 执行它们的不同方式。

### 6.1。`SELECT`声明

NIQL 中的 `SELECT`语句就像一个标准的 SQL `SELECT`。它由三部分组成:

*   `SELECT` `–` 定义了要返回的单据的投影
*   `FROM` `–`描述从中获取文档的键空间；在 SQL 数据库系统中，键空间与表名同义
*   `WHERE` `–`指定附加过滤标准

Couchbase 服务器附带了一些样本桶(数据库)。如果在初始设置时没有加载它们，Web 控制台的`Settings`部分有一个专门的选项卡用于设置它们。

我们将使用`travel-sample`桶。`travel-sample`桶包含航空公司、地标、机场、酒店和路线的数据。数据模型可以在这里找到[。](https://web.archive.org/web/20220629001358/https://docs.couchbase.com/server/6.0/install/startup-shutdown.html)

让我们从旅行样本数据中选择 100 条航空公司记录:

```
String query = "SELECT name FROM `travel-sample` " +
  "WHERE type = 'airport' LIMIT 100";
N1qlQueryResult result1 = bucket.query(N1qlQuery.simple(query));
```

如上所述，N1QL 查询看起来非常类似于 SQL。请注意，密钥空间名称必须放在反斜杠(`)中，因为它包含一个连字符。

**`N1qlQueryResult`只是从数据库返回的原始 JSON 数据的包装器。它延伸出`Iterable<N1qlQueryRow>`并且可以循环。**

调用`result1.allRows()`将返回一个`List<N1qlQueryRow>`对象中的所有行。这对于使用`Stream` API 处理结果和/或通过索引访问每个结果非常有用:

```
N1qlQueryRow row = result1.allRows().get(0);
JsonObject rowJson = row.value();
System.out.println("Name in First Row " + rowJson.get("name"));
```

我们得到了返回结果的第一行，并使用`row.value()`得到一个`JsonObject`——它将行映射到一个键-值对，键对应于列名。

因此，我们使用`get()`得到了第一行的列 `name,`的值。就这么简单。

到目前为止，我们一直使用简单的 N1QL 查询。我们来看 N1QL 中的`parameterized`语句。

在这个查询中，我们将使用通配符(*)来选择`travel-sample`记录中的所有字段，其中`type`是一个`airport`。

`type`将作为参数传递给语句。然后我们处理返回的结果:

```
JsonObject pVal = JsonObject.create().put("type", "airport");
String query = "SELECT * FROM `travel-sample` " +
  "WHERE type = $type LIMIT 100";
N1qlQueryResult r2 = bucket.query(N1qlQuery.parameterized(query, pVal));
```

我们创建了一个 JsonObject 来保存参数作为键值对。`pVal` 对象中键'`type',` '的值将被用来替换`query`字符串中的`$type`占位符。

`N1qlQuery.parameterized()`接受包含一个或多个占位符和一个`JsonObject`的查询字符串，如上所示。

在上面的示例查询中，我们只选择了一个列–`name.` ,这使得将返回的结果映射到一个`JsonObject`变得很容易。

但是现在我们在 select 语句中使用了通配符(*)，事情就没那么简单了。返回的结果是一个原始 JSON 字符串:

```
[  
  {  
    "travel-sample":{  
      "airportname":"Calais Dunkerque",
      "city":"Calais",
      "country":"France",
      "faa":"CQF",
      "geo":{  
        "alt":12,
        "lat":50.962097,
        "lon":1.954764
      },
      "icao":"LFAC",
      "id":1254,
      "type":"airport",
      "tz":"Europe/Paris"
    }
  },
```

因此，我们需要的是一种将每一行映射到一个结构的方法，该结构允许我们通过指定列名来访问数据。

因此，让我们创建一个接受`N1qlQueryResult`的方法，然后将结果中的每一行映射到一个`JsonNode`对象。

我们选择`JsonNode`是因为它可以处理各种各样的 JSON 数据结构，并且我们可以轻松地导航它:

```
public static List<JsonNode> extractJsonResult(N1qlQueryResult result) {
  return result.allRows().stream()
    .map(row -> {
        try {
            return objectMapper.readTree(row.value().toString());
        } catch (IOException e) {
            logger.log(Level.WARNING, e.getLocalizedMessage());
            return null;
        }
    })
    .filter(Objects::nonNull)
    .collect(Collectors.toList());
}
```

我们使用`Stream` API 处理结果中的每一行。我们将每一行映射到一个`JsonNode`对象，然后将结果作为`JsonNodes.` 的`List`返回

现在，我们可以使用方法来处理上次查询返回的结果:

```
List<JsonNode> list = extractJsonResult(r2);
System.out.println(
  list.get(0).get("travel-sample").get("airportname").asText());
```

在前面显示的 JSON 输出示例中，每一行都有一个与在`SELECT` 查询中指定的键空间名称相关的键——在本例中是`travel-sample`。

所以我们得到了结果中的第一行，这是一个`JsonNode`。然后，我们遍历节点以获得`airportname`键，然后将其打印为文本。

根据返回结果的结构，前面分享的示例原始 JSON 输出更加清晰。

### 6.2。 `SELECT`使用 N1QL DSL 的声明

除了使用原始字符串来构建查询，我们还可以使用 N1QL DSL，它是我们正在使用的 Java SDK 附带的。

例如，上面的字符串查询可以用 DSL 表述如下:

```
Statement statement = select("*")
  .from(i("travel-sample"))
  .where(x("type").eq(s("airport")))
  .limit(100);
N1qlQueryResult r3 = bucket.query(N1qlQuery.simple(statement));
```

DSL 是流畅的，可以很容易地解释。数据选择类和方法在`com.couchbase.client.java.query.Select`类中。

**类似`i(), eq(), x(), s()` 的表达方法在`com.couchbase.client.java.query.dsl.Expression`类。在这里阅读更多关于 DSL [的信息。](https://web.archive.org/web/20220629001358/https://docs.couchbase.com/java-sdk/2.7/n1ql-queries-with-sdk.html)**

**N1QL select 语句还可以有`OFFSET`、`GROUP BY`和`ORDER BY`子句。**语法与标准 SQL 非常相似，可以在[这里](https://web.archive.org/web/20220629001358/https://docs.couchbase.com/server/current/n1ql/n1ql-language-reference/select-syntax.html)找到它的引用。

N1QL 的`WHERE`子句在其定义中可以接受逻辑运算符`AND`、`OR`和`NOT`。除此之外，N1QL 还提供了类似>、==、！=，`IS NULL`和[其他](https://web.archive.org/web/20220629001358/https://docs.couchbase.com/server/current/n1ql/n1ql-language-reference/comparisonops.html)。

还有其他操作符使访问存储的文档变得容易——[字符串操作符](https://web.archive.org/web/20220629001358/https://docs.couchbase.com/server/current/n1ql/n1ql-language-reference/stringops.html)可用于连接字段以形成单个字符串，而[嵌套操作符](https://web.archive.org/web/20220629001358/https://docs.couchbase.com/server/current/n1ql/n1ql-language-reference/nestedops.html)可用于分割数组并挑选字段或元素。

让我们来看看这些是如何运作的。

该查询选择 `city`列，将 `airportname`和 `faa`列连接为来自`travel-sample`桶的`portname_faa`，其中`country`列以`‘States'`结束，机场的`latitude`大于或等于 70:

```
String query2 = "SELECT t.city, " +
  "t.airportname || \" (\" || t.faa || \")\" AS portname_faa " +
  "FROM `travel-sample` t " +
  "WHERE t.type=\"airport\"" +
  "AND t.country LIKE '%States'" +
  "AND t.geo.lat >= 70 " +
  "LIMIT 2";
N1qlQueryResult r4 = bucket.query(N1qlQuery.simple(query2));
List<JsonNode> list3 = extractJsonResult(r4);
System.out.println("First Doc : " + list3.get(0));
```

我们可以使用 N1QL DSL 做同样的事情:

```
Statement st2 = select(
  x("t.city, t.airportname")
  .concat(s(" (")).concat(x("t.faa")).concat(s(")")).as("portname_faa"))
  .from(i("travel-sample").as("t"))
  .where( x("t.type").eq(s("airport"))
  .and(x("t.country").like(s("%States")))
  .and(x("t.geo.lat").gte(70)))
  .limit(2);
N1qlQueryResult r5 = bucket.query(N1qlQuery.simple(st2));
//...
```

我们再来看看 N1QL 中的其他语句。我们将在本节所学知识的基础上继续学习。

### 6.3。`INSERT` 声明

N1QL 中 insert 语句的语法是:

```
INSERT INTO `travel-sample` ( KEY, VALUE )
VALUES("unique_key", { "id": "01", "type": "airline"})
RETURNING META().id as docid, *;
```

其中`travel-sample`是键空间名称，`unique_key`是其后的值对象所需的非重复键。

最后一段是指定返回内容的`RETURNING`语句。

在这种情况下，插入文档的`id`作为`docid.` 返回。通配符(*)表示添加文档的其他属性也应该返回——与`docid.` 分开，参见下面的示例结果。

在 Couchbase Web 控制台的 Query 选项卡中执行以下语句将会在`travel-sample`存储桶中插入一条新记录:

```
INSERT INTO `travel-sample` (KEY, VALUE)
VALUES('cust1293', {"id":"1293","name":"Sample Airline", "type":"airline"})
RETURNING META().id as docid, *
```

让我们从 Java 应用程序中做同样的事情。首先，我们可以使用这样的原始查询:

```
String query = "INSERT INTO `travel-sample` (KEY, VALUE) " +
  " VALUES(" +
  "\"cust1293\", " +
  "{\"id\":\"1293\",\"name\":\"Sample Airline\", \"type\":\"airline\"})" +
  " RETURNING META().id as docid, *";
N1qlQueryResult r1 = bucket.query(N1qlQuery.simple(query));
r1.forEach(System.out::println);
```

这将单独将插入文档的`id` 作为`docid`返回，并单独返回完整的文档体:

```
{  
  "docid":"cust1293",
  "travel-sample":{  
    "id":"1293",
    "name":"Sample Airline",
    "type":"airline"
  }
}
```

然而，由于我们使用的是 Java SDK，我们可以通过创建一个`JsonDocument`来以对象的方式实现它，然后通过`Bucket` API 将它插入到 bucket 中:

```
JsonObject ob = JsonObject.create()
  .put("id", "1293")
  .put("name", "Sample Airline")
  .put("type", "airline");
bucket.insert(JsonDocument.create("cust1295", ob));
```

**不使用`insert()`，我们可以使用`upsert()`，如果存在一个具有相同唯一标识符`cust1295`** 的现有文档，它将更新文档。

现在，如果相同的惟一 id 已经存在，使用`insert()`将抛出异常。

然而，`insert()`如果成功，将返回一个`JsonDocument`,它包含唯一的 id 和插入数据的条目。

使用 N1QL 进行大容量插入的语法是:

```
INSERT INTO `travel-sample` ( KEY, VALUE )
VALUES("unique_key", { "id": "01", "type": "airline"}),
VALUES("unique_key", { "id": "01", "type": "airline"}),
VALUES("unique_n", { "id": "01", "type": "airline"})
RETURNING META().id as docid, *;
```

我们可以使用强调 SDK 的反应式 Java 来执行 Java SDK 的批量操作。让我们使用批处理将十个文档添加到一个存储桶中:

```
List<JsonDocument> documents = IntStream.rangeClosed(0,10)
  .mapToObj( i -> {
      JsonObject content = JsonObject.create()
        .put("id", i)
        .put("type", "airline")
        .put("name", "Sample Airline "  + i);
      return JsonDocument.create("cust_" + i, content);
  }).collect(Collectors.toList());

List<JsonDocument> r5 = Observable
  .from(documents)
  .flatMap(doc -> bucket.async().insert(doc))
  .toList()
  .last()
  .toBlocking()
  .single();

r5.forEach(System.out::println);
```

首先，我们生成十个文档并将它们放入一个`List;`中，然后我们使用 RxJava 执行批量操作。

最后，我们打印出每次插入的结果——这些结果已经累积形成了一个`List.`

在 Java SDK 中执行批量操作的参考可以在[这里](https://web.archive.org/web/20220629001358/https://docs.huihoo.com/couchbase/developer-guide/java-2.0/documents-bulk.html)找到。另外，insert 语句的引用可以在[这里](https://web.archive.org/web/20220629001358/https://docs.couchbase.com/server/current/n1ql/n1ql-language-reference/insert.html)找到。

### 6.4。`UPDATE`声明

N1QL 也有`UPDATE`语句。它可以更新由唯一键标识的文档。我们可以使用 update 语句来 `SET`(更新)一个属性的值或者`UNSET`(删除)一个属性。

让我们更新我们最近插入到`travel-sample`桶中的一个文档:

```
String query2 = "UPDATE `travel-sample` USE KEYS \"cust_1\" " +
  "SET name=\"Sample Airline Updated\" RETURNING name";
N1qlQueryResult result = bucket.query(N1qlQuery.simple(query2));
result.forEach(System.out::println);
```

在上面的查询中，我们将 bucket 中的`cust_1`条目的`name`属性更新为`Sample Airline Updated,`，并指示查询返回更新后的名称。

如前所述，我们也可以通过构造一个具有相同 id 的`JsonDocument`并使用`Bucket` API 的`upsert()` 来更新文档来实现同样的事情:

```
JsonObject o2 = JsonObject.create()
  .put("name", "Sample Airline Updated");
bucket.upsert(JsonDocument.create("cust_1", o2));
```

在下一个查询中，让我们使用`UNSET`命令删除`name`属性并返回受影响的文档:

```
String query3 = "UPDATE `travel-sample` USE KEYS \"cust_2\" " +
  "UNSET name RETURNING *";
N1qlQueryResult result1 = bucket.query(N1qlQuery.simple(query3));
result1.forEach(System.out::println);
```

返回的 JSON 字符串是:

```
{  
  "travel-sample":{  
    "id":2,
    "type":"airline"
  }
}
```

注意丢失的`name`属性——它已经从文档对象中删除。N1QL 更新语法参考可以在这里找到[。](https://web.archive.org/web/20220629001358/https://docs.couchbase.com/server/current/n1ql/n1ql-language-reference/update.html)

所以我们来看看插入新文档和更新文档。现在让我们看看 CRUD 首字母缩写词的最后一部分—`DELETE`。

### 6.5。`DELETE`声明

让我们使用`DELETE`查询来删除我们之前创建的一些文档。我们将使用惟一 id 来标识带有关键字`USE KEYS`的文档:

```
String query4 = "DELETE FROM `travel-sample` USE KEYS \"cust_50\"";
N1qlQueryResult result4 = bucket.query(N1qlQuery.simple(query4));
```

N1QL `DELETE`语句也带一个`WHERE`子句。所以我们可以使用条件来选择要删除的记录:

```
String query5 = "DELETE FROM `travel-sample` WHERE id = 0 RETURNING *";
N1qlQueryResult result5 = bucket.query(N1qlQuery.simple(query5));
```

我们也可以直接使用 bucket API 中的`remove()`:

```
bucket.remove("cust_2");
```

简单多了吧？是的，但是现在我们也知道如何使用 N1QL 来实现。`DELETE`语法的参考文档可以在[这里](https://web.archive.org/web/20220629001358/https://docs.couchbase.com/server/current/n1ql/n1ql-language-reference/delete.html)找到。

## 7 .**。N1QL 函数和子查询**

N1QL 不仅仅在语法上类似于 SQL 它一直延伸到一些功能。在 SQL 中，我们有一些类似于`COUNT()` 的函数可以在查询字符串中使用。

N1QL 也有同样的功能，可以用在查询字符串中。

例如，该查询将返回在`travel-sample`桶中的地标记录的总数:

```
SELECT COUNT(*) as landmark_count FROM `travel-sample` WHERE type = 'landmark'
```

在上面的例子中，我们已经在`UPDATE`语句中使用了`META`函数来返回更新文档的`id`。

有一些字符串方法可以修剪尾部空格，区分字母的大小写，甚至检查字符串是否包含标记。让我们在查询中使用这些函数:

让我们在查询中使用这些函数:

```
INSERT INTO `travel-sample` (KEY, VALUE) 
VALUES(LOWER(UUID()), 
  {"id":LOWER(UUID()), "name":"Sample Airport Rand", "created_at": NOW_MILLIS()})
RETURNING META().id as docid, *
```

上面的查询在`travel-sample` 桶中插入了一个新条目。它使用`UUID()` 函数生成一个唯一的随机 id，并使用`LOWER()`函数将其转换为小写。

`NOW_MILLIS()`方法用于将当前时间(以毫秒为单位)设置为`created_at`属性的值。N1QL 函数的完整参考可以在[这里](https://web.archive.org/web/20220629001358/https://docs.couchbase.com/server/current/n1ql/n1ql-language-reference/functions.html)找到。

子查询有时会派上用场，N1QL 为它们提供了支持。仍然使用`travel-sample`桶，让我们选择特定航空公司所有航线的目的地机场，并获取它们所在的国家:

```
SELECT DISTINCT country FROM `travel-sample` WHERE type = "airport" AND faa WITHIN 
  (SELECT destinationairport 
  FROM `travel-sample` t WHERE t.type = "route" and t.airlineid = "airline_10")
```

上述查询中的子查询包含在括号中，并返回与`airline_10`相关联的所有路由的`destinationairport`属性作为集合。

`destinationairport` 属性与`travel-sample`桶中`airport`文档上的`faa`属性相关联。`WITHIN` 关键字是 N1QL 中[集合操作符](https://web.archive.org/web/20220629001358/https://docs.couchbase.com/server/current/n1ql/n1ql-language-reference/collectionops.html)的一部分。

现在，我们已经得到了`airline_10`所有航线的目的地机场所在的国家。让我们做一些有趣的事情，在该国寻找酒店:

```
SELECT name, price, address, country FROM `travel-sample` h 
WHERE h.type = "hotel" AND h.country WITHIN
  (SELECT DISTINCT country FROM `travel-sample` 
  WHERE type = "airport" AND faa WITHIN 
  (SELECT destinationairport FROM `travel-sample` t 
  WHERE t.type = "route" and t.airlineid = "airline_10" )
  ) LIMIT 100
```

前一个查询被用作最外层查询的`WHERE`约束中的子查询。注意`DISTINCT` 关键字——它的作用与 SQL 相同——返回非重复数据。

这里的所有查询示例都可以使用本文前面演示的 SDK 来执行。

## 8。结论

N1QL 将查询基于文档的数据库(如 Couchbase)的过程提升到了另一个层次。它不仅简化了这个过程，还使从关系数据库系统切换变得更加容易。

我们在本文中看到了 N1QL 查询；主要文档可以在这里找到[。你可以在这里了解 Spring Data couch base](https://web.archive.org/web/20220629001358/https://docs.couchbase.com/server/current/n1ql/n1ql-language-reference/index.html)。

和往常一样，完整的源代码可以在 Github 上找到[。](https://web.archive.org/web/20220629001358/https://github.com/eugenp/tutorials/tree/master/couchbase)