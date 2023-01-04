# 如何用 Java 在 AWS Lambda 函数中实现 Hibernate

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-aws-lambda-hibernate>

## 1.概观

AWS Lambda 允许我们创建可以轻松部署和扩展的轻量级应用程序。虽然我们可以使用像 [Spring Cloud Function](/web/20220524114622/https://www.baeldung.com/spring-cloud-function) 这样的框架，但是出于性能原因，我们通常会尽可能少地使用框架代码。

有时我们需要从 Lambda 访问关系数据库。这就是 Hibernate 和 T2 JPA 非常有用的地方。但是，**没有春天，我们怎么给 Lambda 添加 Hibernate 呢？**

在本教程中，我们将看看在 Lambda 中使用任何 RDBMS 的挑战，以及 Hibernate 如何以及何时有用。我们的示例将使用无服务器应用程序模型来构建数据的 REST 接口。

我们将看看如何使用 [Docker](/web/20220524114622/https://www.baeldung.com/tag/docker/) 和 AWS SAM CLI 在本地机器上测试一切。

## 2.在 Lambdas 中使用 RDBMS 和 Hibernate 的挑战

Lambda 代码需要尽可能小，以加速冷启动。此外，Lambda 应该能够在几毫秒内完成它的工作。然而，使用关系数据库会涉及大量的框架代码，运行速度会更慢。

在云原生应用中，我们尝试使用云原生技术进行设计。像 [Dynamo DB](/web/20220524114622/https://www.baeldung.com/aws-lambda-dynamodb-java) 这样的无服务器数据库可能更适合 Lambdas。然而，对关系数据库的需求可能来自于我们项目中的一些其他优先级。

### 2.1.使用 Lambda 中的 RDBMS

Lambdas 运行一小段时间，然后它们的容器暂停。容器可以在将来的调用中重用，或者如果不再需要，可以由 AWS 运行时处理掉。这意味着**容器要求的任何资源都必须在单次调用**的生命周期内小心管理。

具体来说，我们不能依赖传统的数据库连接池，因为任何打开的连接都可能保持打开状态，而不会被安全地处理掉。我们可以在调用期间使用连接池，但是每次我们都必须创建连接池。另外，**我们需要关闭所有连接，释放所有资源**，因为我们的功能结束了。

这意味着在数据库中使用 Lambda 会导致连接问题。我们的 Lambda 突然增大会消耗太多的连接。尽管 Lambda 可能会立即释放连接，但我们仍然依赖数据库为下一次 Lambda 调用做好准备。因此，**在任何使用关系数据库**的 Lambda 上使用最大并发限制通常是个好主意。

**在一些项目中，Lambda 并不是连接 RDBMS** 的最佳选择，而传统的带有连接池的 Spring 数据服务，也许在 EC2 或 ECS 中运行，也许是更好的解决方案。

### 2.2.Hibernate 案例

确定我们是否需要 Hibernate 的一个好方法是问一下如果没有它我们会写什么样的代码。

如果不使用 Hibernate 会导致我们必须编码复杂的连接或者字段和列之间的大量样板映射，那么从编码的角度来看，Hibernate 是一个很好的解决方案。如果我们的应用程序没有经历高负载或者对低延迟的需求，那么 Hibernate 的开销可能不是问题。

### 2.3.Hibernate 是一项重量级技术

然而，我们还需要考虑在 Lambda 中使用 Hibernate 的成本。

Hibernate jar 文件的大小是 7 MB。Hibernate 在启动时需要时间来检查注释并创建它的 ORM 功能。这是非常强大的，但对于一个 Lambda 来说，这可能是矫枉过正。由于 Lambdas 通常是为执行小任务而编写的，Hibernate 的开销可能不值得它带来的好处。

直接使用 [JDBC](/web/20220524114622/https://www.baeldung.com/java-jdbc) 可能更容易。或者，一个轻量级的类似 ORM 的框架，如 [JDBI](/web/20220524114622/https://www.baeldung.com/jdbi) 可以提供一个很好的查询抽象，而不会有太多的开销。

## 3.示例应用程序

在本教程中，我们将为一家小批量运输公司构建一个跟踪应用程序。让我们想象他们从顾客那里收集大件物品来创建一个`Consignment`。然后，无论货物运送到哪里，都会被打上时间戳，这样客户就可以监控了。每批货物都有一个`source`和`destination`，为此我们将使用[what3words.com](https://web.archive.org/web/20220524114622/https://what3words.com/goes.pepper.fence)作为我们的地理定位服务。

让我们想象一下，他们使用的移动设备连接和重试都不好。因此，在创建寄售之后，关于它的其余信息可以以任何顺序到达。这种复杂性，以及每批货物需要两个列表——物品和登记——是使用 Hibernate 的一个很好的理由。

### 3.1.API 设计

我们将使用以下方法创建一个 REST API:

*   `POST /consignment`–创建新的托运，返回 ID，并提供`source `和`destination`；必须在任何其他操作之前完成
*   `POST /consignment/{id}/item`–向托运货物中添加物品；总是添加到列表的末尾
*   `POST /consignment/{id}/checkin`–在沿途任何地点托运货物，提供`location`和时间戳；将始终按照时间戳的顺序保存在数据库中
*   `GET /consignment/{id}`–获得货物的完整历史记录，包括货物是否已经到达目的地

### 3.2.λ设计

我们将使用一个 Lambda 函数为这个 REST API 提供[无服务器应用模型](/web/20220524114622/https://www.baeldung.com/aws-serverless)来定义它。这意味着我们的单个 Lambda 处理函数将需要能够满足上述所有请求。

为了使测试变得快速和容易，没有部署到 AWS 的开销，我们将在我们的开发机器上测试一切。

### 4.创建 Lambda

让我们建立一个新的 Lambda 来满足我们的 API，但是还没有实现它的数据访问层。

### 4.1.先决条件

首先，如果我们还没有 Docker 的话，我们需要[安装它。我们将需要它来托管我们的测试数据库，AWS SAM CLI 使用它来模拟 Lambda 运行时。](https://web.archive.org/web/20220524114622/https://docs.docker.com/get-docker/)

我们可以测试我们是否有 Docker:

```
$ docker --version
Docker version 19.03.12, build 48a66213fe
```

接下来，我们需要[安装 AWS SAM CLI](https://web.archive.org/web/20220524114622/https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-sam-cli-install.html) ，然后测试它:

```
$ sam --version
SAM CLI, version 1.1.0
```

现在我们准备创建我们的 Lambda。

### 4.2.创建 SAM 模板

SAM CLI 为我们提供了一种创建新 Lambda 函数的方法:

```
$ sam init
```

这将提示我们新项目的设置。让我们选择以下选项:

```
1 - AWS Quick Start Templates
13 - Java 8
1 - maven
Project name - shipping-tracker
1 - Hello World Example: Maven
```

我们应该注意，这些选项编号可能会随着 SAM 工具的更高版本而变化。

现在，应该有一个名为`shipping-tracker`的新目录，其中有一个存根应用程序。如果我们查看其`template.yaml`文件的内容，我们会发现一个名为`HelloWorldFunction`的函数，带有一个简单的 REST API:

```
Events:
  HelloWorld:
    Type: Api 
    Properties:
      Path: /hello
      Method: get
```

默认情况下，这满足了对`/hello`的一个基本 GET 请求。我们应该通过使用`sam`来构建和测试它，快速测试一切都在工作:

```
$ sam build
... lots of maven output
$ sam start-api
```

然后我们可以使用`curl`测试`hello world` API:

```
$ curl localhost:3000/hello
{ "message": "hello world", "location": "192.168.1.1" }
```

之后，让我们通过使用`CTRL+C `中止程序来停止`sam`运行它的 API 监听器。

现在我们有了一个空的 Java 8 Lambda，我们需要定制它来成为我们的 API。

### 4.3.创建我们的 API

为了创建我们的 API，我们需要将我们自己的路径添加到`template.yaml`文件的`Events`部分:

```
CreateConsignment:
  Type: Api 
  Properties:
    Path: /consignment
    Method: post
AddItem:
  Type: Api
  Properties:
    Path: /consignment/{id}/item
    Method: post
CheckIn:
  Type: Api
  Properties:
    Path: /consignment/{id}/checkin
    Method: post
ViewConsignment:
  Type: Api
  Properties:
    Path: /consignment/{id}
    Method: get
```

让我们也将我们调用的函数从`HelloWorldFunction`重命名为`ShippingFunction`:

```
Resources:
  ShippingFunction:
    Type: AWS::Serverless::Function 
```

接下来，我们将目录 it 重命名为`ShippingFunction`，并将 Java 包从`helloworld`更改为`com.baeldung.lambda.shipping`。这意味着我们需要更新`template.yaml`中的`CodeUri`和`Handler`属性，以指向新的位置:

```
Properties:
  CodeUri: ShippingFunction
  Handler: com.baeldung.lambda.shipping.App::handleRequest
```

最后，为了给我们自己的实现腾出空间，让我们替换处理程序的主体:

```
public APIGatewayProxyResponseEvent handleRequest(APIGatewayProxyRequestEvent input, Context context) {
    Map<String, String> headers = new HashMap<>();
    headers.put("Content-Type", "application/json");
    headers.put("X-Custom-Header", "application/json");

    return new APIGatewayProxyResponseEvent()
      .withHeaders(headers)
      .withStatusCode(200)
      .withBody(input.getResource());
}
```

尽管单元测试是一个好主意，但是对于这个例子，我们也将通过删除`src/test`目录来删除所提供的单元测试。

### 4.4.测试空 API

现在我们已经移动了一些东西，并创建了我们的 API 和一个基本的处理程序，让我们仔细检查一下一切是否仍然工作:

```
$ sam build
... maven output
$ sam start-api
```

让我们使用`curl`来测试 HTTP GET 请求:

```
$ curl localhost:3000/consignment/123
/consignment/{id}
```

我们也可以使用`curl -d`来发布:

```
$ curl -d '{"source":"data.orange.brings", "destination":"heave.wipes.clay"}' \
  -H 'Content-Type: application/json' \
  http://localhost:3000/consignment/
/consignment
```

如我们所见，两个请求都成功结束。我们的存根代码输出`resource`——请求的路径——我们可以在设置到各种服务方法的路由时使用它。

### 4.5.在 Lambda 中创建端点

我们使用一个 Lambda 函数来处理我们的四个端点。我们可以在同一个代码库中为每个端点创建不同的处理程序类，或者为每个端点编写单独的应用程序，但是将相关的 API 放在一起可以让一个 Lambdas 机群为它们提供公共代码，这可以更好地利用资源。

然而，我们需要构建一个 REST 控制器的等价物，将每个请求分派给一个合适的 Java 函数。因此，我们将创建一个存根`ShippingService`类，并从处理程序路由到它:

```
public class ShippingService {
    public String createConsignment(Consignment consignment) {
        return UUID.randomUUID().toString();
    }

    public void addItem(String consignmentId, Item item) {
    }

    public void checkIn(String consignmentId, Checkin checkin) {
    }

    public Consignment view(String consignmentId) {
        return new Consignment();
    }
}
```

我们还将为`Consignment`、`Item,`和`Checkin`创建空类。这些很快就会成为我们的榜样。

现在我们有了一个服务，让我们使用`resource`来路由到适当的服务方法。我们将在处理程序中添加一个`switch`语句，将请求路由到服务:

```
Object result = "OK";
ShippingService service = new ShippingService();

switch (input.getResource()) {
    case "/consignment":
        result = service.createConsignment(
          fromJson(input.getBody(), Consignment.class));
        break;
    case "/consignment/{id}":
        result = service.view(input.getPathParameters().get("id"));
        break;
    case "/consignment/{id}/item":
        service.addItem(input.getPathParameters().get("id"),
          fromJson(input.getBody(), Item.class));
        break;
    case "/consignment/{id}/checkin":
        service.checkIn(input.getPathParameters().get("id"),
          fromJson(input.getBody(), Checkin.class));
        break;
}

return new APIGatewayProxyResponseEvent()
  .withHeaders(headers)
  .withStatusCode(200)
  .withBody(toJson(result));
```

我们可以使用[杰克逊](/web/20220524114622/https://www.baeldung.com/jackson)来实现我们的`fromJson`和`toJson`功能。

### 4.6.失败的实现

到目前为止，我们已经学习了如何创建一个 AWS Lambda 来支持一个 API，使用`sam`和`curl`来测试它，并在我们的处理程序中构建基本的路由功能。我们可以对错误的输入添加更多的错误处理。

我们应该注意到,`template.yaml`中的映射已经期望 AWS API 网关过滤不在我们 API 中正确路径上的请求。因此，我们需要对坏路径进行更少的错误处理。

现在，是时候用数据库、实体模型和 Hibernate 来实现我们的服务了。

## 5.设置数据库

对于这个例子，我们将使用 PostgreSQL 作为 RDBMS。任何关系数据库都可以工作。

### 5.1.在 Docker 中启动 PostgreSQL

首先，我们将提取一个 PostgreSQL docker 映像:

```
$ docker pull postgres:latest
... docker output
Status: Downloaded newer image for postgres:latest
docker.io/library/postgres:latest
```

现在让我们为这个数据库创建一个 docker 网络来运行。这个网络将允许我们的 Lambda 与数据库容器通信:

```
$ docker network create shipping
```

接下来，我们需要在该网络中启动数据库容器:

```
docker run --name postgres \
  --network shipping \
  -e POSTGRES_PASSWORD=password \
  -d postgres:latest
```

对于`–name,`，我们将容器命名为`postgres`。通过`–network,`,我们将它添加到了我们的`shipping` docker 网络中。为了设置服务器的密码，我们使用了环境变量`POSTGRES_PASSWORD`，用`-e`开关设置。

我们还使用`-d`在后台运行容器，而不是绑定我们的 shell。PostgreSQL 将在几秒钟后启动。

### 5.2.添加模式

我们需要一个新的表模式，所以让我们使用 PostgreSQL 容器中的`psql`客户端来添加`shipping`模式:

```
$ docker exec -it postgres psql -U postgres
psql (12.4 (Debian 12.4-1.pgdg100+1))
Type "help" for help.

postgres=#
```

在这个 shell 中，我们创建了模式:

```
postgres=# create schema shipping;
CREATE SCHEMA 
```

然后我们用`CTRL+D`退出 shell。

我们现在运行 PostgreSQL，准备好让 Lambda 使用它。

## 6.添加我们的实体模型和 DAO

现在我们有了一个数据库，让我们创建我们的实体模型和 DAO。虽然我们只使用了一个连接，但是让我们使用[光连接池](/web/20220524114622/https://www.baeldung.com/hikaricp)来看看如何为可能需要在一次调用中针对数据库运行多个连接的 Lambdas 配置它。

### 6.1.向项目中添加 Hibernate

我们将为 [Hibernate](https://web.archive.org/web/20220524114622/https://search.maven.org/artifact/org.hibernate/hibernate-core) 和[光连接池](https://web.archive.org/web/20220524114622/https://search.maven.org/artifact/org.hibernate/hibernate-hikaricp)添加对`pom.xml`的依赖。我们还将添加 [PostgreSQL JDBC 驱动程序](https://web.archive.org/web/20220524114622/https://search.maven.org/artifact/org.postgresql/postgresql):

```
<dependency>
    <groupId>org.hibernate</groupId>
    <artifactId>hibernate-core</artifactId>
    <version>5.4.21.Final</version>
</dependency>
<dependency>
    <groupId>org.hibernate</groupId>
    <artifactId>hibernate-hikaricp</artifactId>
    <version>5.4.21.Final</version>
</dependency>
<dependency>
    <groupId>org.postgresql</groupId>
    <artifactId>postgresql</artifactId>
    <version>42.2.16</version>
</dependency>
```

### 6.2.实体模型

让我们充实实体对象。一个`Consignment`有一个物品和值机的列表，还有它的`source`、`destination`，还有它是否已经送达(也就是是否已经签到了它的最终目的地):

```
@Entity(name = "consignment")
@Table(name = "consignment")
public class Consignment {
    private String id;
    private String source;
    private String destination;
    private boolean isDelivered;
    private List items = new ArrayList<>();
    private List checkins = new ArrayList<>();

    // getters and setters
}
```

我们将该类注释为一个实体，并带有一个表名。我们也会提供 getters 和 setters。让我们用列名来标记 getters:

```
@Id
@Column(name = "consignment_id")
public String getId() {
    return id;
}

@Column(name = "source")
public String getSource() {
    return source;
}

@Column(name = "destination")
public String getDestination() {
    return destination;
}

@Column(name = "delivered", columnDefinition = "boolean")
public boolean isDelivered() {
    return isDelivered;
}
```

对于我们的列表，我们将使用`@ElementCollection`注释使它们成为独立表中的有序列表，并与`consignment`表建立外键关系:

```
@ElementCollection(fetch = EAGER)
@CollectionTable(name = "consignment_item", joinColumns = @JoinColumn(name = "consignment_id"))
@OrderColumn(name = "item_index")
public List getItems() {
    return items;
}

@ElementCollection(fetch = EAGER)
@CollectionTable(name = "consignment_checkin", joinColumns = @JoinColumn(name = "consignment_id"))
@OrderColumn(name = "checkin_index")
public List getCheckins() {
    return checkins;
}
```

这就是 Hibernate 开始为自己买单的地方，执行管理集合的工作相当容易。

`Item`实体更简单:

```
@Embeddable
public class Item {
    private String location;
    private String description;
    private String timeStamp;

    @Column(name = "location")
    public String getLocation() {
        return location;
    }

    @Column(name = "description")
    public String getDescription() {
        return description;
    }

    @Column(name = "timestamp")
    public String getTimeStamp() {
        return timeStamp;
    }

    // ... setters omitted
}
```

它被标记为`@Embeddable`以使其成为父对象中列表定义的一部分。

类似地，我们将定义`Checkin`:

```
@Embeddable
public class Checkin {
    private String timeStamp;
    private String location;

    @Column(name = "timestamp")
    public String getTimeStamp() {
        return timeStamp;
    }

    @Column(name = "location")
    public String getLocation() {
        return location;
    }

    // ... setters omitted
} 
```

### 6.3.创建一个航运刀

我们的`ShippingDao`类将依赖于被传递一个开放休眠`Session`。这将需要`ShippingService` 来管理会话:

```
public void save(Session session, Consignment consignment) {
    Transaction transaction = session.beginTransaction();
    session.save(consignment);
    transaction.commit();
}

public Optional<Consignment> find(Session session, String id) {
    return Optional.ofNullable(session.get(Consignment.class, id));
}
```

稍后我们将把它连接到我们的`ShippingService` 中。

## 7.Hibernate 周期

到目前为止，我们的实体模型和 DAO 与非 Lambda 实现不相上下。下一个挑战是在 Lambda 的生命周期内创建一个 Hibernate `SessionFactory`。

### 7.1.数据库在哪里？

如果我们要从 Lambda 访问数据库，那么它需要是可配置的。让我们将 JDBC URL 和数据库凭证放入我们的`template.yaml`中的环境变量:

```
Environment: 
  Variables:
    DB_URL: jdbc:postgresql://postgres/postgres
    DB_USER: postgres
    DB_PASSWORD: password 
```

这些环境变量将被注入 Java 运行时。`postgres`用户是 Docker PostgreSQL 容器的默认用户。当我们之前启动容器时，我们将密码指定为`password`。

在`DB_URL`中，我们有服务器名称——`//postgres`是我们给容器的名称——数据库名称`postgres `是默认数据库。

值得注意的是，尽管我们在这个例子中对这些值进行了硬编码，但是 SAM 模板允许我们声明输入和`parameter overrides.` ,因此它们可以在以后进行参数化。

### 7.2.创建会话工厂

我们需要配置 Hibernate 和光连接池。为了给 Hibernate 提供设置，我们将它们添加到一个`Map`:

```
Map<String, String> settings = new HashMap<>();
settings.put(URL, System.getenv("DB_URL"));
settings.put(DIALECT, "org.hibernate.dialect.PostgreSQLDialect");
settings.put(DEFAULT_SCHEMA, "shipping");
settings.put(DRIVER, "org.postgresql.Driver");
settings.put(USER, System.getenv("DB_USER"));
settings.put(PASS, System.getenv("DB_PASSWORD"));
settings.put("hibernate.hikari.connectionTimeout", "20000");
settings.put("hibernate.hikari.minimumIdle", "1");
settings.put("hibernate.hikari.maximumPoolSize", "2");
settings.put("hibernate.hikari.idleTimeout", "30000");
settings.put(HBM2DDL_AUTO, "create-only");
settings.put(HBM2DDL_DATABASE_ACTION, "create");
```

这里，我们使用`System.getenv` 从环境中获取运行时设置。我们添加了`HBM2DDL_`设置，让我们的应用程序[生成数据库表](/web/20220524114622/https://www.baeldung.com/spring-data-jpa-generate-db-schema)。然而，我们应该在数据库模式生成后注释掉或删除这些行，并且应该避免让我们的 Lambda 在生产中这样做。不过，现在这对我们的测试很有帮助。

正如我们所见，许多设置已经在 Hibernate 的`AvailableSettings`类中定义了常量，尽管光特有的设置没有。

现在我们有了设置，我们需要构建`SessionFactory`。我们将分别向其中添加我们的实体类:

```
StandardServiceRegistry registry = new StandardServiceRegistryBuilder()
  .applySettings(settings)
  .build();

return new MetadataSources(registry)
  .addAnnotatedClass(Consignment.class)
  .addAnnotatedClass(Item.class)
  .addAnnotatedClass(Checkin.class)
  .buildMetadata()
  .buildSessionFactory();
```

### 7.3.管理资源

启动时，Hibernate 围绕实体对象执行代码生成。它并不打算让应用程序多次执行该操作，而是使用时间和内存来执行。所以，我们想在 Lambda 冷启动时做一次。

因此，我们应该创建`SessionFactory`，因为我们的处理程序对象是由 Lambda 框架创建的。我们可以在 handler 类的初始化列表中这样做:

```
private SessionFactory sessionFactory = createSessionFactory();
```

然而，因为我们的`SessionFactory`有一个连接池，所以存在一个风险，它会在调用之间保持连接打开，从而占用数据库资源。

更糟糕的是，**没有生命周期事件允许 Lambda 关闭被 AWS 运行时处理的资源**。因此，以这种方式保持的连接有可能永远不会被正确释放。

我们可以通过挖掘连接池的`SessionFactory`并显式关闭任何连接来解决这个问题:

```
private void flushConnectionPool() {
    ConnectionProvider connectionProvider = sessionFactory.getSessionFactoryOptions()
      .getServiceRegistry()
      .getService(ConnectionProvider.class);
    HikariDataSource hikariDataSource = connectionProvider.unwrap(HikariDataSource.class);
    hikariDataSource.getHikariPoolMXBean().softEvictConnections();
}
```

这在这种情况下是可行的，因为我们指定了光连接池，它提供了`softEvictConnections`来允许我们释放它的连接。

我们应该注意到，`SessionFactory`的`close`方法也会关闭连接，但是它也会使`SessionFactory`不可用。

### 7.4.添加到处理程序

**现在，我们需要确保处理程序使用会话工厂并释放它的连接**。记住这一点，让我们将大部分控制器功能提取到一个名为`routeRequest`的方法中，并修改我们的处理程序来释放`finally`块中的资源:

```
try {
    ShippingService service = new ShippingService(sessionFactory, new ShippingDao());
    return routeRequest(input, service);
} finally {
    flushConnectionPool();
} 
```

我们还修改了我们的`Shipping` `Service`，将`SessionFactory`和`ShippingDao`作为属性，通过构造函数注入，但是它还没有使用它们。

### 7.5.测试休眠

此时，尽管`ShippingService`什么都不做，但是调用 Lambda 应该会导致 Hibernate 启动并生成 DDL。

在注释掉它的设置之前，让我们仔细检查它生成的 DDL:

```
$ sam build
$ sam local start-api --docker-network shipping
```

我们像以前一样构建应用程序，但是现在我们将参数`–docker-network`添加到`sam local`中。**它在与我们的数据库**相同的网络中运行测试 Lambda，这样 Lambda 就可以通过使用它的容器名到达数据库容器。

当我们第一次使用`curl`到达终点时，我们的表应该被创建:

```
$ curl localhost:3000/consignment/123
{"id":null,"source":null,"destination":null,"items":[],"checkins":[],"delivered":false}
```

存根代码仍然返回空白的`Consignment`。但是，现在让我们检查数据库，看看这些表是否已经创建:

```
$ docker exec -it postgres pg_dump -s -U postgres
... DDL output
CREATE TABLE shipping.consignment_item (
    consignment_id character varying(255) NOT NULL,
...
```

一旦我们对 Hibernate 设置的运行感到满意，我们就可以注释掉`HBM2DDL_`设置。

## 8.完成业务逻辑

剩下的就是让`ShippingService`使用`ShippingDao`来实现业务逻辑。每个方法都将在`try-with-resources`块中创建一个会话工厂，以确保它被关闭。

### 8.1.创建寄售

新的货物尚未交付，应该会收到一个新的 ID。那么我们应该将它保存在数据库中:

```
public String createConsignment(Consignment consignment) {
    try (Session session = sessionFactory.openSession()) {
        consignment.setDelivered(false);
        consignment.setId(UUID.randomUUID().toString());
        shippingDao.save(session, consignment);
        return consignment.getId();
    }
}
```

### 8.2.查看托运

要获得一批货物，我们需要通过 ID 从数据库中读取它。虽然 REST API 应该对未知请求返回`Not Found`,但是对于这个例子，如果没有找到，我们将返回一个空的委托:

```
public Consignment view(String consignmentId) {
    try (Session session = sessionFactory.openSession()) {
        return shippingDao.find(session, consignmentId)
          .orElseGet(Consignment::new);
    }
}
```

### 8.3.添加项

项目将按照收到的顺序进入我们的项目列表:

```
public void addItem(String consignmentId, Item item) {
    try (Session session = sessionFactory.openSession()) {
        shippingDao.find(session, consignmentId)
          .ifPresent(consignment -> addItem(session, consignment, item));
    }
}

private void addItem(Session session, Consignment consignment, Item item) {
    consignment.getItems()
      .add(item);
    shippingDao.save(session, consignment);
}
```

理想情况下，如果委托不存在，我们会有更好的错误处理，但是对于这个例子，不存在的委托将被忽略。

### 8.4.登记入住

签入需要按照发生的时间排序，而不是按照收到请求的时间排序。此外，当物品到达最终目的地时，应标记为已送达:

```
public void checkIn(String consignmentId, Checkin checkin) {
    try (Session session = sessionFactory.openSession()) {
        shippingDao.find(session, consignmentId)
          .ifPresent(consignment -> checkIn(session, consignment, checkin));
    }
}

private void checkIn(Session session, Consignment consignment, Checkin checkin) {
    consignment.getCheckins().add(checkin);
    consignment.getCheckins().sort(Comparator.comparing(Checkin::getTimeStamp));
    if (checkin.getLocation().equals(consignment.getDestination())) {
        consignment.setDelivered(true);
    }
    shippingDao.save(session, consignment);
}
```

## 9.测试应用程序

让我们模拟一个从白宫到帝国大厦的包裹旅行。

代理创建旅程:

```
$ curl -d '{"source":"data.orange.brings", "destination":"heave.wipes.clay"}' \
  -H 'Content-Type: application/json' \
  http://localhost:3000/consignment/

"3dd0f0e4-fc4a-46b4-8dae-a57d47df5207"
```

我们现在有了货物的 ID `3dd0f0e4-fc4a-46b4-8dae-a57d47df5207`。然后，有人收集了托运的两件物品——一幅画和一架钢琴:

```
$ curl -d '{"location":"data.orange.brings", "timeStamp":"20200101T120000", "description":"picture"}' \
  -H 'Content-Type: application/json' \
  http://localhost:3000/consignment/3dd0f0e4-fc4a-46b4-8dae-a57d47df5207/item
"OK"

$ curl -d '{"location":"data.orange.brings", "timeStamp":"20200101T120001", "description":"piano"}' \
  -H 'Content-Type: application/json' \
  http://localhost:3000/consignment/3dd0f0e4-fc4a-46b4-8dae-a57d47df5207/item
"OK" 
```

一段时间后，有一个签到:

```
$ curl -d '{"location":"united.alarm.raves", "timeStamp":"20200101T173301"}' \
-H 'Content-Type: application/json' \
http://localhost:3000/consignment/3dd0f0e4-fc4a-46b4-8dae-a57d47df5207/checkin
"OK"
```

后来又一次:

```
$ curl -d '{"location":"wink.sour.chasing", "timeStamp":"20200101T191202"}' \
-H 'Content-Type: application/json' \
http://localhost:3000/consignment/3dd0f0e4-fc4a-46b4-8dae-a57d47df5207/checkin
"OK"
```

此时，客户请求货物的状态:

```
$ curl http://localhost:3000/consignment/3dd0f0e4-fc4a-46b4-8dae-a57d47df5207
{
  "id":"3dd0f0e4-fc4a-46b4-8dae-a57d47df5207",
  "source":"data.orange.brings",
  "destination":"heave.wipes.clay",
  "items":[
    {"location":"data.orange.brings","description":"picture","timeStamp":"20200101T120000"},
    {"location":"data.orange.brings","description":"piano","timeStamp":"20200101T120001"}
  ],
  "checkins":[
    {"timeStamp":"20200101T173301","location":"united.alarm.raves"},
    {"timeStamp":"20200101T191202","location":"wink.sour.chasing"}
  ],
  "delivered":false
}% 
```

他们看到了进展，但还没有交付。

应该在 20:12 发送一条消息说它到达了`deflection.famed.apple`，但是它被延迟了，并且来自目的地的 21:46 的消息首先到达那里:

```
$ curl -d '{"location":"heave.wipes.clay", "timeStamp":"20200101T214622"}' \
-H 'Content-Type: application/json' \
http://localhost:3000/consignment/3dd0f0e4-fc4a-46b4-8dae-a57d47df5207/checkin
"OK"
```

此时，客户请求货物的状态:

```
$ curl http://localhost:3000/consignment/3dd0f0e4-fc4a-46b4-8dae-a57d47df5207
{
  "id":"3dd0f0e4-fc4a-46b4-8dae-a57d47df5207",
...
    {"timeStamp":"20200101T191202","location":"wink.sour.chasing"},
    {"timeStamp":"20200101T214622","location":"heave.wipes.clay"}
  ],
  "delivered":true
} 
```

现在送来了。因此，当延迟的消息通过时:

```
$ curl -d '{"location":"deflection.famed.apple", "timeStamp":"20200101T201254"}' \
-H 'Content-Type: application/json' \
http://localhost:3000/consignment/3dd0f0e4-fc4a-46b4-8dae-a57d47df5207/checkin
"OK"

$ curl http://localhost:3000/consignment/3dd0f0e4-fc4a-46b4-8dae-a57d47df5207
{
"id":"3dd0f0e4-fc4a-46b4-8dae-a57d47df5207",
...
{"timeStamp":"20200101T191202","location":"wink.sour.chasing"},
{"timeStamp":"20200101T201254","location":"deflection.famed.apple"},
{"timeStamp":"20200101T214622","location":"heave.wipes.clay"}
],
"delivered":true
} 
```

签入被放在时间轴中的正确位置。

## 10.结论

在本文中，我们讨论了在 AWS Lambda 这样的轻量级容器中使用 Hibernate 这样的重量级框架所面临的挑战。

我们构建了一个 Lambda 和 REST API，并学习了如何使用 Docker 和 AWS SAM CLI 在本地机器上测试它。然后，我们为 Hibernate 构建了一个实体模型，用于我们的数据库。我们还使用 Hibernate 来初始化我们的表。

最后，我们将 Hibernate `SessionFactory`集成到我们的应用程序中，确保在 Lambda 退出之前关闭它。

和往常一样，本文的示例代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220524114622/https://github.com/eugenp/tutorials/tree/master/aws-modules/aws-lambda)