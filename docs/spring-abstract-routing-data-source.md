# Spring 抽象路由数据源指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-abstract-routing-data-source>

## 1。概述

在这篇简短的文章中，我们将 Spring 的`AbstractRoutingDatasource`看作是**基于当前上下文**动态确定实际`DataSource`的一种方式。

结果，我们会看到我们可以将`DataSource`查找逻辑从数据访问代码中去掉。

## 2。Maven 依赖关系

让我们从在`pom.xml`中将`spring-context, spring-jdbc, spring-test,` 和`h2`声明为依赖关系开始:

```java
<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>4.3.8.RELEASE</version>
    </dependency>

    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-jdbc</artifactId>
        <version>4.3.8.RELEASE</version>
    </dependency>

    <dependency> 
        <groupId>org.springframework</groupId> 
        <artifactId>spring-test</artifactId>
        <version>4.3.8.RELEASE</version>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>com.h2database</groupId>
        <artifactId>h2</artifactId>
        <version>1.4.195</version>
        <scope>test</scope>
    </dependency>
</dependencies>
```

依赖关系的最新版本可以在[这里](https://web.archive.org/web/20220927223231/https://search.maven.org/classic/#search%7Cga%7C1%7C(g%3A%22org.springframework%22%20AND%20(a%3A%22spring-context%22%20OR%20a%3A%22spring-jdbc%22%20OR%20a%3A%22spring-test%22))%20OR%20(g%3A%22com.h2database%22%20AND%20a%3A%22h2%22))找到。

如果您使用 Spring Boot，我们可以使用 Spring 数据和测试的启动器:

```java
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>
    <dependency> 
        <groupId>org.springframework.boot</groupId> 
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>com.h2database</groupId>
        <artifactId>h2</artifactId>
        <version>1.4.195</version>
        <scope>test</scope>
    </dependency>
</dependencies>
```

## 3。数据源上下文

`AbstractRoutingDatasource`需要信息来知道要路由到哪个实际的`DataSource`。这种信息通常被称为`Context.`

与`AbstractRoutingDatasource` 一起使用的`Context`可以是任何一个`Object,` ，用`enum`来定义它们。在我们的例子中，我们将使用`ClientDatabase`的概念作为以下实现的上下文:

```java
public enum ClientDatabase {
    CLIENT_A, CLIENT_B
}
```

值得注意的是，在实践中，上下文可以是对所讨论的领域有意义的任何内容。

例如，另一个常见的用例涉及使用`Environment`的概念来定义上下文。在这样的场景中，上下文可以是包含`PRODUCTION`、`DEVELOPMENT`和`TESTING`的枚举。

## 4。上下文持有者

上下文容器实现是一个容器，它将当前上下文存储为一个 [`ThreadLocal`](/web/20220927223231/https://www.baeldung.com/java-threadlocal) 引用。

除了保存引用之外，它还应该包含设置、获取和清除引用的静态方法。`AbstractRoutingDatasource` 将向 ContextHolder 查询上下文，然后使用上下文查找实际的`DataSource`。

在这里使用`ThreadLocal`非常重要，这样上下文就可以绑定到当前正在执行的线程。

必须采用这种方法，以便当数据访问逻辑跨越多个数据源并使用事务时，行为是可靠的:

```java
public class ClientDatabaseContextHolder {

    private static ThreadLocal<ClientDatabase> CONTEXT
      = new ThreadLocal<>();

    public static void set(ClientDatabase clientDatabase) {
        Assert.notNull(clientDatabase, "clientDatabase cannot be null");
        CONTEXT.set(clientDatabase);
    }

    public static ClientDatabase getClientDatabase() {
        return CONTEXT.get();
    }

    public static void clear() {
        CONTEXT.remove();
    }
}
```

## 5。数据源路由器

我们定义我们的`ClientDataSourceRouter`来拉伸弹簧`AbstractRoutingDataSource`。我们实现必要的`determineCurrentLookupKey`方法来查询我们的`ClientDatabaseContextHolder`并返回适当的键。

`AbstractRoutingDataSource`实现为我们处理剩下的工作，并透明地返回适当的`DataSource:`

```java
public class ClientDataSourceRouter
  extends AbstractRoutingDataSource {

    @Override
    protected Object determineCurrentLookupKey() {
        return ClientDatabaseContextHolder.getClientDatabase();
    }
}
```

## 6。配置

我们需要一个`Map`到`DataSource`对象的上下文来配置我们的`AbstractRoutingDataSource`。如果没有设置上下文，我们还可以指定一个默认的`DataSource`来使用。

我们使用的`DataSource`可以来自任何地方，但通常要么在运行时创建，要么使用 JNDI 查找:

```java
@Configuration
public class RoutingTestConfiguration {

    @Bean
    public ClientService clientService() {
        return new ClientService(new ClientDao(clientDatasource()));
    }

    @Bean
    public DataSource clientDatasource() {
        Map<Object, Object> targetDataSources = new HashMap<>();
        DataSource clientADatasource = clientADatasource();
        DataSource clientBDatasource = clientBDatasource();
        targetDataSources.put(ClientDatabase.CLIENT_A, 
          clientADatasource);
        targetDataSources.put(ClientDatabase.CLIENT_B, 
          clientBDatasource);

        ClientDataSourceRouter clientRoutingDatasource 
          = new ClientDataSourceRouter();
        clientRoutingDatasource.setTargetDataSources(targetDataSources);
        clientRoutingDatasource.setDefaultTargetDataSource(clientADatasource);
        return clientRoutingDatasource;
    }

    // ...
}
```

使用 Spring Boot 时，可以在`application.properties`文件中配置`DataSources` (即**ClientA**&**ClientB**):

```java
#database details for CLIENT_A
client-a.datasource.name=CLIENT_A
client-a.datasource.script=SOME_SCRIPT.sql

#database details for CLIENT_B
client-b.datasource.name=CLIENT_B
client-b.datasource.script=SOME_SCRIPT.sql
```

然后您可以创建 POJOs 来保存您的`DataSources`的属性:

```java
@Component
@ConfigurationProperties(prefix = "client-a.datasource")
public class ClientADetails {

    private String name;
    private String script;

    // Getters & Setters
}
```

并使用它们来构建您的数据源 beans:

```java
@Autowired
private ClientADetails clientADetails;
@Autowired
private ClientBDetails clientBDetails;

private DataSource clientADatasource() {
EmbeddedDatabaseBuilder dbBuilder = new EmbeddedDatabaseBuilder();
return dbBuilder.setType(EmbeddedDatabaseType.H2)
.setName(clientADetails.getName())
.addScript(clientADetails.getScript())
.build();
}

private DataSource clientBDatasource() {
EmbeddedDatabaseBuilder dbBuilder = new EmbeddedDatabaseBuilder();
return dbBuilder.setType(EmbeddedDatabaseType.H2)
.setName(clientBDetails.getName())
.addScript(clientBDetails.getScript())
.build();
}
```

## 7 .**。用途**

当使用我们的`AbstractRoutingDataSource`时，我们首先设置上下文，然后执行我们的操作。我们使用一个服务层，它将上下文作为一个参数，在委托给数据访问代码之前设置它，并在调用后清除上下文。

作为手动清除服务方法中的上下文的替代方法，清除逻辑可以由 AOP 点切割来处理。

记住上下文是`thread bound`很重要，尤其是如果数据访问逻辑将跨越多个数据源和事务:

```java
public class ClientService {

    private ClientDao clientDao;

    // standard constructors

    public String getClientName(ClientDatabase clientDb) {
        ClientDatabaseContextHolder.set(clientDb);
        String clientName = this.clientDao.getClientName();
        ClientDatabaseContextHolder.clear();
        return clientName;
    }
}
```

## 8。结论

在本教程中，我们看了如何使用弹簧`AbstractRoutingDataSource`的例子。我们使用`Client`的概念实现了一个解决方案，其中每个客户端都有自己的`DataSource`。

和往常一样，在 GitHub 上可以找到例子[。](https://web.archive.org/web/20220927223231/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-jpa)