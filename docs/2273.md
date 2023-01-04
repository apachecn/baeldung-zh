# 用 Java 将 JDBC 结果集转换成 JSON

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-jdbc-convert-resultset-to-json>

## 1.概观

在某些场景中，我们可能需要通过 API 调用将数据库查询的结果发送到另一个系统或消息传递平台。对于这种情况，我们通常使用 JSON 作为数据交换格式。

在本教程中，我们将看到多种方法将一个 [JDBC](/web/20220627155255/https://www.baeldung.com/java-jdbc) `ResultSet`对象转换为 [JSON](/web/20220627155255/https://www.baeldung.com/java-json) 格式。

## 2.代码示例

我们将使用 [H2](/web/20220627155255/https://www.baeldung.com/spring-boot-h2-database) 数据库作为我们的代码示例。我们有一个示例 CSV 文件，我们使用 JDBC 将它读入一个表`words`。下面是示例 CSV 文件中的三行，第一行是标题:

```
Username,Id,First name,Last name
doe1,7173,John,Doe
smith3,3722,Dana,Smith
john22,5490,John,Wang
```

构成`ResultSet`的代码行如下所示:

```
ResultSet resultSet = stmt.executeQuery("SELECT * FROM words");
```

对于 JSON 处理，我们使用 [JSON-Java](/web/20220627155255/https://www.baeldung.com/java-org-json) ( `org.json`)库。首先，我们将[及其相应的依赖项](https://web.archive.org/web/20220627155255/https://mvnrepository.com/artifact/org.json/json)添加到我们的 POM 文件中:

```
<dependency>
    <groupId>org.json</groupId>
    <artifactId>json</artifactId>
    <version>20220320</version>
</dependency> 
```

## 3.不使用外部依赖

JDBC API 早于现代 Java 集合框架。因此，**我们不能使用类似于`for-`每次迭代和`Stream`的方法。**

相反，我们必须依赖迭代器。此外，我们需要从`ResultSet` 的元数据[中提取列名的数量和列表。](/web/20220627155255/https://www.baeldung.com/jdbc-database-metadata)

这导致了一个基本的循环，包括每行形成一个 JSON 对象，将对象添加到一个`List`，最后将这个`List`转换成一个`JSON`数组。所有这些功能都包含在`org.json`包中:

```
ResultSetMetaData md = resultSet.getMetaData();
int numCols = md.getColumnCount();
List<String> colNames = IntStream.range(0, numCols)
  .mapToObj(i -> {
      try {
          return md.getColumnName(i + 1);
      } catch (SQLException e) {
          e.printStackTrace();
          return "?";
      }
  })
  .collect(Collectors.toList());

JSONArray result = new JSONArray();
while (resultSet.next()) {
    JSONObject row = new JSONObject();
    colNames.forEach(cn -> {
        try {
            row.put(cn, resultSet.getObject(cn));
        } catch (JSONException | SQLException e) {
            e.printStackTrace();
        }
    });
    result.add(row);
}
```

这里，我们首先运行一个循环来提取每一列的名称。我们稍后将使用这些列名来形成最终的 JSON 对象。

在第二个循环中，我们遍历实际结果，并使用上一步中计算的列名将每个结果转换成一个 JSON 对象。然后我们将所有这些对象添加到一个 JSON 数组中。

我们已经将列名和列数的提取排除在循环之外。这有助于加快执行速度。

生成的 JSON 如下所示:

```
[
   {
      "Username":"doe1",
      "First name":"John",
      "Id":"7173",
      "Last name":"Doe"
   },
   {
      "Username":"smith3",
      "First name":"Dana",
      "Id":"3722",
      "Last name":"Smith"
   },
   {
      "Username":"john22",
      "First name":"John",
      "Id":"5490",
      "Last name":"Wang"
   }
]
```

## 4.使用 jOOQ 的默认设置

[jOOQ](/web/20220627155255/https://www.baeldung.com/jooq-intro) 框架(Java 面向对象查询)提供了一组方便的实用函数来处理 JDBC 和`ResultSet `对象。首先，我们需要将 jOOQ 依赖项添加到 POM 文件中:

```
<dependency>
    <groupId>org.jooq</groupId>
    <artifactId>jooq</artifactId>
    <version>3.11.11</version>
</dependency>
```

添加完依赖项后，我们实际上可以使用一个单行解决方案将`ResultSet`转换成 JSON 对象:

```
JSONObject result = new JSONObject(DSL.using(dbConnection)
  .fetch(resultSet)
  .formatJSON());
```

**产生的 JSON 元素是一个由两个名为`fields`和`records`的字段组成的对象，其中`fields`包含列的名称和类型，`records`包含实际数据。**这与之前的 JSON 对象略有不同，在我们的示例表中是这样的:

```
{
   "records":[
      [
         "doe1",
         "7173",
         "John",
         "Doe"
      ],
      [
         "smith3",
         "3722",
         "Dana",
         "Smith"
      ],
      [
         "john22",
         "5490",
         "John",
         "Wang"
      ]
   ],
   "fields":[
      {
         "schema":"PUBLIC",
         "name":"Username",
         "type":"VARCHAR",
         "table":"WORDS"
      },
      {
         "schema":"PUBLIC",
         "name":"Id",
         "type":"VARCHAR",
         "table":"WORDS"
      },
      {
         "schema":"PUBLIC",
         "name":"First name",
         "type":"VARCHAR",
         "table":"WORDS"
      },
      {
         "schema":"PUBLIC",
         "name":"Last name",
         "type":"VARCHAR",
         "table":"WORDS"
      }
   ]
}
```

## 5.通过定制设置使用 jOOQ

万一我们不喜欢 jOOQ 产生的 JSON 对象的默认结构，还有自定义的空间。

我们将通过实现`RecordMapper`接口来实现这一点。这个接口有一个`map()`方法，它接收一个`Record`作为输入，并返回任意类型的所需对象。

然后我们将`RecordMapper`作为 jOOQ 结果类的`map()`方法的输入:

```
List json = DSL.using(dbConnection)
  .fetch(resultSet)
  .map(new RecordMapper() {
      @Override
      public JSONObject map(Record r) {
          JSONObject obj = new JSONObject();
          colNames.forEach(cn -> obj.put(cn, r.get(cn)));
          return obj;
      }
  });
return new JSONArray(json); 
```

这里，我们从`map()` 方法返回了一个`JSONObject` 。

生成的 JSON 如下所示，类似于第 3 节:

```
[
   {
      "Username":"doe1",
      "First name":"John",
      "Id":"7173",
      "Last name":"Doe"
   },
   {
      "Username":"smith3",
      "First name":"Dana",
      "Id":"3722",
      "Last name":"Smith"
   },
   {
      "Username":"john22",
      "First name":"John",
      "Id":"5490",
      "Last name":"Wang"
   }
]
```

## 6.结论

在本文中，我们探索了将 JDBC `ResultSet`转换成 JSON 对象的三种不同方法。

每种方法都有自己的用途。例如，我们的选择取决于输出 JSON 对象所需的结构以及对依赖项大小的可能限制。

和往常一样，这些例子的源代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220627155255/https://github.com/eugenp/tutorials/tree/master/persistence-modules/core-java-persistence-2)