# 使用 MapReduce 视图查询 Couchbase

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/couchbase-query-mapreduce-view>

## 1。概述

在本教程中，我们将介绍一些简单的 MapReduce 视图，并演示如何使用 [Couchbase Java SDK](https://web.archive.org/web/20220626084625/https://docs.couchbase.com/java-sdk/current/hello-world/start-using-sdk.html) 查询它们。

## 2。Maven 依赖关系

要在 Maven 项目中使用 Couchbase，请将 Couchbase SDK 导入到您的`pom.xml`:

```
<dependency>
    <groupId>com.couchbase.client</groupId>
    <artifactId>java-client</artifactId>
    <version>2.4.0</version>
</dependency>
```

你可以在 [Maven Central](https://web.archive.org/web/20220626084625/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22com.couchbase.client%22%20AND%20a%3A%22java-client%22) 上找到最新版本。

## 3。MapReduce 视图

在 Couchbase 中，MapReduce 视图是一种可用于查询数据桶的索引。它是使用一个 JavaScript `map`函数和一个可选的`reduce`函数定义的。

### 3.1。`map`功能

对每个文档运行一次`map`功能。创建视图时，对 bucket 中的每个文档运行一次`map`函数，结果存储在 bucket 中。

一旦创建了一个视图，`map`功能只对新插入或更新的文档运行，以便增量地更新视图。

因为`map`函数的结果存储在数据桶中，所以针对视图的查询表现出较低的延迟。

让我们看一个`map`函数的例子，它在桶中所有文档的`name`字段上创建一个索引，这些文档的`type`字段等于`“StudentGrade”`:

```
function (doc, meta) {
    if(doc.type == "StudentGrade" && doc.name) {    
        emit(doc.name, null);
    }
}
```

`emit`函数告诉 Couchbase 将哪个(些)数据字段存储在索引键(第一个参数)中，以及将什么值(第二个参数)与索引文档相关联。

在这种情况下，我们只在索引键中存储文档`name`属性。由于我们对将任何特定的值与每个条目相关联不感兴趣，所以我们将`null`作为值参数传递。

当 Couchbase 处理视图时，它创建一个由`map`函数发出的键的索引，将每个键与发出该键的所有文档相关联。

例如，如果三个文档的`name`属性被设置为`“John Doe”`，那么索引键`“John Doe”`将与这三个文档相关联。

### 3.2。`reduce`功能

`reduce`函数用于使用`map`函数的结果执行聚合计算。Couchbase 管理用户界面提供了一个简单的方法来将内置的`reduce`功能`“_count”, “_sum”,` 和 `“_stats”`应用到你的`map`功能中。

您也可以为更复杂的聚合编写自己的`reduce`函数。我们将在教程的后面看到使用内置`reduce`函数的例子。

## 4。使用视图和查询

### 4.1。组织视图

视图被组织到每个存储桶的一个或多个设计文档中。理论上，每个设计文档的视图数量没有限制。但是，为了获得最佳性能，建议您将每个设计文档限制在十个视图以下。

当您第一次在设计文档中创建一个视图时，Couchbase 将其指定为一个`development`视图。您可以对一个`development`视图运行查询来测试它的功能。一旦您对视图感到满意，您可以`publish`设计文档，视图就变成了`production`视图。

### 4.2。构建查询

为了构造针对 Couchbase 视图的查询，您需要提供它的设计文档名和视图名来创建一个`ViewQuery`对象:

```
ViewQuery query = ViewQuery.from("design-document-name", "view-name");
```

执行时，该查询将返回视图的所有行。我们将在后面的章节中看到如何基于键值限制结果集。

要构建针对开发视图的查询，您可以在创建查询时应用`development()`方法:

```
ViewQuery query 
  = ViewQuery.from("design-doc-name", "view-name").development();
```

### 4.3。执行查询

一旦我们有了一个`ViewQuery`对象，我们就可以执行查询来获得一个`ViewResult`:

```
ViewResult result = bucket.query(query);
```

### 4.4。处理查询结果

现在我们有了一个`ViewResult`，我们可以遍历这些行来获得文档 id 和/或内容:

```
for(ViewRow row : result.allRows()) {
    JsonDocument doc = row.document();
    String id = doc.id();
    String json = doc.content().toString();
}
```

## 5。示例应用程序

在本教程的剩余部分，我们将为一组学生成绩文档编写 MapReduce 视图和查询，这些文档具有以下格式，成绩限制在 0 到 100 的范围内:

```
{ 
    "type": "StudentGrade",
    "name": "John Doe",
    "course": "History",
    "hours": 3,
    "grade": 95
}
```

我们将这些文档存储在“`baeldung-tutorial`”桶中，并将所有视图存储在名为“`studentGrades`”的设计文档中让我们看一下打开存储桶所需的代码，以便我们可以查询它:

```
Bucket bucket = CouchbaseCluster.create("127.0.0.1")
  .openBucket("baeldung-tutorial");
```

## 6。精确匹配查询

假设您想要查找某门课程或一组课程的所有学生成绩。让我们使用下面的`map`函数编写一个名为`findByCourse`的视图:

```
function (doc, meta) {
    if(doc.type == "StudentGrade" && doc.course && doc.grade) {
        emit(doc.course, null);
    }
}
```

注意，在这个简单的视图中，我们只需要发出`course`字段。

### 6.1。单键匹配

为了找到历史课程的所有分数，我们将`key`方法应用于我们的基本查询:

```
ViewQuery query 
  = ViewQuery.from("studentGrades", "findByCourse").key("History");
```

### 6.2。多键匹配

如果您想要查找数学和科学课程的所有分数，您可以将`keys`方法应用于基本查询，向其传递一个键值数组:

```
ViewQuery query = ViewQuery
  .from("studentGrades", "findByCourse")
  .keys(JsonArray.from("Math", "Science"));
```

## 7。范围查询

为了查询包含一个或多个字段值范围的文档，我们需要一个发出我们感兴趣的字段的视图，并且我们必须为查询指定一个下限和/或上限。

让我们看看如何执行涉及单个字段和多个字段的范围查询。

### 7.1。涉及单个字段的查询

为了找到具有一系列`grade`值的所有文档，而不考虑`course`字段的值，我们需要一个只显示`grade`字段的视图。让我们为“`findByGrade`”视图编写`map`函数:

```
function (doc, meta) {
    if(doc.type == "StudentGrade" && doc.grade) {
        emit(doc.grade, null);
    }
}
```

让我们使用这个视图在 Java 中编写一个查询来查找所有相当于字母“B”的成绩(80 到 89，包括 80 和 89):

```
ViewQuery query = ViewQuery.from("studentGrades", "findByGrade")
  .startKey(80)
  .endKey(89)
  .inclusiveEnd(true);
```

请注意，范围查询中的起始键值总是被视为包含性的。

如果所有的分数都是整数，那么下面的查询将产生相同的结果:

```
ViewQuery query = ViewQuery.from("studentGrades", "findByGrade")
  .startKey(80)
  .endKey(90)
  .inclusiveEnd(false);
```

要找到所有“A”级(90 及以上)，我们只需指定下限:

```
ViewQuery query = ViewQuery
  .from("studentGrades", "findByGrade")
  .startKey(90);
```

为了找出所有不及格的分数(60 分以下)，我们只需指定上限:

```
ViewQuery query = ViewQuery
  .from("studentGrades", "findByGrade")
  .endKey(60)
  .inclusiveEnd(false);
```

### 7.2。涉及多个字段的查询

现在，假设我们想找出某门课程中所有分数在某个范围内的学生。这个查询需要一个新的视图，该视图发出`course`和`grade`字段。

对于多字段视图，每个索引键都作为一个值数组发出。由于我们的查询包含一个固定值的`course`和一个范围的`grade`值，我们将编写映射函数，以[ `course`，`grade` ]形式的数组发出每个键。

让我们看看视图“`findByCourseAndGrade`”的`map`函数:

```
function (doc, meta) {
    if(doc.type == "StudentGrade" && doc.course && doc.grade) {
        emit([doc.course, doc.grade], null);
    }
}
```

当在 Couchbase 中填充这个视图时，索引条目按照`course`和`grade`排序。以下是“`findByCourseAndGrade`”视图中键的子集，按其自然排序顺序显示:

```
["History", 80]
["History", 90]
["History", 94]
["Math", 82]
["Math", 88]
["Math", 97]
["Science", 78]
["Science", 86]
["Science", 92]
```

因为这个视图中的键是数组，所以在指定这个视图的范围查询的下限和上限时，也可以使用这种格式的数组。

这意味着，为了找到所有在数学课程中获得“B”级(80 到 89)的学生，您应该将下限设置为:

```
["Math", 80]
```

上限为:

```
["Math", 89]
```

让我们用 Java 编写范围查询:

```
ViewQuery query = ViewQuery
  .from("studentGrades", "findByCourseAndGrade")
  .startKey(JsonArray.from("Math", 80))
  .endKey(JsonArray.from("Math", 89))
  .inclusiveEnd(true);
```

如果我们要查找所有数学成绩为“A ”( 90 分及以上)的学生，我们可以写:

```
ViewQuery query = ViewQuery
  .from("studentGrades", "findByCourseAndGrade")
  .startKey(JsonArray.from("Math", 90))
  .endKey(JsonArray.from("Math", 100));
```

请注意，因为我们将课程值固定为“`Math`”，所以我们必须包含一个具有最高可能`grade`值的上限。否则，我们的结果集也将包括所有其`course`值在字典序上大于`Math`的文档。

并找出所有不及格的数学成绩(低于 60 分):

```
ViewQuery query = ViewQuery
  .from("studentGrades", "findByCourseAndGrade")
  .startKey(JsonArray.from("Math", 0))
  .endKey(JsonArray.from("Math", 60))
  .inclusiveEnd(false);
```

与前面的例子非常相似，我们必须指定一个最低等级的下限。否则，我们的结果集也将包括所有等级，其中`course`值在字典上小于“`Math`”。

最后，为了找到五个最高的数学分数(排除任何平局)，您可以告诉 Couchbase 执行降序排序，并限制结果集的大小:

```
ViewQuery query = ViewQuery
  .from("studentGrades", "findByCourseAndGrade")
  .descending()
  .startKey(JsonArray.from("Math", 100))
  .endKey(JsonArray.from("Math", 0))
  .inclusiveEnd(true)
  .limit(5);
```

注意，当执行降序排序时，`startKey`和`endKey`的值是相反的，因为 Couchbase 在应用`limit`之前应用排序。

## 8。聚集查询

MapReduce 视图的一个主要优点是，它们对于针对大型数据集运行聚合查询非常高效。例如，在我们的学生成绩数据集中，我们可以很容易地计算出以下总数:

*   每门课程的学生人数
*   每个学生的学分总和
*   每个学生所有课程的平均绩点

让我们使用内置的`reduce`函数为这些计算中的每一个构建一个视图和查询。

### 8.1。使用`count()`功能

首先，让我们为一个视图编写`map`函数来计算每门课程的学生人数:

```
function (doc, meta) {
    if(doc.type == "StudentGrade" && doc.course && doc.name) {
        emit([doc.course, doc.name], null);
    }
}
```

我们将这个视图称为“`countStudentsByCourse`”，并指定它使用内置的`“_count”`函数。由于我们只是执行简单的计数，我们仍然可以发出`null`作为每个条目的值。

要统计每门课程的学生人数:

```
ViewQuery query = ViewQuery
  .from("studentGrades", "countStudentsByCourse")
  .reduce()
  .groupLevel(1);
```

从聚合查询中提取数据与我们之前看到的不同。我们不是为结果中的每一行提取一个匹配的 Couchbase 文档，而是提取聚合键和结果。

让我们运行查询并将计数提取到一个`java.util.Map`:

```
ViewResult result = bucket.query(query);
Map<String, Long> numStudentsByCourse = new HashMap<>();
for(ViewRow row : result.allRows()) {
    JsonArray keyArray = (JsonArray) row.key();
    String course = keyArray.getString(0);
    long count = Long.valueOf(row.value().toString());
    numStudentsByCourse.put(course, count);
}
```

### 8.2。使用`sum()`功能

接下来，让我们编写一个视图，计算每个学生尝试的学分小时数的总和。我们将这个视图称为“`sumHoursByStudent`”，并指定它将使用内置的`“_sum”`函数:

```
function (doc, meta) {
    if(doc.type == "StudentGrade"
         && doc.name
         && doc.course
         && doc.hours) {
        emit([doc.name, doc.course], doc.hours);
    }
}
```

注意，当应用`“_sum”`函数时，我们必须`emit`每个条目的总计值——在本例中是积分数。

让我们编写一个查询来查找每个学生的总学分:

```
ViewQuery query = ViewQuery
  .from("studentGrades", "sumCreditsByStudent")
  .reduce()
  .groupLevel(1);
```

现在，让我们运行查询，并将合计金额提取到一个`java.util.Map`:

```
ViewResult result = bucket.query(query);
Map<String, Long> hoursByStudent = new HashMap<>();
for(ViewRow row : result.allRows()) {
    String name = (String) row.key();
    long sum = Long.valueOf(row.value().toString());
    hoursByStudent.put(name, sum);
}
```

### 8.3。计算平均绩点

假设我们想要计算每个学生所有课程的平均绩点(GPA ),使用基于获得的成绩和课程的学分小时数的常规绩点量表(A=每学分小时 4 分，B=每学分小时 3 分，C=每学分小时 2 分，D=每学分小时 1 分)。

没有内置的`reduce`函数来计算平均值，所以我们将结合两个视图的输出来计算 GPA。

我们已经有了`“sumHoursByStudent”`视图，汇总了每个学生尝试的学分小时数。现在我们需要每个学生获得的总分数。

让我们创建一个名为`“sumGradePointsByStudent”`的视图，它计算每门课程所获得的分数。我们将使用内置的`“_sum”`函数来简化下面的`map`函数:

```
function (doc, meta) {
    if(doc.type == "StudentGrade"
         && doc.name
         && doc.hours
         && doc.grade) {
        if(doc.grade >= 90) {
            emit(doc.name, 4*doc.hours);
        }
        else if(doc.grade >= 80) {
            emit(doc.name, 3*doc.hours);
        }
        else if(doc.grade >= 70) {
            emit(doc.name, 2*doc.hours);
        }
        else if(doc.grade >= 60) {
            emit(doc.name, doc.hours);
        }
        else {
            emit(doc.name, 0);
        }
    }
}
```

现在让我们查询这个视图，并将总和提取到一个`java.util.Map`:

```
ViewQuery query = ViewQuery.from(
  "studentGrades",
  "sumGradePointsByStudent")
  .reduce()
  .groupLevel(1);
ViewResult result = bucket.query(query);

Map<String, Long> gradePointsByStudent = new HashMap<>();
for(ViewRow row : result.allRows()) {
    String course = (String) row.key();
    long sum = Long.valueOf(row.value().toString());
    gradePointsByStudent.put(course, sum);
}
```

最后，让我们结合两个`Map`来计算每个学生的 GPA:

```
Map<String, Float> result = new HashMap<>();
for(Entry<String, Long> creditHoursEntry : hoursByStudent.entrySet()) {
    String name = creditHoursEntry.getKey();
    long totalHours = creditHoursEntry.getValue();
    long totalGradePoints = gradePointsByStudent.get(name);
    result.put(name, ((float) totalGradePoints / totalHours));
}
```

## 9。结论

我们已经演示了如何在 Couchbase 中编写一些基本的 MapReduce 视图，以及如何针对这些视图构造和执行查询，并提取结果。

本教程给出的代码可以在 [GitHub 项目](https://web.archive.org/web/20220626084625/https://github.com/eugenp/tutorials/tree/master/couchbase)中找到。

你可以在官方的 [Couchbase 开发者文档网站](https://web.archive.org/web/20220626084625/https://docs.couchbase.com/home/index.html)了解更多关于 [MapReduce 视图](https://web.archive.org/web/20220626084625/https://docs.couchbase.com/server/6.5/learn/views/views-writing.html)以及如何用 Java 查询它们[的信息。](https://web.archive.org/web/20220626084625/https://docs.couchbase.com/java-sdk/current/howtos/view-queries-with-sdk.html)