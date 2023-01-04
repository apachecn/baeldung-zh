# 持久化查询模型

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/axon-persisting-query-model>

## 1.概观

Axon framework 帮助我们构建事件驱动的微服务系统。在[Axon 框架指南](/web/20221128161122/https://www.baeldung.com/axon-cqrs-event-sourcing)中，我们通过一个简单的 axon [Spring Boot](/web/20221128161122/https://www.baeldung.com/spring-boot-start) 应用了解了 Axon，其中包括构建一个示例`Order`模型供我们更新和查询。在 Axon 框架的[调度查询中，我们添加了所有支持的查询。](/web/20221128161122/https://www.baeldung.com/axon-query-dispatching)

本文将**关注持久化 Axon 框架的查询模型**。我们将介绍如何使用 MongoDB 存储投影，以及测试面临的挑战，以及如何使流与查询模型保持同步。

## 2.持久性考虑

为了创建一个使用数据库持久化查询模型的处理程序，我们实现了`OrdersEventHandler`接口。在生产环境中，**我们不希望每次都从零开始构建查询模型**。有了 Axon 框架，我们可以选择如何持久化模型，选择什么取决于涉及的数据。如果我们想要免费的文本搜索，我们可能想要使用[弹性搜索](/web/20221128161122/https://www.baeldung.com/elasticsearch-java)。当我们有非结构化数据时，我们可能希望使用 [MongoDB](/web/20221128161122/https://www.baeldung.com/spring-data-mongodb-tutorial) 。当实体之间有很多关系时，我们可能希望使用像 [Neo4J](/web/20221128161122/https://www.baeldung.com/java-neo4j) 这样的图形数据库。

### 2.1.令牌存储

当通过遍历事件来构建查询模型时，Axon 使用一个`TokenStore`来跟踪。理想情况下，令牌存储与查询模型保存在同一个数据库中，以确保一致性。使用持久令牌存储还将确保我们可以运行多个实例，其中每个实例只需要处理部分事件。分割成几个实例与[段](https://web.archive.org/web/20221128161122/https://docs.axoniq.io/reference-guide/axon-server/performance/tuning-event-processing)一起工作，其中一个实例可以要求处理所有或部分段。如果我们使用 [JPA](/web/20221128161122/https://www.baeldung.com/the-persistence-layer-with-spring-data-jpa) 或 [JDBC](/web/20221128161122/https://www.baeldung.com/spring-jdbc-jdbctemplate) 进行持久化，那么使用`[JpaTokenStore](https://web.archive.org/web/20221128161122/https://apidocs.axoniq.io/4.6/org/axonframework/eventhandling/tokenstore/jpa/JpaTokenStore.html)` 或 [JdbcTokenStore](https://web.archive.org/web/20221128161122/https://apidocs.axoniq.io/4.6/org/axonframework/eventhandling/tokenstore/jdbc/JdbcTokenStore.html) 。这两种令牌存储实现都可以在 Axon 框架中使用，而不需要扩展。

### 2.2.构建查询模型

启动时，[流式事件处理器](https://web.archive.org/web/20221128161122/https://docs.axoniq.io/reference-guide/axon-framework/events/event-processors/streaming)将开始从事件存储中读取事件。使用持久`TokenStore`，处理器从先前离开的地方开始。否则，默认情况下，它将从头开始。**对于每一个事件，处理器都会调用事件处理程序带注释的方法。**

让我们进一步构建订单应用程序，并允许以多种方式创建和更新订单。通过更新内存模型在`InMemoryOrdersEventHandler` 中处理`ProductAddedEvent`:

```
@EventHandler
public void on(ProductAddedEvent event) {
    orders.computeIfPresent(event.getOrderId(), (orderId, order) -> {
      order.addProduct(event.getProductId());
      return order;
    });
}
```

这里，将使用`addProduct`函数更新内存映射中的顺序。我们可以将数据存储在数据库中，而不是内存模型。

## 3.Mongo 扩展

让我们使用 MongoDB 来持久化我们的查询模型。**我们使用 [Axon framework mongo 扩展](https://web.archive.org/web/20221128161122/https://github.com/AxonFramework/extension-mongo)在 mongo 中持久化令牌存储。**因为我们已经添加了`[axon-bom](https://web.archive.org/web/20221128161122/https://search.maven.org/search?q=a:axon-bom)`，所以在向我们的`pom.xml`添加扩展时，我们不需要指定版本:

```
<dependency>
    <groupId>org.axonframework.extensions.mongo</groupId>
    <artifactId>axon-mongo</artifactId>
</dependency>
```

### 3.1.令牌存储

有了依赖关系，我们可以**配置 Axon 来使用`MongoTokenStore`** :

```
@Bean
public TokenStore getTokenStore(MongoClient client, Serializer serializer){
    return MongoTokenStore.builder()
      .mongoTemplate(
        DefaultMongoTemplate.builder()
          .mongoDatabase(client)
          .build()
      )
      .serializer(serializer)
      .build();
}
```

### 3.2.事件句柄类

名为`mongo`的 [Spring Profile](/web/20221128161122/https://www.baeldung.com/spring-profiles) 支持在事件处理程序的实现之间切换。当`mongo`概要文件激活时，将使用`MongoOrdersEventHandler`，以及令牌存储配置。这使得事件处理程序类:

```
@Service
@ProcessingGroup("orders")
@Profile("mongo")
public class MongoOrdersEventHandler implements OrdersEventHandler {
    // all methods regarding updating an querying the projection
}
```

同时，我们在`InMemoryOrdersEventHandler`中增加了`@Profile("!mongo")`，这样就不会同时有两个活动。 [Spring profiles](/web/20221128161122/https://www.baeldung.com/spring-profiles) 是有条件地启用组件的一种很好的方式。

我们将在构造函数中使用依赖注入来获得`MongoClient`和`QueryUpdateEmitter.` ，我们使用`MongoClient`来创建 MongoCollection 和索引。我们注入`QueryUpdateEmitter`来启用[订阅查询](/web/20221128161122/https://www.baeldung.com/axon-query-dispatching#subscription-queries):

```
public MongoOrdersEventHandler(MongoClient client, QueryUpdateEmitter emitter) {
    orders = client
      .getDatabase(AXON_FRAMEWORK_DATABASE_NAME)
      .getCollection(ORDER_COLLECTION_NAME);
    orders.createIndex(Indexes.ascending(ORDER_ID_PROPERTY_NAME),
      new IndexOptions().unique(true));
    this.emitter = emitter;
}
```

请注意，我们将订单 id 设置为唯一的。这样，我们可以确保不会有两个具有相同订单 id 的文档。

`MongoOrdersEventHandler` 使用`orders` mongo 集合来处理查询。我们需要使用`documentToOrder()`方法将 Mongo 文档映射到订单:

```
@QueryHandler
public List<Order> handle(FindAllOrderedProductsQuery query) {
    List<Order> orderList = new ArrayList<>();
    orders
      .find()
      .forEach(d -> orderList.add(documentToOrder(d)));
    return orderList;
}
```

### 3.3.复杂的查询

为了能够处理`TotalProductsShippedQuery,`,我们添加了一个 **`shippedProductFilter`,它过滤出已经发货并有产品的订单:**

```
private Bson shippedProductFilter(String productId){
    return and(
      eq(ORDER_STATUS_PROPERTY_NAME, OrderStatus.SHIPPED.toString()),
      exists(String.format(PRODUCTS_PROPERTY_NAME + ".%s", productId))
    );
}
```

然后，在查询处理程序中使用该过滤器提取和添加产品计数:

```
@QueryHandler
public Integer handle(TotalProductsShippedQuery query) {
    AtomicInteger result = new AtomicInteger();
    orders
      .find(shippedProductFilter(query.getProductId()))
      .map(d -> d.get(PRODUCTS_PROPERTY_NAME, Document.class))
      .map(d -> d.getInteger(query.getProductId(), 0))
      .forEach(result::addAndGet);
    return result.get();
}
```

该查询将获取所有发货的产品，并检索这些文档中的所有产品。然后，它将对查询的特定产品进行计数，并返回总数。

## 4.测试持久查询模型

使用持久模型进行测试会使事情变得更加困难，因为我们希望每个测试都有一个隔离的环境。

### 4.1.单元测试

对于`MongoOrdersEventHandler,` **，我们需要确保删除集合，这样我们就不会保留之前测试的状态**。我们通过实现`getHandler()`方法来实现这一点:

```
@Override
protected OrdersEventHandler getHandler() {
    mongoClient.getDatabase("axonframework").drop();
    return new MongoOrdersEventHandler(mongoClient, emitter);
}
```

使用`@BeforeEach`注释方法，我们可以确保每个测试都是全新开始的。在这种情况下，我们使用一个[嵌入式 Mongo](/web/20221128161122/https://www.baeldung.com/spring-boot-embedded-mongodb) 进行测试。另一种选择是使用[测试容器](/web/20221128161122/https://www.baeldung.com/spring-boot-testcontainers-integration-test)。在这方面，测试查询模型与其他需要数据库的应用程序测试没有什么不同。

### 4.2.集成测试

对于集成测试，我们使用类似的方法。然而，由于集成测试使用了`OrdersEventHandler`接口，**我们依赖于实现的`reset()`方法**。

`reset()`方法的实现是:

```
@Override
public void reset(List<Order> orderList) {
    orders.deleteMany(new Document());
    orderList.forEach(o -> orders.insertOne(orderToDocument(o)));
}
```

`reset()`方法确保只有列表中的订单是集合的一部分。该方法在`OrderQueryServiceIntegrationTest`中每次测试前执行:

```
@BeforeEach
void setUp() {
    orderId = UUID.randomUUID().toString();
    Order order = new Order(orderId);
    handler.reset(Collections.singletonList(order));
}
```

至于测试查询，我们至少需要一个订单。由于已经存储了一个订单，这使得测试本身更加容易。

## 5.结论

在本文中，我们展示了如何持久化查询模型。我们学习了如何使用 MongoDB 查询和测试模型。

和往常一样，本文中使用的完整代码可以从 GitHub 上的[处获得。](https://web.archive.org/web/20221128161122/https://github.com/eugenp/tutorials/tree/master/axon)