# 使用 Java 的 Neo4J 指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-neo4j>

## 1。简介

这篇文章是关于[`Neo4j`](https://web.archive.org/web/20220926184025/https://neo4j.com/)——当今市场上最成熟、功能最全的图形数据库之一。图形数据库处理数据建模任务的观点是，生活中的许多事情都适合表示为一组`nodes` (V)和它们之间的连接`edges` (E)。

## 2。嵌入式 Neo4j

开始使用`Neo4j`最简单的方法是使用嵌入式版本，其中`Neo4j`与您的应用程序运行在同一个 JVM 中。

首先，我们需要添加一个 Maven 依赖项:

```java
<dependency>
    <groupId>org.neo4j</groupId>
    <artifactId>neo4j</artifactId>
    <version>3.4.6</version>
</dependency>
```

可以查看[这个链接](https://web.archive.org/web/20220926184025/https://search.maven.org/search?q=g:org.neo4j%20AND%20a:neo4j&core=gav)下载最新版本。

接下来，让我们创建一个工厂:

```java
GraphDatabaseFactory graphDbFactory = new GraphDatabaseFactory();
```

最后，我们创建一个嵌入式数据库:

```java
GraphDatabaseService graphDb = graphDbFactory.newEmbeddedDatabase(
  new File("data/cars"));
```

现在真正的行动可以开始了！首先，我们需要在我们的图中创建一些节点，为此，我们需要启动一个事务，因为`Neo4j`将拒绝任何破坏性操作，除非已经启动了一个事务:

```java
graphDb.beginTx();
```

一旦我们有了正在进行的事务，我们就可以开始添加节点:

```java
Node car = graphDb.createNode(Label.label("Car"));
car.setProperty("make", "tesla");
car.setProperty("model", "model3");

Node owner = graphDb.createNode(Label.label("Person"));
owner.setProperty("firstName", "baeldung");
owner.setProperty("lastName", "baeldung");
```

这里我们添加了一个属性为`make`和`model`的节点`Car`，以及属性为`firstName`和`lastName`的节点`Person`

现在我们可以添加一个关系:

```java
owner.createRelationshipTo(car, RelationshipType.withName("owner"));
```

上面的语句添加了一条连接两个带有`owner`标签的节点的边。我们可以通过运行用强大的`Cypher`语言编写的查询来验证这种关系:

```java
Result result = graphDb.execute(
  "MATCH (c:Car) <-[owner]- (p:Person) " +
  "WHERE c.make = 'tesla'" +
  "RETURN p.firstName, p.lastName");
```

在这里，我们要求找到任何特斯拉汽车的车主，并返回他/她的名字和姓氏。不出所料，这个返回:`{p.firstName=baeldung, p.lastName=baeldung}`

## 3。密码查询语言

提供了一个非常强大和非常直观的查询语言，它支持数据库的所有功能。让我们看看如何完成标准创建、检索、更新和删除任务。

### 3.1。创建节点

Create 关键字可用于创建节点和关系。

```java
CREATE (self:Company {name:"Baeldung"})
RETURN self
```

在这里，我们创建了一个拥有单一资产的公司`name`。节点定义用括号标记，其属性用花括号括起来。在这种情况下，`self`是节点的别名，`Company` 是节点标签。

### 3.2。创建关系

可以在一个查询中创建一个节点和与该节点的关系:

```java
Result result = graphDb.execute(
  "CREATE (baeldung:Company {name:\"Baeldung\"}) " +
  "-[:owns]-> (tesla:Car {make: 'tesla', model: 'modelX'})" +
  "RETURN baeldung, tesla");
```

这里我们已经创建了节点`baeldung`和`tesla`，并在它们之间建立了所有权关系。当然，创建与预先存在的节点的关系也是可能的。

### 3.3。检索数据

`MATCH`关键字用于查找数据，结合`RETURN`控制返回哪些数据点。`WHERE`子句可以用来过滤掉那些具有我们想要的属性的节点。

让我们找出拥有特斯拉 modelX 的公司名称:

```java
Result result = graphDb.execute(
  "MATCH (company:Company)-[:owns]-> (car:Car)" +
  "WHERE car.make='tesla' and car.model='modelX'" +
  "RETURN company.name");
```

### 3.4。更新节点

`SET`关键字可用于更新节点属性或标签。让我们为我们的特斯拉增加里程:

```java
Result result = graphDb.execute("MATCH (car:Car)" +
  "WHERE car.make='tesla'" +
  " SET car.milage=120" +
  " SET car :Car:Electro" +
  " SET car.model=NULL" +
  " RETURN car");
```

这里我们添加了一个名为`milage`的新属性，将标签修改为`Car`和`Electro`，最后，我们将`model`属性一起删除。

### 3.5。删除节点

DELETE 关键字可用于从图形中永久删除节点或关系:

```java
graphDb.execute("MATCH (company:Company)" +
  " WHERE company.name='Baeldung'" +
  " DELETE company");
```

这里我们删除了一家名为 Baeldung 的公司。

### 3.6。参数绑定

在上面的例子中，我们有硬编码的参数值，这不是最佳实践。幸运的是，`Neo4j`提供了将变量绑定到查询的工具:

```java
Map<String, Object> params = new HashMap<>();
params.put("name", "baeldung");
params.put("make", "tesla");
params.put("model", "modelS");

Result result = graphDb.execute("CREATE (baeldung:Company {name:$name}) " +
  "-[:owns]-> (tesla:Car {make: $make, model: $model})" +
  "RETURN baeldung, tesla", params);
```

## 4。Java 驱动程序

到目前为止，我们一直在寻找与嵌入式`Neo4j`实例的交互，然而，在所有生产可能性中，我们希望运行一个独立的服务器，并通过提供的驱动程序连接到它。首先，我们需要在 maven `pom.xml`中添加另一个依赖项:

```java
<dependency>
    <groupId>org.neo4j.driver</groupId>
    <artifactId>neo4j-java-driver</artifactId>
    <version>1.6.2</version>
</dependency>
```

你可以点击[这个链接](https://web.archive.org/web/20220926184025/https://search.maven.org/search?q=a:neo4j-java-driver)来查看这个驱动程序的最新版本。

现在我们可以建立联系了:

```java
Driver driver = GraphDatabase.driver(
  "bolt://localhost:7687", AuthTokens.basic("neo4j", "12345"));
```

然后，创建一个会话:

```java
Session session = driver.session();
```

最后，我们可以运行一些查询:

```java
session.run("CREATE (baeldung:Company {name:\"Baeldung\"}) " +
  "-[:owns]-> (tesla:Car {make: 'tesla', model: 'modelX'})" +
  "RETURN baeldung, tesla");
```

完成所有工作后，我们需要关闭会话和驱动程序:

```java
session.close();
driver.close();
```

## 5。JDBC 司机

也可以通过 JDBC 驱动程序与`Neo4j`交互。我们`pom.xml`的另一个依赖:

```java
<dependency>
    <groupId>org.neo4j</groupId>
    <artifactId>neo4j-jdbc-driver</artifactId>
    <version>3.4.0</version>
</dependency>
```

你可以点击[这个链接](https://web.archive.org/web/20220926184025/https://search.maven.org/search?q=a:neo4j-jdbc-driver)下载这个驱动的最新版本。

接下来，让我们建立一个 JDBC 连接:

```java
Connection con = DriverManager.getConnection(
  "jdbc:neo4j:bolt://localhost/?user=neo4j,password=12345,scheme=basic");
```

这里的`con`是一个常规的 JDBC 连接，可用于创建和执行语句或准备好的语句:

```java
try (Statement stmt = con.
  stmt.execute("CREATE (baeldung:Company {name:\"Baeldung\"}) "
  + "-[:owns]-> (tesla:Car {make: 'tesla', model: 'modelX'})"
  + "RETURN baeldung, tesla")

    ResultSet rs = stmt.executeQuery(
      "MATCH (company:Company)-[:owns]-> (car:Car)" +
      "WHERE car.make='tesla' and car.model='modelX'" +
      "RETURN company.name");

    while (rs.next()) {
        rs.getString("company.name");
    }
}
```

## 6。对象图映射

对象图映射或 OGM 是一种技术，它使我们能够使用我们的领域 POJOs 作为`Neo4j`数据库中的实体。让我们来看看这是如何工作的。第一步，通常，我们向我们的`pom.xml`添加新的依赖项:

```java
<dependency>
    <groupId>org.neo4j</groupId>
    <artifactId>neo4j-ogm-core</artifactId>
    <version>3.1.2</version>
</dependency>

<dependency> 
    <groupId>org.neo4j</groupId>
    <artifactId>neo4j-ogm-embedded-driver</artifactId>
    <version>3.1.2</version>
</dependency>
```

你可以查看 [OGM 核心链接](https://web.archive.org/web/20220926184025/https://search.maven.org/classic/#artifactdetails%7Corg.neo4j%7Cneo4j-ogm%7C2.1.1%7Cpom)和 [OGM 嵌入式驱动链接](https://web.archive.org/web/20220926184025/https://search.maven.org/classic/#artifactdetails%7Corg.neo4j%7Cneo4j-ogm-embedded-driver%7C2.1.1%7Cjar)来查看这些库的最新版本。

其次，我们用 OGM 注释来注释 POJO:

```java
@NodeEntity
public class Company {
    private Long id;

    private String name;

    @Relationship(type="owns")
    private Car car;
}

@NodeEntity
public class Car {
    private Long id;

    private String make;

    @Relationship(direction = "INCOMING")
    private Company company;
}
```

`@NodeEntity`通知`Neo4j`这个对象将需要由结果图中的一个节点来表示。`@Relationship`传达与代表相关类型的节点创建关系的需求。在这种情况下，一个`company`拥有一个`car`。

请注意，`Neo4j`要求每个实体都有一个主键，默认情况下会选择一个名为`id`的字段。通过用 *@Id @GeneratedValue 对其进行注释，可以使用另一个命名的字段。*

然后，我们需要创建一个用于引导`Neo4j`的 OGM 的配置。为简单起见，让我们使用一个嵌入式内存数据库:

```java
Configuration conf = new Configuration.Builder().build();
```

之后，我们用我们创建的配置和包名初始化`SessionFactory`,带注释的 POJO 驻留在这个包名中:

```java
SessionFactory factory = new SessionFactory(conf, "com.baeldung.graph");
```

最后，我们可以创建一个`Session`并开始使用它:

```java
Session session = factory.openSession();
Car tesla = new Car("tesla", "modelS");
Company baeldung = new Company("baeldung");

baeldung.setCar(tesla);
session.save(baeldung);
```

在这里，我们启动了一个会话，创建了 POJO，并请求 OGM 会话持久化它们。`Neo4j` OGM 运行时透明地将对象转换成一组`Cypher`查询，这些查询在数据库中创建适当的节点和边。

如果这个过程看起来很熟悉，那是因为它确实如此！这正是 JPA 的工作方式，唯一的区别是对象是被转换成持久化到 RDBMS 的行，还是持久化到图形数据库的一系列节点和边。

## 7。结论

本文研究了面向图形的数据库 Neo4j 的一些基础知识。

和往常一样，这篇文章中的代码都可以在 Github 的[上找到。](https://web.archive.org/web/20220926184025/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-data-neo4j)