# Ebean ORM 指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/ebean-orm>

## 1.介绍

**[Ebean](https://web.archive.org/web/20220524015350/https://ebean-orm.github.io/) 是一个用 Java 编写的对象关系映射工具。**

它支持用于声明实体的标准 JPA 注释。然而，它为持久化提供了更简单的 API。事实上，关于 Ebean 架构值得一提的一点是，它是无会话的，这意味着它不能完全管理实体。

除此之外，它还附带了一个查询 API，并支持用原生 SQL 编写查询。Ebean 支持所有主要的数据库提供商，如 Oracle、Postgres、MySql、H2 等。

在本教程中，我们将看看如何使用 Ebean 和 H2 创建、持久化和查询实体。

## 2.设置

首先，让我们了解我们的依赖项以及一些基本配置。

### 2.1.Maven 依赖性

在开始之前，让我们导入所需的依赖项:

```java
<dependency>
    <groupId>io.ebean</groupId>
    <artifactId>ebean</artifactId>
    <version>11.22.4</version>
</dependency>
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <version>1.4.196</version>
</dependency>
<dependency>
    <groupId>ch.qos.logback</groupId>
    <artifactId>logback-classic</artifactId>
    <version>1.2.6</version>
</dependency>
```

最新版本的 [Ebean](https://web.archive.org/web/20220524015350/https://search.maven.org/search?q=g:io.ebean%20AND%20a:ebean&core=gav) 、 [H2](https://web.archive.org/web/20220524015350/https://search.maven.org/search?q=g:com.h2database%20AND%20a:h2&core=gav) 和 [Logback](https://web.archive.org/web/20220524015350/https://search.maven.org/search?q=g:ch.qos.logback%20AND%20a:logback-classic&core=gav) 可以在 Maven Central 上找到。

### 2.2.提高

Ebean 需要修改实体 bean，以便它们可以被服务器管理。因此，我们将添加一个 Maven 插件来完成这项工作:

```java
<plugin>
    <groupId>io.ebean</groupId>
    <artifactId>ebean-maven-plugin</artifactId>
    <version>11.11.2</version>
    <executions>
        <execution>
            <id>main</id>
            <phase>process-classes</phase>
            <configuration>
                <transformArgs>debug=1</transformArgs>
            </configuration>
            <goals>
                <goal>enhance</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

我们还需要向 Maven 插件提供包含使用事务的实体和类的包的名称。为此，我们创建了文件`ebean.mf:`

```java
entity-packages: com.baeldung.ebean.model
transactional-packages: com.baeldung.ebean.app
```

### 2.3.记录

让我们也创建`logback.xml` 并将一些包的日志级别设置为`TRACE`，这样我们就可以看到正在执行的语句:

```java
<logger name="io.ebean.DDL" level="TRACE"/>
<logger name="io.ebean.SQL" level="TRACE"/>
<logger name="io.ebean.TXN" level="TRACE"/>
```

## 3.配置服务器

我们需要创建一个`EbeanServer`实例来保存实体或在数据库上运行查询。有两种方法可以创建服务器实例——使用默认属性文件或以编程方式创建。

### 3.1.使用默认属性文件

默认属性文件可以是类型`properties`或`yaml`。Ebean 将在名为`application.properties`、`ebean.properties`或`application.yml`的文件中搜索配置。

除了提供数据库连接细节，我们还可以指示 Ebean 创建并运行 DDL 语句。

现在，让我们来看一个示例配置:

```java
ebean.db.ddl.generate=true
ebean.db.ddl.run=true

datasource.db.username=sa
datasource.db.password=
datasource.db.databaseUrl=jdbc:h2:mem:customer
datasource.db.databaseDriver=org.h2.Driver
```

### 3.2.使用 ServerConfig

接下来，让我们看看如何使用`EbeanServerFactory`和`ServerConfig`以编程方式创建相同的服务器:

```java
ServerConfig cfg = new ServerConfig();

Properties properties = new Properties();
properties.put("ebean.db.ddl.generate", "true");
properties.put("ebean.db.ddl.run", "true");
properties.put("datasource.db.username", "sa");
properties.put("datasource.db.password", "");
properties.put("datasource.db.databaseUrl","jdbc:h2:mem:app2";
properties.put("datasource.db.databaseDriver", "org.h2.Driver");

cfg.loadFromProperties(properties);
EbeanServer server = EbeanServerFactory.create(cfg);
```

### 3.3.默认服务器实例

**单个`EbeanServer`实例映射到单个数据库。**根据我们的需求，我们也可以创建多个`EbeanServer`实例。

如果只创建了一个服务器实例，默认情况下，它会注册为默认服务器实例。使用`Ebean`类上的静态方法，可以在应用程序的任何地方访问它:

```java
EbeanServer server = Ebean.getDefaultServer();
```

如果有多个数据库，可以将其中一个服务器实例注册为默认实例:

```java
cfg.setDefaultServer(true);
```

## 4.创建实体

Ebean 提供了对 JPA 注释的完全支持，以及使用它自己的注释的附加特性。

让我们使用 JPA 和 Ebean 注释创建几个实体。首先，我们将创建一个`BaseModel`，它包含实体`:`共有的属性

```java
@MappedSuperclass
public abstract class BaseModel {

    @Id
    protected long id;

    @Version
    protected long version;

    @WhenCreated
    protected Instant createdOn;

    @WhenModified
    protected Instant modifiedOn;

    // getters and setters
}
```

这里，我们使用了`MappedSuperClass` JPA 注释来定义`BaseModel.`和两个用于审计目的的 Ebean 注释`io.ebean.annotation.WhenCreated` 和`io.ebean.annotation.WhenModified`。

接下来，我们将创建两个实体`Customer`和`Address`，它们扩展了`BaseModel`:

```java
@Entity
public class Customer extends BaseModel {

    public Customer(String name, Address address) {
        super();
        this.name = name;
        this.address = address;
    }

    private String name;

    @OneToOne(cascade = CascadeType.ALL)
    Address address;

    // getters and setters
} 
```

```java
@Entity
public class Address extends BaseModel{

    public Address(String addressLine1, String addressLine2, String city) {
        super();
        this.addressLine1 = addressLine1;
        this.addressLine2 = addressLine2;
        this.city = city;
    }

    private String addressLine1;
    private String addressLine2;
    private String city;

    // getters and setters
}
```

在`Customer`中，我们已经定义了与`Address`的一对一映射，并为`ALL`添加了集合级联类型，以便子实体也随父实体一起更新。

## 5.基本 CRUD 操作

前面我们已经看到了如何配置`EbeanServer`并创建了两个实体。现在，**让我们对它们进行一些基本的 CRUD 操作**。

我们将使用默认的服务器实例来保存和访问数据。`Ebean`类还提供静态方法来保存和访问数据，这些方法将请求代理到默认的服务器实例:

```java
Address a1 = new Address("5, Wide Street", null, "New York");
Customer c1 = new Customer("John Wide", a1);

EbeanServer server = Ebean.getDefaultServer();
server.save(c1);

c1.setName("Jane Wide");
c1.setAddress(null);
server.save(c1);

Customer foundC1 = Ebean.find(Customer.class, c1.getId());

Ebean.delete(foundC1);
```

首先，我们创建一个`Customer`对象，并使用默认的服务器实例通过 `save()`保存它。

接下来，我们将更新客户详细信息，并使用`save()`再次保存。

最后，我们在`Ebean`上使用静态方法`find()`来获取客户，并使用`delete()`删除它。

## 6.问题

**查询 API 也可以用来创建带有过滤器和谓词的对象图。**我们可以使用`Ebean` 或`EbeanServer`来创建和执行查询。

让我们来看一个查询，它根据城市找到一个`Customer`，并返回一个`Customer`和`Address `对象，其中只填充了一些字段:

```java
Customer customer = Ebean.find(Customer.class)
            .select("name")
            .fetch("address", "city")
            .where()
            .eq("city", "San Jose")
            .findOne();
```

在这里，我们用`find()`表示我们想要查找类型为`Customer`的实体。接下来，我们使用`select()`来指定要填充到`Customer`对象中的属性。

稍后，我们使用`fetch()`来表示我们想要获取属于`Customer`的`Address`对象，并且我们想要获取`city` 字段`.`

最后，我们添加一个谓词，并将结果的大小限制为 1。

## 7.处理

默认情况下，Ebean 在新事务中执行每个语句或查询。

尽管在某些情况下这可能不是问题。有时，我们可能希望在单个事务中执行一组语句。

在这种情况下，如果我们用`io.ebean.annotations.Transactional, `注释该方法，那么该方法中的所有语句都将在同一个事务中执行:

```java
@Transactional
public static void insertAndDeleteInsideTransaction() {
    Customer c1 = getCustomer();
    EbeanServer server = Ebean.getDefaultServer();
    server.save(c1);
    Customer foundC1 = server.find(Customer.class, c1.getId());
    server.delete(foundC1);
}
```

## 8.构建项目

最后，我们可以使用以下命令构建 Maven 项目:

```java
compile io.ebean:ebean-maven-plugin:enhance
```

## 9.结论

总而言之，我们已经了解了 Ebean 的基本特性，它可以用来在关系数据库中持久化和查询实体。

最后，这段代码可以在 [Github](https://web.archive.org/web/20220524015350/https://github.com/eugenp/tutorials/tree/master/libraries-data-db) 上找到。