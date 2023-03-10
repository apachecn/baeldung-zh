# Eclipse JNoSQL 指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/eclipse-jnosql>

## 1。概述

Eclipse [JNoSQL](https://web.archive.org/web/20221221013845/https://projects.eclipse.org/projects/technology.jnosql) 是一组 API 和实现，**简化了 Java 应用程序与 NoSQL 数据库的交互**。

在本文中，我们将学习如何设置和配置 JNoSQL 来与 NoSQL 数据库交互。我们将与通信和映射层一起工作。

## 2。Eclipse JNoSQL 通信层

从技术上来说，通信层由两个模块组成: **Diana API 和一个驱动程序。**

虽然 API 定义了 NoSQL 数据库类型的抽象，但驱动程序为大多数已知的数据库提供了实现。

我们可以将它与关系数据库中的 JDBC API 和 JDBC 驱动程序进行比较。

### 2.1。Eclipse JNoSQL Diana API

简单地说，NoSQL 数据库有四种基本类型:键值、列、文档和图形。

Eclipse JNoSQL Diana API 定义了三个模块:

1.  戴安娜-键值
2.  戴安娜专栏
3.  戴安娜-文件

API 没有覆盖 NoSQL 图形类型，因为它已经被 [Apache ThinkerPop](https://web.archive.org/web/20221221013845/https://tinkerpop.apache.org/) 覆盖了。

该 API 基于核心模块 diana-core，并定义了常见概念的抽象，如配置、工厂、管理器、实体和值。

为了使用 API，我们需要提供相应模块对我们的 NoSQL 数据库类型的依赖。

因此，对于面向文档的数据库，我们需要`diana-document`依赖关系:

```java
<dependency>
    <groupId>org.jnosql.diana</groupId>
    <artifactId>diana-document</artifactId>
    <version>0.0.6</version>
</dependency>
```

类似地，如果工作中的 NoSQL 数据库是面向键值的，我们应该使用`diana-key-value`模块:

```java
<dependency>
    <groupId>org.jnosql.diana</groupId>
    <artifactId>diana-key-value</artifactId>
    <version>0.0.6</version>
</dependency>
```

最后是`diana-column`模块，如果它是面向列的:

```java
<dependency>
    <groupId>org.jnosql.diana</groupId>
    <artifactId>diana-column</artifactId>
    <version>0.0.6</version>
</dependency>
```

最新版本可以在 [Maven Central](https://web.archive.org/web/20221221013845/https://search.maven.org/classic/#search%7Cga%7C1%7Corg.jnosql.diana) 上找到。

### 2.2.Eclipse JNoSQL Diana 驱动程序

**驱动程序是最常见的 NoSQL 数据库的一组 API 实现**。

每个 NoSQL 数据库都有一个实现。**如果数据库是多模型的，驱动程序应该实现所有支持的 API**。

例如，`couchbase-driver`同时实现了`diana-document`和`diana-key-value`，因为 [Couchbase](https://web.archive.org/web/20221221013845/https://www.couchbase.com/) 同时面向文档和键值。

与关系数据库不同，在关系数据库中，驱动程序通常由数据库供应商提供，在这里**驱动程序由 Eclipse JNoSQL** 提供。在大多数情况下，这个驱动程序是官方供应商库的包装。

要开始使用驱动程序，我们应该包括 API 和所选 NoSQL 数据库的相应实现。

例如，对于 MongoDB，我们需要包含以下依赖项:

```java
<dependency>
    <groupId>org.jnosql.diana</groupId>
    <artifactId>diana-document</artifactId>
    <version>0.0.6</version>
</dependency>
<dependency>
    <groupId>org.jnosql.diana</groupId>
    <artifactId>mongodb-driver</artifactId>
    <version>0.0.6</version>
</dependency>
```

与驱动程序一起工作的过程很简单。

首先，我们需要一个`Configuration` bean。通过从类路径或硬编码值中读取配置文件，`Configuration` 能够创建一个`Factory.`,然后我们使用它来创建一个`Manager.`

**最后，`Manager`负责将`Entity`推送到 NoSQL 数据库**，以及从其中检索`Entity`。

在接下来的小节中，我们将为每种 NoSQL 数据库类型解释这个过程。

### 2.3。使用面向文档的数据库

在本例中，**我们将使用嵌入式 MongoDB** ,因为它很容易上手，不需要安装。它是面向文档的，下面的说明适用于任何其他面向文档的 NoSQL 数据库。

在最开始，我们应该提供应用程序所需的所有必要设置，以便以最基本的形式正确地与数据库`.`交互，我们应该提供 MongoDB 的运行实例的`host`和`port`。

我们可以在位于类路径的`mongodb-driver.properties`中提供这些设置:

```java
#Define Host and Port
mongodb-server-host-1=localhost:27017
```

或者作为硬编码的值:

```java
Map<String, Object> map = new HashMap<>();
map.put("mongodb-server-host-1", "localhost:27017");
```

接下来，我们为文档类型创建`Configuration` bean:

```java
DocumentConfiguration configuration = new MongoDBDocumentConfiguration();
```

从这个`Configuration` bean，我们能够创建一个`ManagerFactory`:

```java
DocumentCollectionManagerFactory managerFactory = configuration.get();
```

隐式地，`Configuration` bean 的`get()`方法使用属性文件中的设置。我们还可以从硬编码的值中获得这个工厂:

```java
DocumentCollectionManagerFactory managerFactory 
  = configuration.get(Settings.of(map));
```

`ManagerFactory`有一个简单的方法`get(),`，它将数据库名称作为参数，并创建`Manager`:

```java
DocumentCollectionManager manager = managerFactory.get("my-db");
```

最后，我们准备好了。`Manager `提供了通过`DocumentEntity.`与底层 NoSQL 数据库交互的所有必要方法

例如，我们可以插入一个文档:

```java
DocumentEntity documentEntity = DocumentEntity.of("books");
documentEntity.add(Document.of("_id", "100"));
documentEntity.add(Document.of("name", "JNoSQL in Action"));
documentEntity.add(Document.of("pages", "620"));
DocumentEntity saved = manager.insert(documentEntity);
```

我们还可以搜索文档:

```java
DocumentQuery query = select().from("books").where("_id").eq(100).build();
List<DocumentEntity> entities = manager.select(query);
```

同样，我们可以更新现有的文档:

```java
saved.add(Document.of("author", "baeldung"));
DocumentEntity updated = manager.update(saved);
```

最后，我们可以删除存储的文档:

```java
DocumentDeleteQuery deleteQuery = delete().from("books").where("_id").eq("100").build();
manager.delete(deleteQuery);
```

要运行这个示例，我们只需要访问`jnosql-diana`模块并运行`DocumentApp`应用程序。

我们应该会在控制台中看到输出:

```java
DefaultDocumentEntity{documents={pages=620, name=JNoSQL in Action, _id=100}, name='books'}
DefaultDocumentEntity{documents={pages=620, author=baeldung, name=JNoSQL in Action, _id=100}, name='books'}
[]
```

### 2.4。使用面向列的数据库

在本节中，**我们将使用 Cassandra 数据库**的嵌入式版本，因此不需要安装。

使用面向列的数据库的过程非常相似。首先，我们将 Cassandra 驱动程序和列 API 添加到 pom:

```java
<dependency>
    <groupId>org.jnosql.diana</groupId>
    <artifactId>diana-column</artifactId>
    <version>0.0.6</version>
</dependency>
<dependency>
    <groupId>org.jnosql.diana</groupId>
    <artifactId>cassandra-driver</artifactId>
    <version>0.0.6</version>
</dependency>
```

接下来，我们需要在类路径上的配置文件`diana-cassandra.properties, `中指定配置设置。或者，我们也可以使用硬编码的配置值。

然后，用类似的方法，我们将创建一个`ColumnFamilyManager`并开始操作`ColumnEntity:`

```java
ColumnConfiguration configuration = new CassandraConfiguration();
ColumnFamilyManagerFactory managerFactory = configuration.get();
ColumnFamilyManager entityManager = managerFactory.get("my-keySpace");
```

因此，要创建一个新实体，让我们调用`insert()`方法:

```java
ColumnEntity columnEntity = ColumnEntity.of("books");
Column key = Columns.of("id", 10L);
Column name = Columns.of("name", "JNoSQL in Action");
columnEntity.add(key);
columnEntity.add(name);
ColumnEntity saved = entityManager.insert(columnEntity);
```

要运行示例并在控制台中查看输出，请运行`ColumnFamilyApp`应用程序。

### 2.5。使用面向键值的数据库

在本节中，我们将使用 Hazelcast。Hazelcast 是一个面向键值的 NoSQL 数据库。关于黑兹尔卡斯特数据库的更多信息，你可以查看这个[链接](/web/20221221013845/https://www.baeldung.com/java-hazelcast)。

使用面向键值的类型的过程也是类似的。我们从将这些依赖项添加到 pom 开始:

```java
<dependency>
    <groupId>org.jnosql.diana</groupId>
    <artifactId>diana-key-value</artifactId>
    <version>0.0.6</version>
</dependency>
<dependency>
    <groupId>org.jnosql.diana</groupId>
    <artifactId>hazelcast-driver</artifactId>
    <version>0.0.6</version>
</dependency>
```

然后我们需要提供配置设置。接下来，我们可以获得一个`BucketManager`，然后操纵`KeyValueEntity:`

```java
KeyValueConfiguration configuration = new HazelcastKeyValueConfiguration();
BucketManagerFactory managerFactory = configuration.get();
BucketManager entityManager = managerFactory.getBucketManager("books"); 
```

假设我们想要保存下面的`Book`模型:

```java
public class Book implements Serializable {

    private String isbn;
    private String name;
    private String author;
    private int pages;

    // standard constructor
    // standard getters and setters
}
```

所以我们创建一个`Book`实例，然后通过调用`put()`方法保存它；

```java
Book book = new Book(
  "12345", "JNoSQL in Action", 
  "baeldung", 420);
KeyValueEntity keyValueEntity = KeyValueEntity.of(
  book.getIsbn(), book);
entityManager.put(keyValueEntity);
```

然后检索保存的`Book`实例:

```java
Optional<Value> optionalValue = manager.get("12345");
Value value = optionalValue.get(); // or any other adequate Optional handling
Book savedBook = value.get(Book.class);
```

要运行示例并在控制台中查看输出，请运行`KeyValueApp`应用程序。

## 3。Eclipse JNoSQL 映射层

映射层，Artemis API，是一组帮助将 java 注释对象映射到 NoSQL 数据库的 API。它基于 Diana API 和 CDI(上下文和依赖注入)。

我们可以把这个 API 看作 JPA 或 ORM，就像 NoSQL 世界一样。该层还为每种 NoSQL 类型提供了一个 API，并为通用特性提供了一个核心 API。

在本节中，我们将使用 MongoDB 面向文档的数据库。

### 3.1。所需的依赖关系

为了在应用程序中启用 Artemis，我们需要添加`[artemis-configuration](https://web.archive.org/web/20221221013845/https://search.maven.org/classic/#search%7Cga%7C1%7Cartemis-configuration)`依赖项。因为 MongoDB 是面向文档的，所以还需要依赖关系 [artemis-document](https://web.archive.org/web/20221221013845/https://search.maven.org/classic/#search%7Cga%7C1%7Cartemis-document) 。

对于其他类型的 NoSQL 数据库，我们将使用`artemis-column, artemis-key-value`和`artemis-graph`。

MongoDB 还需要 Diana 驱动程序:

```java
<dependency>
    <groupId>org.jnosql.artemis</groupId>
    <artifactId>artemis-configuration</artifactId>
    <version>0.0.6</version>
</dependency>
<dependency>
    <groupId>org.jnosql.artemis</groupId>
    <artifactId>artemis-document</artifactId>
    <version>0.0.6</version>
</dependency>
<dependency>
    <groupId>org.jnosql.diana</groupId>
    <artifactId>mongodb-driver</artifactId>
    <version>0.0.6</version>
</dependency>
```

Artemis 基于 CDI，因此我们还需要提供这种 Maven 依赖性:

```java
<dependency>
    <groupId>javax</groupId>
    <artifactId>javaee-web-api</artifactId>
    <version>8.0</version>
    <scope>provided</scope>
</dependency>
```

### 3.2。文档配置文件

配置是给定数据库的一组属性，让我们在代码之外提供设置。默认情况下，我们需要在 META-INF 资源下提供`jnosql.json`文件。

这是配置文件的一个示例:

```java
[
    {
        "description": "The mongodb document configuration",
        "name": "document",
        "provider": "org.jnosql.diana.mongodb.document.MongoDBDocumentConfiguration",
        "settings": {
            "mongodb-server-host-1":"localhost:27019"
        }
    }
]
```

我们需要通过在我们的`ConfigurationUnit` `.`中设置`name`属性来指定上面的配置名称。如果配置在不同的文件中，可以使用`fileName`属性来指定。

给定这个配置，我们创建一个工厂:

```java
@Inject
@ConfigurationUnit(name = "document")
private DocumentCollectionManagerFactory<MongoDBDocumentCollectionManager> managerFactory;
```

从这个工厂，我们可以创造一个`DocumentCollectionManager`:

```java
@Produces
public MongoDBDocumentCollectionManager getEntityManager() {
    return managerFactory.get("todos");
}
```

`DocumentCollectionManager`是一个支持 CDI 的 bean，它在`Template`和`Repository.`中都有使用

### 3.3。映射

映射是一个注释驱动的过程，通过这个过程`Entity`模型被转换成 Diana `EntityValue.`

让我们从定义一个`Todo`模型开始:

```java
@Entity
public class Todo implements Serializable {

    @Id("id")
    public long id;

    @Column
    public String name;

    @Column
    public String description;

    // standard constructor
    // standard getters and setters
}
```

如上所示，我们有基本的映射注释:`@Entity, @Id,`和`@Column.`

现在要操作这个模型，我们需要一个`Template`类或者一个`Repository`接口。

### 3.4。使用模板

**模板是实体模型和 Diana API** 之间的 **桥梁。对于面向文档的数据库，我们从注入`DocumentTemplate` bean 开始:**

```java
@Inject
DocumentTemplate documentTemplate;
```

然后，我们可以操纵`Todo`实体。例如，我们可以创建一个`Todo`:

```java
public Todo add(Todo todo) {
    return documentTemplate.insert(todo);
}
```

或者我们可以通过`id`检索一个`Todo`:

```java
public Todo get(String id) {
    Optional<Todo> todo = documentTemplate
      .find(Todo.class, id);
    return todo.get(); // or any other proper Optional handling
}
```

为了选择所有实体，我们构建一个`DocumentQuery`，然后调用`select()`方法:

```java
public List<Todo> getAll() {
    DocumentQuery query = select().from("Todo").build();
    return documentTemplate.select(query);
}
```

最后我们可以通过`id`删除一个`Todo`实体:

```java
public void delete(String id) {
    documentTemplate.delete(Todo.class, id);
}
```

### 3.5。使用存储库

除了`Template`类，我们还可以通过**`Repository`接口管理实体，该接口具有创建、更新、删除和检索**信息的方法。

为了使用`Repository`接口，我们只需提供一个`Repository:`的子接口

```java
public interface TodoRepository extends Repository<Todo, String> {
    List<Todo> findByName(String name);
    List<Todo> findAll();
}
```

通过下面的方法和参数命名约定，该接口的实现在运行时作为 CDI bean 提供。

在这个例子中，所有具有匹配的`name`的`Todo`实体都被`findByName()`方法检索到。

我们现在可以使用它:

```java
@Inject
TodoRepository todoRepository;
```

`Database`限定符让我们可以在同一个应用程序中使用多个 NoSQL 数据库。它有两个属性，类型和提供者。

如果数据库是多模型的，那么我们需要指定我们正在使用哪个模型:

```java
@Inject
@Database(value = DatabaseType.DOCUMENT)
TodoRepository todoRepository;
```

此外，如果我们有多个相同型号的数据库，我们需要指定提供者:

```java
@Inject
@Database(value = DatabaseType.DOCUMENT, provider="org.jnosql.diana.mongodb.document.MongoDBDocumentConfiguration")
TodoRepository todoRepository;
```

要运行该示例，只需访问 jnosql-artemis 模块并调用以下命令:

```java
mvn package liberty:run
```

这个命令通过 [liberty-maven-plugin](https://web.archive.org/web/20221221013845/https://github.com/WASdev/ci.maven#liberty-maven-plugin) 构建、部署并启动 [Open Liberty](https://web.archive.org/web/20221221013845/https://www.openliberty.io/) 服务器。

### 3.6。测试应用程序

由于应用程序公开了一个 REST 端点，我们可以使用任何 REST 客户端进行测试。这里我们使用了卷曲工具。

所以要保存一个 Todo 类:

```java
curl -d '{"id":"120", "name":"task120", "description":"Description 120"}' -H "Content-Type: application/json" -X POST http://localhost:9080/jnosql-artemis/todos
```

要做所有的事情:

```java
curl -H "Accept: application/json" -X GET http://localhost:9080/jnosql-artemis/todos
```

或者只做一件事:

```java
curl -H "Accept: application/json" -X GET http://localhost:9080/jnosql-artemis/todos/120
```

## 4。结论

在本教程中，我们探讨了 JNoSQL 如何能够抽象出与 NoSQL 数据库的交互。

**首先，我们已经使用 JNoSQL Diana API，用底层代码与数据库进行交互。然后，我们使用 JNoSQL Artemis API 来处理友好的 Java 注释模型**。

像往常一样，我们可以在 Github 上找到本文中使用的代码[。](https://web.archive.org/web/20221221013845/https://github.com/eugenp/tutorials/tree/master/persistence-modules/jnosql)