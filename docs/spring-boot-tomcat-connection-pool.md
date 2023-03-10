# 在 Spring Boot 配置 Tomcat 连接池

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-tomcat-connection-pool>

## 1。概述

[Spring Boot](https://web.archive.org/web/20220626090456/https://spring.io/projects/spring-boot) 是一个自以为是但功能强大的抽象层，位于一个普通的 Spring 平台之上，这使得开发独立和 web 应用程序变得轻而易举。Spring Boot 提供了一些方便的“入门”依赖项，旨在以最小的内存占用运行和测试 Java 应用程序。

**这些启动器依赖项的一个关键组件是`[spring-boot-starter-data-jpa](https://web.archive.org/web/20220626090456/https://search.maven.org/search?q=a:spring-boot-starter-data-jpa%20AND%20%20g:org.springframework.boot)`** 。这允许我们使用 [JPA](https://web.archive.org/web/20220626090456/https://en.wikipedia.org/wiki/Java_Persistence_API) 并通过使用一些流行的 JDBC 连接池实现来处理生产数据库，例如 [HikariCP](https://web.archive.org/web/20220626090456/https://github.com/brettwooldridge/HikariCP) 和 [Tomcat JDBC 连接池](https://web.archive.org/web/20220626090456/https://tomcat.apache.org/tomcat-7.0-doc/jdbc-pool.html)。

在本教程中，**我们将学习如何在 Spring Boot** 中配置 Tomcat 连接池。

## 2。美芬依赖

由于其卓越的性能和企业级特性，Spring Boot 将 HikariCP 用作默认连接池。

**下面是 Spring Boot 如何自动配置连接池数据源:**

1.  Spring Boot 将在类路径中查找 HikariCP，并在默认情况下使用它
2.  如果在类路径中找不到 HikariCP，那么 Spring Boot 将获得 Tomcat JDBC 连接池，如果它可用的话
3.  如果这两个选项都不可用，Spring Boot 将选择 [Apache Commons DBCP2](https://web.archive.org/web/20220626090456/https://commons.apache.org/proper/commons-dbcp/) ，如果可用的话

为了配置一个 Tomcat JDBC 连接池而不是默认的 HikariCP，我们将**从`spring-boot-starter-data-jpa`依赖关系中排除`HikariCP`并将`[tomcat-jdbc](https://web.archive.org/web/20220626090456/https://search.maven.org/artifact/org.apache.tomcat/tomcat-jdbc/9.0.11/jar) ` Maven 依赖关系**添加到我们的`pom.xml`中:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
    <exclusions>
        <exclusion>
            <groupId>com.zaxxer</groupId>
            <artifactId>HikariCP</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<dependency>
    <groupId>org.apache.tomcat</groupId>
    <artifactId>tomcat-jdbc</artifactId>
    <version>9.0.10</version>
</dependency>
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <version>1.4.197</version>
    <scope>runtime</scope>
</dependency>
```

这种简单的方法允许我们使用 Tomcat 连接池获得 Spring Boot，而不必编写一个`[@Configuration](https://web.archive.org/web/20220626090456/https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/annotation/Configuration.html)`类，并以编程方式定义一个 [`DataSource`](https://web.archive.org/web/20220626090456/https://docs.oracle.com/en/java/javase/11/docs/api/java.sql/javax/sql/DataSource.html) bean。

**同样值得注意的是，在本例中，我们使用的是 H2 内存数据库**。 **Spring Boot 将为我们自动配置 H2，而无需指定数据库 URL、用户和密码**。

我们只需要在`“pom.xml”` 文件中包含相应的依赖关系，Spring Boot 会为我们完成剩下的工作。

或者，**可以跳过 Spring Boot 使用的连接池扫描算法，使用`“spring.datasource.type”`属性在`“application.properties” file` `,`中显式指定连接池数据源**:

```java
spring.datasource.type=org.apache.tomcat.jdbc.pool.DataSource
// other spring datasource properties
```

## 3。用“`application.properties`”文件调整连接池

一旦我们在 Spring Boot 成功配置了一个 Tomcat 连接池，**我们很可能想要设置一些额外的属性，以优化它的性能并满足一些特定的需求**。

我们可以在`“application.properties”`文件中这样做:

```java
spring.datasource.tomcat.initial-size=15
spring.datasource.tomcat.max-wait=20000
spring.datasource.tomcat.max-active=50
spring.datasource.tomcat.max-idle=15
spring.datasource.tomcat.min-idle=8
spring.datasource.tomcat.default-auto-commit=true 
```

请注意，我们已经配置了一些附加的连接池属性，比如池的初始大小，以及空闲连接的最大和最小数量。

我们还可以指定一些特定于 Hibernate 的属性:

```java
# Hibernate specific properties
spring.jpa.show-sql=false
spring.jpa.hibernate.ddl-auto=update
spring.jpa.hibernate.naming-strategy=org.hibernate.cfg.ImprovedNamingStrategy
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.H2Dialect
spring.jpa.properties.hibernate.id.new_generator_mappings=false 
```

## 4。测试连接池

让我们编写一个简单的集成测试来检查 Spring Boot 是否正确配置了连接池:

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class SpringBootTomcatConnectionPoolIntegrationTest {

    @Autowired
    private DataSource dataSource;

    @Test
    public void givenTomcatConnectionPoolInstance_whenCheckedPoolClassName_thenCorrect() {
        assertThat(dataSource.getClass().getName())
          .isEqualTo("org.apache.tomcat.jdbc.pool.DataSource");
    }
}
```

## 5。示例命令行应用程序

已经设置好所有连接池，让我们构建一个简单的命令行应用程序。

通过这样做，我们可以看到如何使用强大的 [DAO](/web/20220626090456/https://www.baeldung.com/java-dao-pattern) 层在 H2 数据库上执行一些 CRUD 操作，该层由 [Spring Data JPA](https://web.archive.org/web/20220626090456/https://docs.spring.io/spring-data/jpa/docs/current/reference/html/) (以及 Spring Boot)提供。

关于如何开始使用 Spring Data JPA 的详细指南，请查看本文。

### 5.1。`Customer`实体类

让我们首先定义一个简单的`Customer`实体类:

```java
@Entity
@Table(name = "customers")
public class Customer {

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private long id;
    @Column(name = "first_name")
    private String firstName;

    // standard constructors / getters / setters / toString
}
```

### 5.2.`CustomerRepository`界面

在这种情况下，我们只想对几个`Customer`实体**执行 CRUD 操作。**此外，我们需要获取与给定姓氏匹配的所有客户。

所以，**我们所要做的就是扩展 Spring Data JPA 的`[CrudRepository](https://web.archive.org/web/20220626090456/https://docs.spring.io/spring-data/commons/docs/current/api/org/springframework/data/repository/CrudRepository.html)`接口，并定义一个定制的方法**:

```java
public interface CustomerRepository extends CrudRepository<Customer, Long> {
    List<Customer> findByLastName(String lastName);
}
```

现在我们可以很容易地通过一个实体的姓来获取这个实体。

### 5.3。`CommandLineRunner`实施

最后，我们至少需要在数据库中持久化一些`Customer`实体，并**验证我们的 Tomcat 连接池实际上正在工作**。

让我们创建一个 Spring Boot 的`CommandLineRunner`接口的实现。Spring Boot 将在启动应用程序之前引导实施:

```java
public class CommandLineCrudRunner implements CommandLineRunner {

    private static final Logger logger = LoggerFactory.getLogger(CommandLineCrudRunner.class);

    @Autowired
    private final CustomerRepository repository;

    public void run(String... args) throws Exception {
        repository.save(new Customer("John", "Doe"));
        repository.save(new Customer("Jennifer", "Wilson"));

        logger.info("Customers found with findAll():");
        repository.findAll().forEach(c -> logger.info(c.toString()));

        logger.info("Customer found with findById(1L):");
        Customer customer = repository.findById(1L)
          .orElseGet(() -> new Customer("Non-existing customer", ""));
        logger.info(customer.toString());

        logger.info("Customer found with findByLastName('Wilson'):");
        repository.findByLastName("Wilson").forEach(c -> {
            logger.info(c.toString());
        });
    }
}
```

简而言之，`CommandLineCrudRunner`类首先在数据库中保存几个`Customer`实体。接下来，它使用`findById()`方法获取第一个。最后，它用`findByLastName()`方法检索一个客户。

### 5.4。运行 Spring Boot 应用程序

当然，我们需要做的最后一件事就是运行示例应用程序。然后我们可以看到 Spring Boot/Tomcat 连接池的运行:

```java
@SpringBootApplication
public class SpringBootConsoleApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringBootConsoleApplication.class);
    }
}
```

## 6。结论

在本教程中，我们学习了如何在 Spring Boot 配置和使用 Tomcat 连接池。此外，我们开发了一个基本的命令行应用程序来展示使用 Spring Boot、Tomcat 连接池和 H2 数据库是多么容易。

像往常一样，本教程中显示的所有代码示例都可以从 GitHub 上的[处获得。](https://web.archive.org/web/20220626090456/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-boot-persistence-2)