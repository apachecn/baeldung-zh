# Hibernate 5 中的多租户指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/hibernate-5-multitenancy>

## 1。简介

多承租允许多个客户机或租户使用单个资源，或者在本文的上下文中，使用单个数据库实例。目的是**从共享数据库**中分离出每个租户需要的信息。

在本教程中，我们将介绍在 Hibernate 5 中配置多租户的各种方法。

## 2。Maven 依赖关系

我们需要在`pom.xml`文件中包含[和`hibernate-core`依赖关系](https://web.archive.org/web/20220701014001/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.hibernate%22%20AND%20a%3A%22hibernate-core%22):

```java
<dependency>
   <groupId>org.hibernate</groupId>
   <artifactId>hibernate-core</artifactId>
   <version>5.2.12.Final</version>
</dependency>
```

为了测试，我们将使用一个 H2 内存数据库，所以让我们也将[这个依赖关系](https://web.archive.org/web/20220701014001/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22com.h2database%22%20AND%20a%3A%22h2%22)添加到`pom.xml`文件中:

```java
<dependency>
   <groupId>com.h2database</groupId>
   <artifactId>h2</artifactId>
   <version>1.4.196</version>
</dependency>
```

## 3。了解 Hibernate 中的多租户功能

正如官方的 [Hibernate 用户指南](https://web.archive.org/web/20220701014001/https://docs.jboss.org/hibernate/orm/5.2/userguide/html_single/Hibernate_User_Guide.html#multitenacy)中提到的，Hibernate 中有三种多租户方法:

*   同一个物理数据库实例中的每个租户一个模式
*   每个租户一个单独的物理数据库实例
*   `Partitioned (Discriminator) Data –`每个租户的数据由一个鉴别器值划分

Hibernate 还不支持**分区(鉴别器)数据方法。**跟进[这个 JIRA 问题](https://web.archive.org/web/20220701014001/https://hibernate.atlassian.net/browse/HHH-6054)的未来进展。

和往常一样，Hibernate 抽象了每种方法实现的复杂性。

我们所需要的就是**提供这两个接口**的实现:

*   [`MultiTenantConnectionProvider`](https://web.archive.org/web/20220701014001/https://docs.jboss.org/hibernate/orm/current/javadocs/org/hibernate/engine/jdbc/connections/spi/MultiTenantConnectionProvider.html)–为每个租户提供连接

*   [`CurrentTenantIdentifierResolver`](https://web.archive.org/web/20220701014001/https://docs.jboss.org/hibernate/orm/current/javadocs/org/hibernate/context/spi/CurrentTenantIdentifierResolver.html)–解析要使用的租户标识符

在浏览数据库和模式方法示例之前，让我们更详细地了解每个概念。

### 3.1。 `**MultiTenantConnectionProvider**`

基本上，这个接口为具体的租户标识符提供了一个数据库连接。

让我们看看它的两个主要方法:

```java
interface MultiTenantConnectionProvider extends Service, Wrapped {
    Connection getAnyConnection() throws SQLException;

    Connection getConnection(String tenantIdentifier) throws SQLException;
     // ...
}
```

如果 Hibernate 无法解析要使用的租户标识符，它将使用方法`getAnyConnection`来获取连接。否则，它将使用方法`getConnection`。

**根据我们如何定义数据库连接，Hibernate 提供了该接口的两种实现:**

*   使用来自 Java 的[数据源](https://web.archive.org/web/20220701014001/https://docs.oracle.com/en/java/javase/11/docs/api/java.sql/javax/sql/DataSource.html)接口——我们将使用`DataSourceBasedMultiTenantConnectionProviderImpl`实现
*   使用 Hibernate 的`ConnectionProvider`接口——我们将使用`AbstractMultiTenantConnectionProvider`实现

### 3.2。 `**CurrentTenantIdentifierResolver**`

有许多可能的方法来解析租户标识符。例如，我们的实现可以使用配置文件中定义的一个租户标识符。

另一种方法是使用 path 参数中的租户标识符。

让我们看看这个界面:

```java
public interface CurrentTenantIdentifierResolver {

    String resolveCurrentTenantIdentifier();

    boolean validateExistingCurrentSessions();
}
```

Hibernate 调用方法`resolveCurrentTenantIdentifier`来获取租户标识符。如果我们希望 Hibernate 验证所有现有的会话都属于同一个租户标识符，那么方法`validateExistingCurrentSessions`应该返回 true。

## 4。模式方法

在这个策略中，我们将在同一个物理数据库实例中使用不同的模式或用户。**当我们需要应用程序的最佳性能，并且可以牺牲特殊的数据库功能(如每租户备份)时，应该使用这种方法。**

此外，我们将模拟`CurrentTenantIdentifierResolver`接口，在测试期间提供一个租户标识符作为我们的选择:

```java
public abstract class MultitenancyIntegrationTest {

    @Mock
    private CurrentTenantIdentifierResolver currentTenantIdentifierResolver;

    private SessionFactory sessionFactory;

    @Before
    public void setup() throws IOException {
        MockitoAnnotations.initMocks(this);

        when(currentTenantIdentifierResolver.validateExistingCurrentSessions())
          .thenReturn(false);

        Properties properties = getHibernateProperties();
        properties.put(
          AvailableSettings.MULTI_TENANT_IDENTIFIER_RESOLVER, 
          currentTenantIdentifierResolver);

        sessionFactory = buildSessionFactory(properties);

        initTenant(TenantIdNames.MYDB1);
        initTenant(TenantIdNames.MYDB2);
    }

    protected void initTenant(String tenantId) {
        when(currentTenantIdentifierResolver
         .resolveCurrentTenantIdentifier())
           .thenReturn(tenantId);
        createCarTable();
    }
}
```

我们实现的`MultiTenantConnectionProvider`接口将**设置每次请求连接时使用的模式**:

```java
class SchemaMultiTenantConnectionProvider
  extends AbstractMultiTenantConnectionProvider {

    private ConnectionProvider connectionProvider;

    public SchemaMultiTenantConnectionProvider() throws IOException {
        this.connectionProvider = initConnectionProvider();
    }

    @Override
    protected ConnectionProvider getAnyConnectionProvider() {
        return connectionProvider;
    }

    @Override
    protected ConnectionProvider selectConnectionProvider(
      String tenantIdentifier) {

        return connectionProvider;
    }

    @Override
    public Connection getConnection(String tenantIdentifier)
      throws SQLException {

        Connection connection = super.getConnection(tenantIdentifier);
        connection.createStatement()
          .execute(String.format("SET SCHEMA %s;", tenantIdentifier));
        return connection;
    }

    private ConnectionProvider initConnectionProvider() throws IOException {
        Properties properties = new Properties();
        properties.load(getClass()
          .getResourceAsStream("/hibernate.properties"));

        DriverManagerConnectionProviderImpl connectionProvider 
          = new DriverManagerConnectionProviderImpl();
        connectionProvider.configure(properties);
        return connectionProvider;
    }
}
```

因此，我们将使用一个具有两种模式的内存中 H2 数据库——每个租户一个模式。

让我们配置 `hibernate.properties` 来使用模式多承租模式和我们的`MultiTenantConnectionProvider` 接口`:`的实现

```java
hibernate.connection.url=jdbc:h2:mem:mydb1;DB_CLOSE_DELAY=-1;\
  INIT=CREATE SCHEMA IF NOT EXISTS MYDB1\\;CREATE SCHEMA IF NOT EXISTS MYDB2\\;
hibernate.multiTenancy=SCHEMA
hibernate.multi_tenant_connection_provider=\
  com.baeldung.hibernate.multitenancy.schema.SchemaMultiTenantConnectionProvider
```

出于测试的目的，我们已经配置了`hibernate.connection.url`属性来创建两个模式。对于真正的应用程序来说，这不是必须的，因为模式应该已经存在了。

对于我们的测试，我们将在租户`myDb1\.` 中添加一个`Car`条目，我们将验证该条目是否存储在我们的数据库中，并且不在租户`myDb2`中:

```java
@Test
void whenAddingEntries_thenOnlyAddedToConcreteDatabase() {
    whenCurrentTenantIs(TenantIdNames.MYDB1);
    whenAddCar("myCar");
    thenCarFound("myCar");
    whenCurrentTenantIs(TenantIdNames.MYDB2);
    thenCarNotFound("myCar");
}
```

正如我们在测试中看到的，我们在调用`whenCurrentTenantIs`方法时更改了租户。

## 5。数据库方法

数据库多租户方法为每个租户使用**个不同的物理数据库实例。由于每个租户都是完全隔离的，**当我们需要特殊的数据库功能(如每个租户的备份)而不是最佳性能时，我们应该选择这种策略。****

对于数据库方法，我们将使用与上面相同的`MultitenancyIntegrationTest`类和`CurrentTenantIdentifierResolver` 接口。

对于`MultiTenantConnectionProvider`接口，我们将使用一个`Map`集合来获取每个租户的`ConnectionProvider`标识符:

```java
class MapMultiTenantConnectionProvider
  extends AbstractMultiTenantConnectionProvider {

    private Map<String, ConnectionProvider> connectionProviderMap
     = new HashMap<>();

    public MapMultiTenantConnectionProvider() throws IOException {
        initConnectionProviderForTenant(TenantIdNames.MYDB1);
        initConnectionProviderForTenant(TenantIdNames.MYDB2);
    }

    @Override
    protected ConnectionProvider getAnyConnectionProvider() {
        return connectionProviderMap.values()
          .iterator()
          .next();
    }

    @Override
    protected ConnectionProvider selectConnectionProvider(
      String tenantIdentifier) {

        return connectionProviderMap.get(tenantIdentifier);
    }

    private void initConnectionProviderForTenant(String tenantId)
     throws IOException {
        Properties properties = new Properties();
        properties.load(getClass().getResourceAsStream(
          String.format("/hibernate-database-%s.properties", tenantId)));
        DriverManagerConnectionProviderImpl connectionProvider 
          = new DriverManagerConnectionProviderImpl();
        connectionProvider.configure(properties);
        this.connectionProviderMap.put(tenantId, connectionProvider);
    }
}
```

每个`ConnectionProvider`通过配置文件`hibernate-database-<tenant identifier>.properties,` 填充，该文件包含所有连接细节:

```java
hibernate.connection.driver_class=org.h2.Driver
hibernate.connection.url=jdbc:h2:mem:<Tenant Identifier>;DB_CLOSE_DELAY=-1
hibernate.connection.username=sa
hibernate.dialect=org.hibernate.dialect.H2Dialect
```

最后，让我们再次更新`hibernate.properties`以使用数据库多租户模式和我们的`MultiTenantConnectionProvider` 接口实现:

```java
hibernate.multiTenancy=DATABASE
hibernate.multi_tenant_connection_provider=\
  com.baeldung.hibernate.multitenancy.database.MapMultiTenantConnectionProvider
```

如果我们运行与模式方法完全相同的测试，测试将再次通过。

## 6。结论

本文介绍了 Hibernate 5 对使用独立数据库和独立模式方法的多承租的支持。我们提供了非常简单的实现和示例来探究这两种策略之间的差异。

本文中使用的完整代码示例可以在我们的 [GitHub 项目](https://web.archive.org/web/20220701014001/https://github.com/eugenp/tutorials/tree/master/persistence-modules/hibernate-enterprise)中获得。