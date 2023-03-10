# 在 Spring Boot 中以编程方式配置数据源

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-configure-data-source-programmatic>

## 1.概观

使用自以为是的算法扫描并配置一个 [`DataSource`](https://web.archive.org/web/20221205184009/https://docs.oracle.com/en/java/javase/11/docs/api/java.sql/javax/sql/DataSource.html) 。这使得我们可以很容易地得到一个默认的完全配置的 `DataSource`实现。

此外，Spring Boot 自动配置了一个快如闪电的[连接池](/web/20221205184009/https://www.baeldung.com/java-connection-pooling)，或者是 [HikariCP](/web/20221205184009/https://www.baeldung.com/hikaricp) 、 [Apache Tomcat](/web/20221205184009/https://www.baeldung.com/spring-boot-tomcat-connection-pool) 或者是 [Commons DBCP](https://web.archive.org/web/20221205184009/https://commons.apache.org/proper/commons-dbcp/) ，按照这个顺序，这取决于哪个连接池在类路径上。

**虽然 Spring Boot 的自动`DataSource`配置在大多数情况下工作得很好，但有时我们需要更高级别的控制**，所以我们必须建立自己的`DataSource`实现，从而跳过自动配置过程。

在本教程中，我们将学习**如何在 Spring Boot** 中以编程方式配置`DataSource`。

## 延伸阅读:

## [Spring JPA–多个数据库](/web/20221205184009/https://www.baeldung.com/spring-data-jpa-multiple-databases)

How to set up Spring Data JPA to work with multiple, separate databases.[Read more](/web/20221205184009/https://www.baeldung.com/spring-data-jpa-multiple-databases) →

## [为测试配置单独的 Spring 数据源](/web/20221205184009/https://www.baeldung.com/spring-testing-separate-data-source)

A quick, practical tutorial on how to configure a separate data source for testing in a Spring application.[Read more](/web/20221205184009/https://www.baeldung.com/spring-testing-separate-data-source) →

## 2.Maven 依赖项

总体来说，以编程方式创建一个`DataSource`实现是简单明了的。

为了了解如何实现这一点，我们将实现一个简单的存储库层，它将对一些 [JPA](/web/20221205184009/https://www.baeldung.com/the-persistence-layer-with-spring-and-jpa) 实体执行 CRUD 操作。

让我们看看我们的演示项目的依赖项:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <version>2.4.1</version> 
    <scope>runtime</scope> 
</dependency>
```

如上所示，我们将使用内存中的 [H2 数据库](https://web.archive.org/web/20221205184009/https://search.maven.org/search?q=g:org.hsqldb%20AND%20a:hsqldb)实例来测试存储库层。这样做，我们将能够测试我们以编程方式配置的`DataSource,`,而不需要执行昂贵的数据库操作。

另外，我们一定要在 Maven Central 上查看一下`[spring-boot-starter-data-jpa](https://web.archive.org/web/20221205184009/https://search.maven.org/search?q=g:org.springframework.boot%20AND%20a:spring-boot-starter-data-jpa)`的最新版本。

## 3。以编程方式配置一个`DataSource`

现在，如果我们坚持使用 Spring Boot 的自动`DataSource`配置，并在当前状态下运行我们的项目，它会像预期的那样工作。

**Spring Boot 将为我们完成所有的大型基础设施管道工程。**这包括创建一个 H2 `DataSource`实现，它将由 HikariCP、Apache Tomcat 或 Commons DBCP 自动处理，并设置一个内存中的数据库实例。

此外，我们甚至不需要创建一个`application.properties`文件，因为 Spring Boot 也会提供一些默认的数据库设置。

正如我们之前提到的，有时我们需要更高级别的定制，所以我们必须以编程方式配置我们自己的`DataSource`实现。

**最简单的方法是定义一个`DataSource`工厂方法，并把它放在一个用 [`@Configuration`](https://web.archive.org/web/20221205184009/https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/annotation/Configuration.html) 注释**注释的类中:

```java
@Configuration
public class DataSourceConfig {

    @Bean
    public DataSource getDataSource() {
        DataSourceBuilder dataSourceBuilder = DataSourceBuilder.create();
        dataSourceBuilder.driverClassName("org.h2.Driver");
        dataSourceBuilder.url("jdbc:h2:mem:test");
        dataSourceBuilder.username("SA");
        dataSourceBuilder.password("");
        return dataSourceBuilder.build();
    }
}
```

在这种情况下，**我们使用方便的`[DataSourceBuilder](https://web.archive.org/web/20221205184009/https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/jdbc/DataSourceBuilder.html)`类，**一个非流畅版本的 [Joshua Bloch 的构建器模式](https://web.archive.org/web/20221205184009/https://www.pearson.com/us/higher-education/program/Bloch-Effective-Java-3rd-Edition/PGM1763855.html)，**以编程方式创建我们的自定义`DataSource` 对象**。

这种方法非常好，因为构建器使得使用一些公共属性配置`DataSource`变得很容易。它也使用底层连接池。

## 4.用`application.properties`文件外部化`DataSource`配置

当然，也可以部分具体化我们的`DataSource`配置。例如，我们可以在工厂方法中定义一些基本的`DataSource`属性:

```java
@Bean 
public DataSource getDataSource() { 
    DataSourceBuilder dataSourceBuilder = DataSourceBuilder.create(); 
    dataSourceBuilder.username("SA"); 
    dataSourceBuilder.password(""); 
    return dataSourceBuilder.build(); 
}
```

然后我们可以在`application.properties`文件中指定几个额外的:

```java
spring.datasource.url=jdbc:h2:mem:test
spring.datasource.driver-class-name=org.h2.Driver 
```

**在外部源中定义的属性，比如上面的 `application.properties`文件，或者通过用[*@ configuration properties*](/web/20221205184009/https://www.baeldung.com/configuration-properties-in-spring-boot)注释的类，将覆盖在 Java API 中定义的属性。**

很明显，使用这种方法，我们将不再把我们的`DataSource`配置设置保存在一个单独的地方**。**

另一方面，它允许我们很好地将编译时和运行时配置设置彼此分开。

这真的很好，因为它允许我们轻松地设置配置绑定点。这样我们可以包含来自其他来源的不同的`DataSource`设置，而不必重构我们的 bean factory 方法。

## 5。测试`DataSource`配置

测试我们的定制`DataSource`配置非常简单。整个过程归结为创建一个 [JPA](/web/20221205184009/https://www.baeldung.com/the-persistence-layer-with-spring-and-jpa) 实体，定义一个基本的存储库接口，并测试存储库层。

### 5.1。创建 JPA 实体

让我们从定义示例 JPA 实体类开始，它将为用户建模:

```java
@Entity
@Table(name = "users")
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private long id;
    private String name;
    private String email;

    // standard constructors / setters / getters / toString

}
```

### 5.2。一个简单的存储库层

接下来，我们需要实现一个基本的存储库层，它允许我们对上面定义的`User`实体类的实例执行 CRUD 操作。

由于我们使用的是 [Spring Data JPA](/web/20221205184009/https://www.baeldung.com/the-persistence-layer-with-spring-data-jpa) ，我们不必从头开始创建我们自己的 [DAO](/web/20221205184009/https://www.baeldung.com/java-dao-pattern) 实现。我们只需扩展 [`CrudRepository`](https://web.archive.org/web/20221205184009/https://docs.spring.io/spring-data/commons/docs/current/api/org/springframework/data/repository/CrudRepository.html) 接口就可以得到一个可用的存储库实现:

```java
@Repository
public interface UserRepository extends CrudRepository<User, Long> {} 
```

### 5.3。测试存储库层

最后，我们需要检查我们以编程方式配置的`DataSource`是否真的在工作。我们可以通过集成测试轻松实现这一点:

```java
@RunWith(SpringRunner.class)
@DataJpaTest
public class UserRepositoryIntegrationTest {

    @Autowired
    private UserRepository userRepository;

    @Test
    public void whenCalledSave_thenCorrectNumberOfUsers() {
        userRepository.save(new User("Bob", "[[email protected]](/web/20221205184009/https://www.baeldung.com/cdn-cgi/l/email-protection)"));
        List<User> users = (List<User>) userRepository.findAll();

        assertThat(users.size()).isEqualTo(1);
    }    
}
```

`UserRepositoryIntegrationTest`类非常简单明了。它只是练习存储库接口的两个 CRUD 方法来持久化和查找实体。

**注意，不管我们是决定以编程方式配置我们的`DataSource`实现，还是将它分成一个 Java 配置方法和 `application.properties`文件，我们应该总是得到一个工作的数据库连接**。

### 5.4。运行示例应用程序

最后，我们可以使用标准的`main()`方法运行我们的演示应用程序:

```java
@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

    @Bean
    public CommandLineRunner run(UserRepository userRepository) throws Exception {
        return (String[] args) -> {
            User user1 = new User("John", "[[email protected]](/web/20221205184009/https://www.baeldung.com/cdn-cgi/l/email-protection)");
            User user2 = new User("Julie", "[[email protected]](/web/20221205184009/https://www.baeldung.com/cdn-cgi/l/email-protection)");
            userRepository.save(user1);
            userRepository.save(user2);
            userRepository.findAll().forEach(user -> System.out.println(user);
        };
    }
} 
```

我们已经测试了存储库层，所以我们确信我们的`DataSource`已经成功配置。因此，如果我们运行示例应用程序，我们应该在控制台输出中看到存储在数据库中的`User`实体的列表。

## 6。结论

在本文中，**我们学习了如何在 Spring Boot** 中以编程方式配置一个`DataSource`实现。

像往常一样，本文中显示的所有代码示例都可以在 GitHub 上的[处获得。](https://web.archive.org/web/20221205184009/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-boot-persistence)