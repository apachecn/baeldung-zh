# 在 Spring Data MongoDB 中只返回查询的特定字段

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/mongodb-return-specific-fields>

## 1.概观

使用 [Spring Data MongoDB](/web/20220617075805/https://www.baeldung.com/spring-data-mongodb-tutorial) 时，我们可能需要限制从数据库对象映射的属性。通常，我们可能需要这样做，例如，出于安全原因，以避免暴露存储在服务器上的敏感信息。或者，例如，我们可能需要过滤掉 web 应用程序中显示的部分数据。

在这个简短的教程中，我们将看到 MongoDB 如何应用字段限制。

## 2.使用投影的 MongoDB 字段限制

**MongoDB 使用[投影](https://web.archive.org/web/20220617075805/https://www.mongodb.com/docs/manual/tutorial/project-fields-from-query-results/)来指定或限制从查询**返回的字段。然而，如果我们使用 Spring 数据，我们希望用 [`MongoTemplate`](https://web.archive.org/web/20220617075805/https://docs.spring.io/spring-data/mongodb/docs/current/api/org/springframework/data/mongodb/core/MongoTemplate.html) 或 [`MongoRepository`](https://web.archive.org/web/20220617075805/https://docs.spring.io/spring-data/mongodb/docs/current/api/org/springframework/data/mongodb/repository/MongoRepository.html) 来应用它。

因此，我们想要为`MongoTemplate`和`MongoRepository`创建测试用例，在这里我们可以应用字段限制。

## 3.实施投影

### 3.1.建立实体

首先，让我们创建一个`Inventory`类:

```
@Document(collection = "inventory")
public class Inventory {

    @Id
    private String id;
    private String status;
    private Size size;
    private InStock inStock;

    // standard getters and setters    
}
```

### 3.2.设置存储库

然后，为了测试`MongoRepository`，我们创建了一个`InventoryRepository`。我们还将在`@Query`中使用一个`where`条件。例如，我们想过滤库存状态:

```
public interface InventoryRepository extends MongoRepository<Inventory, String> {

    @Query(value = "{ 'status' : ?0 }", fields = "{ 'item' : 1, 'status' : 1 }")
    List<Inventory> findByStatusIncludeItemAndStatusFields(String status);

    @Query(value = "{ 'status' : ?0 }", fields = "{ 'item' : 1, 'status' : 1, '_id' : 0 }")
    List<Inventory> findByStatusIncludeItemAndStatusExcludeIdFields(String status);

    @Query(value = "{ 'status' : ?0 }", fields = "{ 'status' : 0, 'inStock' : 0 }")
    List<Inventory> findByStatusIncludeAllButStatusAndStockFields(String status);

    @Query(value = "{ 'status' : ?0 }", fields = "{ 'item' : 1, 'status' : 1, 'size.uom': 1 }")
    List<Inventory> findByStatusIncludeEmbeddedFields(String status);

    @Query(value = "{ 'status' : ?0 }", fields = "{ 'size.uom': 0 }")
    List<Inventory> findByStatusExcludeEmbeddedFields(String status);

    @Query(value = "{ 'status' : ?0 }", fields = "{ 'item' : 1, 'status' : 1, 'inStock.quantity': 1 }")
    List<Inventory> findByStatusIncludeEmbeddedFieldsInArray(String status);

    @Query(value = "{ 'status' : ?0 }", fields = "{ 'item' : 1, 'status' : 1, 'inStock': { $slice: -1 } }")
    List<Inventory> findByStatusIncludeEmbeddedFieldsLastElementInArray(String status);

}
```

### 3.3.添加 Maven 依赖项

**我们还将使用[嵌入式 MongoDB](/web/20220617075805/https://www.baeldung.com/spring-boot-embedded-mongodb) 。** L et 将 [`spring-data-mongodb`](https://web.archive.org/web/20220617075805/https://search.maven.org/artifact/org.springframework.data/spring-data-mongodb) 和 [`de.flapdoodle.embed.mongo`](https://web.archive.org/web/20220617075805/https://search.maven.org/artifact/de.flapdoodle.embed/de.flapdoodle.embed.mongo) 依赖关系添加到我们的`pom.xml`文件中:

```
<dependency>
    <groupId>org.springframework.data</groupId>
    <artifactId>spring-data-mongodb</artifactId>
    <version>3.0.3.RELEASE</version>
</dependency>
<dependency>
    <groupId>de.flapdoodle.embed</groupId>
    <artifactId>de.flapdoodle.embed.mongo</artifactId>
    <version>3.2.6</version>
    <scope>test</scope>
</dependency>
```

## 4.使用`MongoRepository`和`MongoTemplate`进行测试

**对于`MongoRepository`，我们将看到使用`@Query`和应用[字段限制](https://web.archive.org/web/20220617075805/https://docs.spring.io/spring-data/mongodb/docs/3.3.1/reference/html/#mongodb.repositories.queries.json-based)、** **的示例，而对于、我们将使用**[**`Query`**](https://web.archive.org/web/20220617075805/https://docs.spring.io/spring-data/mongodb/docs/3.3.1/reference/html/#mongo.query)**类。**

我们将尝试涵盖所有不同的包含和排除组合。**特别是，我们将看到如何使用`slice`属性**来限制嵌入字段，或者更有趣的是，限制数组。

对于每一个测试，我们会先添加 `MongoRepository` 的例子，然后是 `MongoTemplate` 的例子。

### 4.1.仅包含字段

让我们从包含一些字段开始。所有被排除的将是`null`。投影默认添加了 _`id`:

```
List<Inventory> inventoryList = inventoryRepository.findByStatusIncludeItemAndStatusFields("A");

inventoryList.forEach(i -> {
  assertNotNull(i.getId());
  assertNotNull(i.getItem());
  assertNotNull(i.getStatus());
  assertNull(i.getSize());
  assertNull(i.getInStock());
});
```

现在，让我们来看看`MongoTemplate`版本:

```
Query query = new Query();
 query.fields()
   .include("item")
   .include("status"); 
```

### 4.2.包括和排除字段

这一次，我们将看到明确包括一些字段但排除其他字段的示例—在这种情况下，我们将排除 _ `id`字段:

```
List<Inventory> inventoryList = inventoryRepository.findByStatusIncludeItemAndStatusExcludeIdFields("A");

inventoryList.forEach(i -> {
   assertNotNull(i.getItem());
   assertNotNull(i.getStatus());
   assertNull(i.getId());
   assertNull(i.getSize());
   assertNull(i.getInStock());
});
```

使用`MongoTemplate`的等效查询将是:

```
Query query = new Query();
query.fields()
  .include("item")
  .include("status")
  .exclude("_id"); 
```

### 4.3.仅排除字段

让我们继续排除一些字段。所有其他字段将为非空:

```
List<Inventory> inventoryList = inventoryRepository.findByStatusIncludeAllButStatusAndStockFields("A");

inventoryList.forEach(i -> {
  assertNotNull(i.getItem());
  assertNotNull(i.getId());
  assertNotNull(i.getSize());
  assertNull(i.getInStock());
  assertNull(i.getStatus());
});
```

让我们来看看`MongoTemplate`版本:

```
Query query = new Query();
query.fields()
  .exclude("status")
  .exclude("inStock"); 
```

### 4.4.包括嵌入字段

同样，包含嵌入字段会将它们添加到我们的结果中:

```
List<Inventory> inventoryList = inventoryRepository.findByStatusIncludeEmbeddedFields("A");

inventoryList.forEach(i -> {
  assertNotNull(i.getItem());
  assertNotNull(i.getStatus());
  assertNotNull(i.getId());
  assertNotNull(i.getSize());
  assertNotNull(i.getSize().getUom());
  assertNull(i.getSize().getHeight());
  assertNull(i.getSize().getWidth());
  assertNull(i.getInStock());
});
```

让我们看看如何用`MongoTemplate`做同样的事情:

```
Query query = new Query();
query.fields()
  .include("item")
  .include("status")
  .include("size.uom"); 
```

### 4.5.排除嵌入字段

同样，**排除嵌入的字段使它们不在我们的结果中，但是，它会添加剩余的嵌入字段**:

```
List<Inventory> inventoryList = inventoryRepository.findByStatusExcludeEmbeddedFields("A");

inventoryList.forEach(i -> {
  assertNotNull(i.getItem());
  assertNotNull(i.getStatus());
  assertNotNull(i.getId());
  assertNotNull(i.getSize());
  assertNull(i.getSize().getUom());
  assertNotNull(i.getSize().getHeight());
  assertNotNull(i.getSize().getWidth());
  assertNotNull(i.getInStock());
});
```

我们来看看`MongoTemplate`版本:

```
Query query = new Query();
query.fields()
  .exclude("size.uom"); 
```

### 4.6.在数组中包含嵌入字段

与其他字段类似，我们也可以添加数组字段的投影:

```
List<Inventory> inventoryList = inventoryRepository.findByStatusIncludeEmbeddedFieldsInArray("A");

inventoryList.forEach(i -> {
  assertNotNull(i.getItem());
  assertNotNull(i.getStatus());
  assertNotNull(i.getId());
  assertNotNull(i.getInStock());
  i.getInStock()
    .forEach(stock -> {
      assertNull(stock.getWareHouse());
      assertNotNull(stock.getQuantity());
     });
  assertNull(i.getSize());
});
```

让我们使用`MongoTemplate`来实现同样的功能:

```
Query query = new Query();
query.fields()
  .include("item")
  .include("status")
  .include("inStock.quantity"); 
```

### 4.7.使用`slice`在数组中包含嵌入字段

MongoDB 可以使用 JavaScript 函数来限制数组的结果——例如，使用`slice`只获取数组中的最后一个元素:

```
List<Inventory> inventoryList = inventoryRepository.findByStatusIncludeEmbeddedFieldsLastElementInArray("A");

inventoryList.forEach(i -> {
  assertNotNull(i.getItem());
  assertNotNull(i.getStatus());
  assertNotNull(i.getId());
  assertNotNull(i.getInStock());
  assertEquals(1, i.getInStock().size());
  assertNull(i.getSize());
}); 
```

让我们使用`MongoTemplate`执行同样的查询:

```
Query query = new Query();
query.fields()
  .include("item")
  .include("status")
  .slice("inStock", -1); 
```

## 5.结论

在本文中，我们研究了 Spring 数据 MongoDB 中的投影。

我们已经看到了使用 `fields`、的例子，既有对 `MongoRepository` 接口和 `@Query` 的注释，也有对 `MongoTemplate` 和 `Query` 类的注释。

和往常一样，这些例子的代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220617075805/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-data-mongodb-2)