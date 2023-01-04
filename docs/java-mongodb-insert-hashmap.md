# 如何用 Java 在 MongoDB 中插入一个 HashMap？

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-mongodb-insert-hashmap>

## 1.介绍

在这个快速教程中，我们将学习如何在 MongoDB 中使用 Java [`HashMap`](/web/20221107202807/https://www.baeldung.com/java-hashmap) 。 **MongoDB 有一个地图友好的 API，而 [Spring Data MongoDB](/web/20221107202807/https://www.baeldung.com/spring-data-mongodb-tutorial) 让使用地图或地图列表变得更加简单。**

## 2.设置我们的场景

Spring Data MongoDB 附带了`MongoTemplate`，它有许多重载版本的`insert()`，允许我们在集合中插入地图。 **MongoDB 表示一个 [JSON](/web/20221107202807/https://www.baeldung.com/java-json) 格式的文档。所以，我们可以用 Java 中的`Map<String, Object>`来复制它。**

**我们将使用`MongoTemplate `和一个简单的可重用地图来实现我们的用例。**让我们从创建地图参考和注入`MongoTemplate`开始:

```java
class MongoDbHashMapIntegrationTest {
    private static final Map<String, Object> MAP = new HashMap<>();

    @Autowired
    private MongoTemplate mongo;
}
```

然后，我们将用几个条目初始化我们的映射，每个条目都是不同的类型:

```java
@BeforeAll
static void init() {
    MAP.put("name", "Document A");
    MAP.put("number", 2);
    MAP.put("dynamic", true);
}
```

我们用 [@BeforeAll](/web/20221107202807/https://www.baeldung.com/junit-before-beforeclass-beforeeach-beforeall) 标记它，这样我们所有的测试都可以使用它。

## 3.直接插入单个`Map`

首先，我们调用`mongo.insert()`并选择一个要放入的集合:

```java
@Test
void whenUsingMap_thenInsertSucceeds() {
    Map<String, Object> saved = mongo.insert(MAP, "map-collection");

    assertNotNull(saved.get("_id"));
}
```

**不需要特殊的包装。**插入之后，我们的地图就变成了集合中的一个 JSON 文档。**最重要的是，我们可以检查 MongoDB 生成的`_id`属性的存在，确保它被正确处理。**

## 4.直接批量插入

我们还可以插入一个 [`Collection`](/web/20221107202807/https://www.baeldung.com/java-collections) 的`Map` s，每次插入都成为一个不同的文档。此外，为了确保我们不插入重复，我们将使用一个`[Set](/web/20221107202807/https://www.baeldung.com/java-set-vs-list).`

**让我们将之前创建的地图和一张新地图添加到我们的地图集中:**

```java
@Test
void whenMapSet_thenInsertSucceeds() {
    Set<Map<String, Object>> set = new HashSet<>();

    Map<String, Object> otherMap = new HashMap<>();
    otherMap.put("name", "Other Document");
    otherMap.put("number", 22);

    set.add(MAP);
    set.add(otherMap);

    Collection<Map<String, Object>> insert = mongo.insert(set, "map-set");

    assertEquals(2, insert.size());
}
```

结果，我们得到两个插入。这种方法有助于减少一次添加大量文档的开销。

## 5.从`Map `构造`Document`并插入

**`Document`类是[推荐的用 Java 处理 MongoDB 文档的](https://web.archive.org/web/20221107202807/https://www.mongodb.com/docs/drivers/java/sync/v4.3/fundamentals/data-formats/documents/#overview)方式。**实现了`Map`和`Bson`，方便工作。让我们使用接受映射的构造函数:

```java
@Test
void givenMap_whenDocumentConstructed_thenInsertSucceeds() {
    Document document = new Document(MAP);

    Document saved = mongo.insert(document, "doc-collection");

    assertNotNull(saved.get("_id"));
}
```

在内部，`Document`使用一个`LinkedHashMap`，保证插入的顺序。

## 6.从`Map `构造`BasicDBObject`并插入

虽然首选的是`Document`类，但是我们也可以从地图中构造一个`BasicDBObject`:

```java
@Test
void givenMap_whenBasicDbObjectConstructed_thenInsertSucceeds() {
    BasicDBObject dbObject = new BasicDBObject(MAP);

    BasicDBObject saved = mongo.insert(dbObject, "db-collection");

    assertNotNull(saved.get("_id"));
}
```

如果我们正在处理遗留代码，那么 **`BasicDBObject`仍然是有用的，因为`Document`类只在 MongoDB 驱动版本 3 以后才可用。**

## 7.从`Object`值流构建`Document`并插入

在我们最后的例子中，我们将从一个`Map`构建一个`Document`对象，其中每个键的值是一个`Object`值的`List`。因为我们知道值的格式，所以我们可以通过为每个值添加一个属性名来构建我们的`Document`。

让我们从构建我们的`input`地图开始:

```java
Map<String, List<Object>> input = new HashMap<>();
List<Object> listOne = new ArrayList<>();
listOne.add("Doc A");
listOne.add(1);

List<Object> listTwo = new ArrayList<>();
listTwo.add("Doc B");
listTwo.add(2);

input.put("a", listOne);
input.put("b", listTwo);
```

正如我们所看到的，没有属性名，只有值。所以，让我们[流](/web/20221107202807/https://www.baeldung.com/java-streams)我们的`input`的 [`entrySet()`](/web/20221107202807/https://www.baeldung.com/java-map-entries-methods) 并从中构建我们的`result`。为此，我们将把`input`中的每个条目收集到一个`HashSet`中。**然后，我们将在累加器函数中构建一个*文档*，将输入键作为`_id`属性。之后，我们将遍历条目值，将它们放在适当的属性名下。**最后，我们将把每个`Document`添加到我们的`result`中:

```java
Set<Document> result = input.entrySet()
  .stream()
  .collect(HashSet<Document>::new, 
    (set, entry) -> {
      Document document = new Document();

      document.put("_id", entry.getKey());
      Iterator<Object> iterator = entry.getValue()
        .iterator();
      document.put("name", iterator.next());
      document.put("number", iterator.next());

      set.add(document);
    }, 
    Set::addAll
  );
```

请注意，在这个例子中，我们不需要来自`collect()`的第三个参数。但那是因为只有[并行流](/web/20221107202807/https://www.baeldung.com/java-when-to-use-parallel-stream)使用合并器功能。

**最后，我们可以将`result`插入到 MongoDB:**

```java
mongo.insert(result, "custom-set");
```

例如，如果我们想将 CSV 值转换成 JSON，这种策略就很有用。

## 8.结论

在本文中，我们看到了使用一个`HashMap`和一列`HashMap`将文档插入 MongoDB 集合的不同方法。我们使用`MongoTemplate`来简化任务，并使用最常见的文档抽象:`BasicDBObject`和`Document`。

和往常一样，源代码可以从 GitHub 上的[处获得。](https://web.archive.org/web/20221107202807/https://github.com/eugenp/tutorials/tree/master//persistence-modules/spring-boot-persistence-mongodb-3)