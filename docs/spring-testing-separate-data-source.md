# 为测试配置单独的 Spring 数据源

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-testing-separate-data-source>

## 1。概述

当测试依赖于持久层(比如 JPA)的 Spring 应用程序时，我们可能希望设置一个测试数据源，使用一个更小、更快的数据库，这个数据库不同于我们用来运行应用程序的数据库，以便使运行我们的测试更加容易。

在 Spring 中配置数据源需要定义一个类型为`DataSource.` 的 bean，我们可以手动完成，或者如果使用 Spring Boot，可以通过标准的应用程序属性来完成。

在这个快速教程中，我们将学习几种方法来为 Spring 中的测试配置单独的数据源。

## 2。Maven 依赖关系

我们将使用 Spring JPA 和 testing 创建一个 Spring Boot 应用程序，因此我们需要以下依赖项:

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency> 
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
</dependency>
```

最新版本的[spring-boot-starter-data-JPA](https://web.archive.org/web/20220926200024/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22spring-boot-starter-data-jpa%22)、 [h2](https://web.archive.org/web/20220926200024/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22h2%22%20AND%20g%3A%22com.h2database%22) 和 [spring-boot-starter-test](https://web.archive.org/web/20220926200024/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22spring-boot-starter-test%22%20AND%20g%3A%22org.springframework.boot%22) 可以从 Maven Central 下载。

现在让我们看看配置测试用的`DataSource`的几种不同方式。

## 3。使用 Spring Boot 的标准属性文件

Spring Boot 在运行应用程序时自动选择的标准属性文件叫做`application.properties.`，它位于`src/main/resources`文件夹中。

如果我们想为测试使用不同的属性，我们可以通过在`src/test/resources`中放置另一个同名文件来覆盖`main`文件夹中的属性文件。

`src/test/resources`文件夹中的`application.properties`文件应该包含配置数据源所需的标准键值对。这些属性以`spring.datasource`为前缀。

例如，让我们配置一个`H2`内存数据库作为测试的数据源:

```
spring.datasource.driver-class-name=org.h2.Driver
spring.datasource.url=jdbc:h2:mem:db;DB_CLOSE_DELAY=-1
spring.datasource.username=sa
spring.datasource.password=sa
```

Spring Boot 将使用这些属性来自动配置一个`DataSource` bean。

让我们使用 Spring JPA 定义一个非常简单的`GenericEntity`和存储库:

```
@Entity
public class GenericEntity {
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;
    private String value;

    //standard constructors, getters, setters
}
```

```
public interface GenericEntityRepository
  extends JpaRepository<GenericEntity, Long> { }
```

接下来，让我们为存储库编写一个`JUnit`测试。为了让 Spring Boot 应用程序中的测试获得我们定义的标准数据源属性，我们必须用`@SpringBootTest`对其进行注释:

```
@RunWith(SpringRunner.class)
@SpringBootTest(classes = Application.class)
public class SpringBootJPAIntegrationTest {

    @Autowired
    private GenericEntityRepository genericEntityRepository;

    @Test
    public void givenGenericEntityRepository_whenSaveAndRetreiveEntity_thenOK() {
        GenericEntity genericEntity = genericEntityRepository
          .save(new GenericEntity("test"));
        GenericEntity foundEntity = genericEntityRepository
          .findOne(genericEntity.getId());

        assertNotNull(foundEntity);
        assertEquals(genericEntity.getValue(), foundEntity.getValue());
    }
}
```

## 4。使用自定义属性文件

如果我们不想使用标准的`application.properties`文件和密钥，或者如果我们不使用 Spring Boot，我们可以定义一个带有自定义密钥的自定义`.properties`文件，然后在一个`@Configuration`类中读取这个文件，根据它包含的值创建一个`DataSource` bean。

对于应用程序的正常运行模式，该文件将被放置在`src/main/resources`文件夹中，并被放置在 `src/test/resources`中，以便被测试获取。

让我们创建一个名为`persistence-generic-entity.properties`的文件，它使用一个`H2` 内存数据库进行测试，并将它放在 `src/test/resources`文件夹中:

```
jdbc.driverClassName=org.h2.Driver
jdbc.url=jdbc:h2:mem:db;DB_CLOSE_DELAY=-1
jdbc.username=sa
jdbc.password=sa
```

接下来，我们可以在一个将我们的`persistence-generic-entity.properties`作为属性源加载的`@Configuration`类中基于这些属性定义`DataSource` bean:

```
@Configuration
@EnableJpaRepositories(basePackages = "org.baeldung.repository")
@PropertySource("persistence-generic-entity.properties")
@EnableTransactionManagement
public class H2JpaConfig {
    // ...
}
```

关于这种配置的更详细的例子，我们可以通读我们之前关于使用内存数据库进行[自包含测试的文章中的“JPA 配置”一节。](/web/20220926200024/https://www.baeldung.com/spring-jpa-test-in-memory-database)

然后我们可以创建一个类似于前一个的`JUnit`测试，除了它将加载我们的配置类:

```
@RunWith(SpringRunner.class)
@SpringBootTest(classes = {Application.class, H2JpaConfig.class})
public class SpringBootH2IntegrationTest {
    // ...
}
```

## 5。使用弹簧型材

我们可以为测试配置一个单独的`DataSource`的另一种方法是利用 Spring 概要文件来定义一个只在`test`概要文件中可用的`DataSource` bean。

为此，我们可以像以前一样使用一个`.properties`文件，或者我们可以在类本身中写入值。

让我们为`@Configuration`类中的`test`概要文件定义一个`DataSource` bean，它将被我们的测试加载:

```
@Configuration
@EnableJpaRepositories(basePackages = {
  "org.baeldung.repository",
  "org.baeldung.boot.repository"
})
@EnableTransactionManagement
public class H2TestProfileJPAConfig {

    @Bean
    @Profile("test")
    public DataSource dataSource() {
        DriverManagerDataSource dataSource = new DriverManagerDataSource();
        dataSource.setDriverClassName("org.h2.Driver");
        dataSource.setUrl("jdbc:h2:mem:db;DB_CLOSE_DELAY=-1");
        dataSource.setUsername("sa");
        dataSource.setPassword("sa");

        return dataSource;
    }

    // configure entityManagerFactory
    // configure transactionManager
    // configure additional Hibernate properties
}
```

然后，在`JUnit`测试类中，我们需要通过添加`@ActiveProfiles`注释来指定我们想要使用`test`概要文件:

```
@RunWith(SpringRunner.class)
@SpringBootTest(classes = {
  Application.class, 
  H2TestProfileJPAConfig.class})
@ActiveProfiles("test")
public class SpringBootProfileIntegrationTest {
    // ...
}
```

## 6。结论

在这篇简短的文章中，我们探索了几种配置单独的`DataSource`用于 Spring 测试的方法。

和往常一样，例子的完整源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220926200024/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-boot-persistence)