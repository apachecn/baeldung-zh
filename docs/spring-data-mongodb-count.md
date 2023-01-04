# 使用 Spring Data MongoDB 存储库计算文档数

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-data-mongodb-count>

## 1.概观

在本教程中，我们将看到使用 [Spring Data MongoDB](/web/20220904120154/https://www.baeldung.com/spring-data-mongodb-tutorial) 对集合中的文档进行计数的不同方法。我们将使用`MongoRepository`中所有可用的工具。

我们将使用注释、查询方法和来自`CrudRepository`的方法。此外，我们将构建一个简单的服务来聚合我们不同的用例。

## 2.用例设置

我们的用例由一个模型类、一个存储库和一个服务类组成。此外，我们将创建一个测试类来帮助我们确保一切按预期运行。

### 2.1.创建模型

我们将从创建模型类开始。它基于汽车的几个特性:

```java
@Document
public class Car {
    private String name;

    private String brand;

    public Car(String brand) {
        this.brand = brand;
    }

    // getters and setters
}
```

我们省略了 ID 属性，因为在我们的示例中不需要它。此外，我们添加了一个构造函数，它将一个`brand`属性作为参数，以使测试更容易。

### 2.2.定义存储库

让我们不使用任何方法来定义我们的存储库:

```java
public interface CarRepository extends MongoRepository<Car, String> {
}
```

我们正在考虑一个字符串 ID，尽管我们没有在模型中声明 ID 属性。这是因为 MongoDB 创建了一个默认的惟一 ID，如果我们愿意，我们仍然可以通过`findById()`访问它。

### 2.3.定义服务类别

我们的服务将以不同的方式利用 Spring 数据存储库接口。

**让我们参考我们的存储库来定义它:**

```java
@Service
public class CountCarService {

    @Autowired
    private CarRepository repo;
}
```

我们将在下一节构建这个类，包括例子。

### 2.4.准备测试

我们所有的测试都将在我们的服务类上运行。我们只需要一点设置，这样我们就不会以重复的代码结束:

```java
public class CountCarServiceIntegrationTest {
    @Autowired
    private CountCarService service;

    Car car1 = new Car("B-A");

    @Before
    public void init() {
        service.insertCar(car1);
        service.insertCar(new Car("B-B"));
    }
}
```

**我们将在每次测试之前运行这个模块，以简化我们的测试场景。**此外，我们在`init()`之外定义了`car1`,以便在以后的测试中可以访问它。

## 3.使用`CrudRepository`

**当使用扩展了`CrudRepository`的`MongoRepository`时，我们可以访问基本功能，包括一个`count()`方法。**

### 3.1.`count()`方法

因此，在我们的第一个 count 示例中，我们的存储库中没有任何方法，我们可以在服务中调用它:

```java
public long getCountWithCrudRepository() {
    return repo.count();
}
```

我们可以测试它:

```java
@Test
public void givenAllDocs_whenCrudRepositoryCount_thenCountEqualsSize() {
    List<Car> all = service.findCars();

    long count = service.getCountWithCrudRepository();

    assertEquals(count, all.size());
}
```

因此，我们确保`count()`输出与集合中所有文档的列表大小相同的数字。

最重要的是，我们必须记住，计数操作比列出所有文档更具成本效益。这是从性能和减少代码两方面考虑的。这对于小的集合来说不会有什么不同，但是对于大的集合来说，我们可能会以一个[内存不足错误](/web/20220904120154/https://www.baeldung.com/java-gc-overhead-limit-exceeded)结束。**简而言之，通过列出整个集合来统计文档不是一个好主意。**

### 3.2.使用`Example` 对象过滤

如果我们想对具有特定属性值的文档进行计数,`CrudRepository`也有帮助。**`count()`方法有一个重载版本，它接收一个[示例](/web/20220904120154/https://www.baeldung.com/spring-data-query-by-example)对象:**

```java
public long getCountWithExample(Car item) {
    return repo.count(Example.of(item));
}
```

因此，这简化了任务。现在我们只需用我们想要过滤的属性填充一个对象，Spring 会完成剩下的工作。让我们在测试中涵盖它:

```java
@Test
public void givenFilteredDocs_whenExampleCount_thenCountEqualsSize() {
    long all = service.findCars()
      .stream()
      .filter(car -> car.getBrand().equals(car1.getBrand()))
      .count();

    long count = service.getCountWithExample(car1);

    assertEquals(count, all);
}
```

## 4.使用@Query 注释

我们的下一个例子将基于`@Query`注释:

```java
@Query(value = "{}", count = true)
Long countWithAnnotation();
```

我们必须指定`value`属性，否则 Spring 将尝试从我们的方法名创建一个查询。**但是，由于我们想要统计所有文档，我们只需指定一个空查询。**

**然后，我们通过将`count`属性设置为`true`来指定该查询的结果应该是计数[投影](/web/20220904120154/https://www.baeldung.com/java-mongodb-aggregations)。**

让我们来测试一下:

```java
@Test
public void givenAllDocs_whenQueryAnnotationCount_thenCountEqualsSize() {
    List<Car> all = service.findCars();

    long count = service.getCountWithQueryAnnotation();

    assertEquals(count, all.size());
}
```

### 4.1.对属性进行筛选

**我们可以扩展我们的例子，用`brand`过滤。**让我们向我们的资源库添加一个新方法:

```java
@Query(value = "{brand: ?0}", count = true)
public long countBrand(String brand);
```

在我们的查询`value`中，我们指定了完整的 [MongoDB 风格查询](https://web.archive.org/web/20220904120154/https://www.mongodb.com/docs/manual/tutorial/query-documents/)。“`?0`”占位符代表我们方法的第一个参数，它将是我们的[查询](/web/20220904120154/https://www.baeldung.com/queries-in-spring-data-mongodb)的参数值。

MongoDB 查询有一个 JSON 结构，在这个结构中，我们指定字段名以及我们想要过滤的值。因此，当我们调用`countBrand(“A”)`时，查询被翻译成`{brand: “A”}`。这意味着我们将通过其`brand`属性的值为“A”的项目来过滤我们的集合。

## 5.编写派生查询方法

一个[派生的查询方法](https://web.archive.org/web/20220904120154/https://courses.baeldung.com/courses/1295711/lectures/30127898)是我们的存储库中不包含带有`value`的`@Query`注释的任何方法。Spring 按名称解析这些方法，因此我们不必编写查询。

**因为我们的`CrudRepository`中已经有了一个`count()`方法，所以让我们创建一个按特定品牌计数的示例:**

```java
Long countByBrand(String brand);
```

**该方法将统计所有`brand`属性与参数值匹配的文档。**

现在，让我们将它添加到我们的服务中:

```java
public long getCountBrandWithQueryMethod(String brand) {
    return repo.countByBrand(brand);
}
```

然后，我们通过将其与过滤的流计数操作进行比较来确保我们的方法行为正确:

```java
@Test
public void givenFilteredDocs_whenQueryMethodCountByBrand_thenCountEqualsSize() {
    String filter = "B-A";
    long all = service.findCars()
      .stream()
      .filter(car -> car.getBrand().equals(filter))
      .count();

    long count = service.getCountBrandWithQueryMethod(filter);

    assertEquals(count, all);
}
```

当我们只需要编写几个不同的查询时，这非常有用。但是，如果我们需要太多不同的计数查询，维护会变得很困难。

## 6.使用带有条件的动态计数查询

**当我们需要更健壮一点的东西时，我们可以将`Criteria` 与`Query`对象一起使用。**

但是，要运行一个`Query`，我们需要`MongoTemplate`。它在启动时被实例化，并在`mongoOperations`字段的`SimpleMongoRepository`中可用。

访问它的一种方法是扩展`SimpleMongoRepository`并创建一个定制的实现，而不是简单地扩展`MongoRepository`。但是，有一个更简单的方法。我们可以将它注入到我们的服务中:

```java
@Autowired
private MongoTemplate mongo;
```

然后我们可以创建新的计数方法，将`Query`传递给`MongoTemplate`中的`count()`方法:

```java
public long getCountBrandWithCriteria(String brand) {
    Query query = new Query();
    query.addCriteria(Criteria.where("brand")
      .is(brand));
    return mongo.count(query, Car.class);
}
```

当我们需要创建动态查询时，这种方法很有用。我们可以完全控制如何创建投影。

### 6.1.使用`Example` 对象过滤

**`Criteria`对象也允许我们传递一个`Example`对象:**

```java
public long getCountWithExampleCriteria(Car item) {
    Query query = new Query();
    query.addCriteria(Criteria.byExample(item));
    return mongo.count(query, Car.class);
}
```

这使得按属性过滤更容易，同时仍然允许动态部分。

## 7.结论

在本文中，我们看到了在 Spring Data MongoDB 中通过存储库方法使用计数投影的不同方式。

我们使用了现有的方法，也用不同的方法创造了新的方法。此外，我们通过比较计数方法和列出集合中的所有对象来创建测试。同样，我们知道了为什么像这样计算文档不是一个好主意。

此外，我们更深入一点，使用`MongoTemplate`来创建更多的动态计数查询。

和往常一样，源代码可以从 GitHub 上的[处获得。](https://web.archive.org/web/20220904120154/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-boot-persistence-mongodb-2)