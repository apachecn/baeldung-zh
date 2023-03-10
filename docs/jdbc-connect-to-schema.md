# 连接到 JDBC 的特定模式

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jdbc-connect-to-schema>

## 1.介绍

在本文中，我们将介绍数据库模式的基础知识，为什么我们需要它们，以及它们如何有用。之后，我们将重点讨论在 JDBC 用 PostgreSQL 作为数据库设置模式的实际例子。

## 2。什么是数据库模式

通常，数据库模式是一组管理数据库的规则。它是数据库的一个额外的抽象层。有两种模式:

1.  逻辑数据库模式定义了应用于存储在数据库中的数据的规则。
2.  物理数据库模式定义了数据在存储系统中的物理存储方式。

在 PostgreSQL 中，schema 指的是第一种。**模式是一个逻辑名称空间，包含数据库对象，如表、视图、索引等。**每个模式属于一个数据库，每个数据库至少有一个模式。如果没有另外指定，`the default schema in PostgreSQL is public.` 我们创建的每个数据库对象，在没有指定模式的情况下，都属于`public`模式。

PostgreSQL 中的模式允许我们将表和视图组织成组，使它们更易于管理。这样，我们可以在更细粒度的级别上设置数据库对象的特权。此外，模式允许多个用户同时使用相同的数据库，而不会相互干扰。

## 3.如何在 PostgreSQL 中使用模式

要访问数据库模式的对象，我们必须在要使用的给定数据库对象的名称之前指定模式的名称。例如，要查询模式 s `tore,` 中的表`product` ，我们需要使用表的限定名:

```java
SELECT * FROM store.product;
```

建议是避免硬编码模式名，以防止将具体的模式耦合到我们的应用程序。这意味着我们直接使用数据库对象名，让数据库系统决定使用哪个模式。PostgreSQL 通过遵循搜索路径来确定在哪里搜索给定的表。

### 3.1.PostgreSQL `search_path`

搜索路径是定义数据库系统对给定数据库对象的搜索的模式的有序列表。如果对象出现在任何(或多个)模式中，我们得到第一个找到的匹配项。否则，我们会得到一个错误。搜索路径中的第一个模式也称为当前模式。要预览搜索路径上有哪些模式，我们可以使用以下查询:

```java
SHOW search_path;
```

默认 PostgreSQL 配置将返回`$user`和公共模式。我们已经提到的公共模式`$user`模式是以当前用户命名的模式，它可能不存在。在这种情况下，数据库会忽略该模式。

要将商店模式添加到搜索路径中，我们可以执行以下查询:

```java
SET search_path TO store,public;
```

在此之后，我们可以查询产品表，而无需指定模式。此外，我们可以从搜索路径中删除公共模式。

如上所述设置搜索路径是角色级别的配置。我们可以通过更改`postgresql.conf`文件并重新加载数据库实例来更改整个数据库上的搜索路径。

### 3.2.JDBC 网址

我们可以使用 [JDBC](/web/20220628113624/https://www.baeldung.com/java-jdbc) URL 来指定连接建立期间的各种参数。通常的参数是数据库类型、地址、端口、数据库名称等。**从[Postgres 9.4 版本开始。](https://web.archive.org/web/20220628113624/https://jdbc.postgresql.org/documentation/94/connect.html#connection-parameters)增加了使用 URL 指定当前模式的支持。**

在我们将这个概念付诸实践之前，让我们建立一个测试环境。为此，我们将使用 [testcontainers](/web/20220628113624/https://www.baeldung.com/spring-boot-testcontainers-integration-test) 库并创建以下测试设置:

```java
@ClassRule
public static PostgresqlTestContainer container = PostgresqlTestContainer.getInstance();

@BeforeClass
public static void setup() throws Exception {
    Properties properties = new Properties();
    properties.setProperty("user", container.getUsername());
    properties.setProperty("password", container.getPassword());
    Connection connection = DriverManager.getConnection(container.getJdbcUrl(), properties);
    connection.createStatement().execute("CREATE SCHEMA store");
    connection.createStatement().execute("CREATE TABLE store.product(id SERIAL PRIMARY KEY, name VARCHAR(20))");
    connection.createStatement().execute("INSERT INTO store.product VALUES(1, 'test product')");
}
```

用 [`@ClassRule`，](/web/20220628113624/https://www.baeldung.com/junit-4-rules)我们创建一个 PostgreSQL 数据库容器的实例。接下来，在`setup`方法中，创建到该数据库的连接，并创建所需的对象。

现在，当数据库设置好后，让我们使用 JDBC URL 连接到`store`模式:

```java
@Test
public void settingUpSchemaUsingJdbcURL() throws Exception {
    Properties properties = new Properties();
    properties.setProperty("user", container.getUsername());
    properties.setProperty("password", container.getPassword());
    Connection connection = DriverManager.getConnection(container.getJdbcUrl().concat("&" + "currentSchema=store"), properties);

    ResultSet resultSet = connection.createStatement().executeQuery("SELECT * FROM product");
    resultSet.next();

    assertThat(resultSet.getInt(1), equalTo(1));
    assertThat(resultSet.getString(2), equalTo("test product"));
}
```

**要改变默认模式，我们需要指定`currentSchema`参数。**如果我们输入一个不存在的模式，在`select`查询中会抛出`PSQLException` ，表示数据库对象丢失。

### 3.3. `PGSimpleDataSource`

为了连接到数据库，**我们可以使用来自 PostgreSQL 驱动程序库`PGSimpleDataSource`的`javax.sql.DataSource` 实现。**这个具体的实现支持建立一个模式:

```java
@Test
public void settingUpSchemaUsingPGSimpleDataSource() throws Exception {
    int port = //extracting port from container.getJdbcUrl()
    PGSimpleDataSource ds = new PGSimpleDataSource();
    ds.setServerNames(new String[]{container.getHost()});
    ds.setPortNumbers(new int[]{port});
    ds.setUser(container.getUsername());
    ds.setPassword(container.getPassword());
    ds.setDatabaseName("test");
    ds.setCurrentSchema("store");

    ResultSet resultSet = ds.getConnection().createStatement().executeQuery("SELECT * FROM product");
    resultSet.next();

    assertThat(resultSet.getInt(1), equalTo(1));
    assertThat(resultSet.getString(2), equalTo("test product"));
}
```

当使用`PGSimpleDataSource`时，如果我们不设置模式，驱动程序将使用公共模式作为默认模式。

### `3.4\. @Table Annotation From javax.persistence Package`

如果我们在项目中使用 JPA，**我们可以使用 [`@Table`](/web/20220628113624/https://www.baeldung.com/jpa-entities) 注释在实体级别上指定模式。**这个注释可以保存模式的值，或者默认为空一个`String.` 让我们将`product` 表映射到`Product`实体:

```java
@Entity
@Table(name = "product", schema = "store")
public class Product {

    @Id
    private int id;
    private String name;

    // getters and setters
}
```

为了验证这一行为，我们设置了`[EntityManager](/web/20220628113624/https://www.baeldung.com/hibernate-entitymanager)` 实例来查询`product`表:

```java
@Test
public void settingUpSchemaUsingTableAnnotation(){
    Map<String,String> props = new HashMap<>();
    props.put("hibernate.connection.url", container.getJdbcUrl());
    props.put("hibernate.connection.user", container.getUsername());
    props.put("hibernate.connection.password", container.getPassword());
    EntityManagerFactory emf = Persistence.createEntityManagerFactory("postgresql_schema_unit", props);
    EntityManager entityManager = emf.createEntityManager();

    Product product = entityManager.find(Product.class, 1);

    assertThat(product.getName(), equalTo("test product"));
}
```

正如我们之前在第 3 节中提到的，出于各种原因，最好避免将模式耦合到代码中。正因为如此，这个特性经常被忽略，但是在访问多个模式时，它是很有优势的。

## 4.结论

在本教程中，首先，我们介绍了数据库模式的基本理论。之后，我们描述了使用不同方法和技术设置数据库模式的多种方式。像往常一样，所有的代码示例都可以在 GitHub 上的[处获得。](https://web.archive.org/web/20220628113624/https://github.com/eugenp/tutorials/tree/master/persistence-modules/java-jpa-3)