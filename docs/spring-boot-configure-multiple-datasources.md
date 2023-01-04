# 在 Spring Boot 配置和使用多个数据源

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-configure-multiple-datasources>

## 1.概观

Spring Boot 应用程序的典型场景是将数据存储在单个关系数据库中。但是我们有时需要访问多个数据库。

在本教程中，我们将学习如何通过 Spring Boot 配置和使用多个数据源。

要了解如何处理单个数据源，请查看我们的[Spring Data JPA](/web/20221102081757/https://www.baeldung.com/the-persistence-layer-with-spring-data-jpa)简介。

## 2.默认行为

让我们记住在`application.yml`中在 Spring Boot 声明一个数据源是什么样子的:

```
spring:
  datasource:
    url: ...
    username: ...
    password: ...
    driverClassname: ...
```

在内部，Spring 将这些设置映射到一个`org.springframework.boot.autoconfigure.jdbc.DataSourceProperties`实例。

让我们来看看实现情况:

```
@ConfigurationProperties(prefix = "spring.datasource")
public class DataSourceProperties implements BeanClassLoaderAware, InitializingBean {

    // ...

    /**
     * Fully qualified name of the JDBC driver. Auto-detected based on the URL by default.
     */
    private String driverClassName;

    /**
     * JDBC URL of the database.
     */
    private String url;

    /**
     * Login username of the database.
     */
    private String username;

    /**
     * Login password of the database.
     */
    private String password;

    // ...

}
```

我们应该指出将配置属性自动映射到 Java 对象的`@ConfigurationProperties`注释。

## 3.扩展默认值

因此，为了使用多个数据源，我们需要在 Spring 的应用程序上下文中声明多个具有不同映射的 beans。

我们可以通过使用配置类来实现这一点:

```
@Configuration
public class TodoDatasourceConfiguration {

    @Bean
    @ConfigurationProperties("spring.datasource.todos")
    public DataSourceProperties todosDataSourceProperties() {
        return new DataSourceProperties();
    }

    @Bean
    @ConfigurationProperties("spring.datasource.topics")
    public DataSourceProperties topicsDataSourceProperties() {
        return new DataSourceProperties();
    }

}
```

数据源的配置必须如下所示:

```
spring:
  datasource:
    todos:
      url: ...
      username: ...
      password: ...
      driverClassName: ...
    topics:
      url: ...
      username: ...
      password: ...
      driverClassName: ...
```

然后我们可以通过使用`DataSourceProperties`对象来创建数据源:

```
@Bean
public DataSource todosDataSource() {
    return todosDataSourceProperties()
      .initializeDataSourceBuilder()
      .build();
}

@Bean
public DataSource topicsDataSource() {
    return topicsDataSourceProperties()
      .initializeDataSourceBuilder()
      .build();
}
```

## 4.JDBC 春季数据

当使用 Spring 数据 JDBC 时，我们还需要为每个`DataSource`配置一个`JdbcTemplate`实例:

```
@Bean
public JdbcTemplate todosJdbcTemplate(@Qualifier("todosDataSource") DataSource dataSource) {
    return new JdbcTemplate(dataSource);
}

@Bean
public JdbcTemplate topicsJdbcTemplate(@Qualifier("topicsDataSource") DataSource dataSource) {
    return new JdbcTemplate(dataSource);
} 
```

然后我们也可以通过指定一个`@Qualifier`来使用它们:

```
@Autowired
@Qualifier("topicsJdbcTemplate")
JdbcTemplate jdbcTemplate;
```

## 5.春季数据 JPA

当使用 Spring Data JPA 时，我们希望使用如下的存储库，其中`Todo`是实体:

```
public interface TodoRepository extends JpaRepository<Todo, Long> {}
```

因此，我们需要为每个数据源声明`EntityManager`工厂:

```
@Configuration
@EnableTransactionManagement
@EnableJpaRepositories(
  basePackageClasses = Todo.class,
  entityManagerFactoryRef = "todosEntityManagerFactory",
  transactionManagerRef = "todosTransactionManager"
)
public class TodoJpaConfiguration {

    @Bean
    public LocalContainerEntityManagerFactoryBean todosEntityManagerFactory(
      Qualifier("todosDataSource") DataSource dataSource,
      EntityManagerFactoryBuilder builder) {
        return builder
          .dataSource(todosDataSource())
          .packages(Todo.class)
          .build();
    }

    @Bean
    public PlatformTransactionManager todosTransactionManager(
      @Qualifier("todosEntityManagerFactory") LocalContainerEntityManagerFactoryBean todosEntityManagerFactory) {
        return new JpaTransactionManager(Objects.requireNonNull(todosEntityManagerFactory.getObject()));
    }

}
```

让我们来看看我们应该注意的一些限制。

我们需要分割包，以允许每个数据源有一个`@EnableJpaRepositories`。

不幸的是，要注入`EntityManagerFactoryBuilder`，我们需要将数据源之一声明为`@Primary`。

这是因为`EntityManagerFactoryBuilder`是在`org.springframework.boot.autoconfigure.orm.jpa.JpaBaseConfiguration`中声明的，这个类需要注入单个数据源。通常，框架的某些部分可能不期望配置多个数据源。

## 6.配置光连接池

如果我们想配置[光](/web/20221102081757/https://www.baeldung.com/spring-boot-hikari)，我们只需要给数据源定义添加一个`@ConfigurationProperties`:

```
@Bean
@ConfigurationProperties("spring.datasource.todos.hikari")
public DataSource todosDataSource() {
    return todosDataSourceProperties()
      .initializeDataSourceBuilder()
      .build();
}
```

然后我们可以将下面几行插入到`application.properties`文件中:

```
spring.datasource.todos.hikari.connectionTimeout=30000 
spring.datasource.todos.hikari.idleTimeout=600000 
spring.datasource.todos.hikari.maxLifetime=1800000 
```

## 7.结论

在本文中，我们学习了如何使用 Spring Boot 配置多个数据源。

我们看到，我们需要一些配置，偏离标准时可能会有陷阱，但最终是可能的。

和往常一样，GitHub 上的所有代码[都是可用的。](https://web.archive.org/web/20221102081757/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-data-jdbc)