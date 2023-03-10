# spring JPA–多个数据库

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-data-jpa-multiple-databases>

 ![](img/da9334758d663b3cc7b00e6e960bd95b.png)

JPA can behave very differently depending on the exact circumstances under which it is used. Code that works in our local environment or in staging performs very poorly (or even flat out fails) when thrown against real-scale databases in production environments.

Debugging these JPA issues in production is pretty difficult - existing APMs don’t provide enough granular insights at the code level, and tracking every single place someone queried entities one by one instead of in bulk can be a grueling, time-consuming task.

**Lightrun** is a new approach to debugging in production. Using Lightrun’s Logs and Snapshots, you can now get debugger-level granularity in production without opening inbound ports, redeploying, restarting, or even stropping the running application.

In addition, instrumenting Lightrun Metrics at runtime allows you to track down persistence issues securely and in real-time. Want to see it in action? **Check out our 2-minute tutorial** for debugging JPA performance issues in production using Lightrun:

[>> Debugging Spring Persistence and JPA Issues Using Lightrun](/web/20220523145709/https://www.baeldung.com/lightrun-n-jpa)

## 1。概述

在本教程中，我们将为带有多个数据库的 **Spring Data JPA 系统实现一个简单的 Spring 配置。**

## 延伸阅读:

## [Spring 数据 JPA 派生的删除方法](/web/20220523145709/https://www.baeldung.com/spring-data-jpa-deleteby)

Learn how to define Spring Data deleteBy and removeBy methods[Read more](/web/20220523145709/https://www.baeldung.com/spring-data-jpa-deleteby) →

## [在 Spring Boot 以编程方式配置数据源](/web/20220523145709/https://www.baeldung.com/spring-boot-configure-data-source-programmatic)

Learn how to configure a Spring Boot DataSource programmatically, thereby side-stepping Spring Boot's automatic DataSource configuration algorithm.[Read more](/web/20220523145709/https://www.baeldung.com/spring-boot-configure-data-source-programmatic) →

## 2。实体

首先，让我们创建两个简单的实体，每个实体位于一个单独的数据库中。

这里是第一个`User`实体:

```java
package com.baeldung.multipledb.model.user;

@Entity
@Table(schema = "users")
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private int id;

    private String name;

    @Column(unique = true, nullable = false)
    private String email;

    private int age;
}
```

这是第二个实体，`Product`:

```java
package com.baeldung.multipledb.model.product;

@Entity
@Table(schema = "products")
public class Product {

    @Id
    private int id;

    private String name;

    private double price;
}
```

我们可以看到**这两个实体也放在独立的包中。**在我们进入配置阶段时，这一点非常重要。

## 3。JPA 存储库

接下来，让我们看看我们的两个 JPA 存储库，`UserRepository`:

```java
package com.baeldung.multipledb.dao.user;

public interface UserRepository
  extends JpaRepository<User, Integer> { }
```

和 `ProductRepository`:

```java
package com.baeldung.multipledb.dao.product;

public interface ProductRepository
  extends JpaRepository<Product, Integer> { }
```

再次注意我们是如何在不同的包中创建这两个存储库的。

## 4。用 Java 配置 JPA

现在我们来看看实际的弹簧配置。**我们将首先设置两个配置类——一个用于`User`，另一个用于`Product`。**

在每个配置类中，我们需要为`User`定义以下接口:

*   `DataSource`
*   `EntityManagerFactory` ( `userEntityManager`)
*   `TransactionManager` ( `userTransactionManager`)

让我们从查看用户配置开始:

```java
@Configuration
@PropertySource({ "classpath:persistence-multiple-db.properties" })
@EnableJpaRepositories(
    basePackages = "com.baeldung.multipledb.dao.user", 
    entityManagerFactoryRef = "userEntityManager", 
    transactionManagerRef = "userTransactionManager"
)
public class PersistenceUserConfiguration {
    @Autowired
    private Environment env;

    @Bean
    @Primary
    public LocalContainerEntityManagerFactoryBean userEntityManager() {
        LocalContainerEntityManagerFactoryBean em
          = new LocalContainerEntityManagerFactoryBean();
        em.setDataSource(userDataSource());
        em.setPackagesToScan(
          new String[] { "com.baeldung.multipledb.model.user" });

        HibernateJpaVendorAdapter vendorAdapter
          = new HibernateJpaVendorAdapter();
        em.setJpaVendorAdapter(vendorAdapter);
        HashMap<String, Object> properties = new HashMap<>();
        properties.put("hibernate.hbm2ddl.auto",
          env.getProperty("hibernate.hbm2ddl.auto"));
        properties.put("hibernate.dialect",
          env.getProperty("hibernate.dialect"));
        em.setJpaPropertyMap(properties);

        return em;
    }

    @Primary
    @Bean
    public DataSource userDataSource() {

        DriverManagerDataSource dataSource
          = new DriverManagerDataSource();
        dataSource.setDriverClassName(
          env.getProperty("jdbc.driverClassName"));
        dataSource.setUrl(env.getProperty("user.jdbc.url"));
        dataSource.setUsername(env.getProperty("jdbc.user"));
        dataSource.setPassword(env.getProperty("jdbc.pass"));

        return dataSource;
    }

    @Primary
    @Bean
    public PlatformTransactionManager userTransactionManager() {

        JpaTransactionManager transactionManager
          = new JpaTransactionManager();
        transactionManager.setEntityManagerFactory(
          userEntityManager().getObject());
        return transactionManager;
    }
}
```

注意我们如何通过用`@Primary`注释 bean 定义来使用`userTransactionManager`作为我们的**主`TransactionManager`** 。每当我们打算隐式或显式地注入事务管理器而不指定名称时，这是很有帮助的。

接下来，让我们讨论 `PersistenceProductConfiguration`，在这里我们定义相似的 beans:

```java
@Configuration
@PropertySource({ "classpath:persistence-multiple-db.properties" })
@EnableJpaRepositories(
    basePackages = "com.baeldung.multipledb.dao.product", 
    entityManagerFactoryRef = "productEntityManager", 
    transactionManagerRef = "productTransactionManager"
)
public class PersistenceProductConfiguration {
    @Autowired
    private Environment env;

    @Bean
    public LocalContainerEntityManagerFactoryBean productEntityManager() {
        LocalContainerEntityManagerFactoryBean em
          = new LocalContainerEntityManagerFactoryBean();
        em.setDataSource(productDataSource());
        em.setPackagesToScan(
          new String[] { "com.baeldung.multipledb.model.product" });

        HibernateJpaVendorAdapter vendorAdapter = new HibernateJpaVendorAdapter();
        em.setJpaVendorAdapter(vendorAdapter);
        HashMap<String, Object> properties = new HashMap<>();
        properties.put("hibernate.hbm2ddl.auto",
          env.getProperty("hibernate.hbm2ddl.auto"));
        properties.put("hibernate.dialect",
          env.getProperty("hibernate.dialect"));
        em.setJpaPropertyMap(properties);

        return em;
    }

    @Bean
    public DataSource productDataSource() {

        DriverManagerDataSource dataSource
          = new DriverManagerDataSource();
        dataSource.setDriverClassName(
          env.getProperty("jdbc.driverClassName"));
        dataSource.setUrl(env.getProperty("product.jdbc.url"));
        dataSource.setUsername(env.getProperty("jdbc.user"));
        dataSource.setPassword(env.getProperty("jdbc.pass"));

        return dataSource;
    }

    @Bean
    public PlatformTransactionManager productTransactionManager() {

        JpaTransactionManager transactionManager
          = new JpaTransactionManager();
        transactionManager.setEntityManagerFactory(
          productEntityManager().getObject());
        return transactionManager;
    }
}
```

## 5。简单测试

最后，让我们测试我们的配置。

为此，我们将为每个实体创建一个实例，并确保它已创建:

```java
@RunWith(SpringRunner.class)
@SpringBootTest
@EnableTransactionManagement
public class JpaMultipleDBIntegrationTest {

    @Autowired
    private UserRepository userRepository;

    @Autowired
    private ProductRepository productRepository;

    @Test
    @Transactional("userTransactionManager")
    public void whenCreatingUser_thenCreated() {
        User user = new User();
        user.setName("John");
        user.setEmail("[[email protected]](/web/20220523145709/https://www.baeldung.com/cdn-cgi/l/email-protection)");
        user.setAge(20);
        user = userRepository.save(user);

        assertNotNull(userRepository.findOne(user.getId()));
    }

    @Test
    @Transactional("userTransactionManager")
    public void whenCreatingUsersWithSameEmail_thenRollback() {
        User user1 = new User();
        user1.setName("John");
        user1.setEmail("[[email protected]](/web/20220523145709/https://www.baeldung.com/cdn-cgi/l/email-protection)");
        user1.setAge(20);
        user1 = userRepository.save(user1);
        assertNotNull(userRepository.findOne(user1.getId()));

        User user2 = new User();
        user2.setName("Tom");
        user2.setEmail("[[email protected]](/web/20220523145709/https://www.baeldung.com/cdn-cgi/l/email-protection)");
        user2.setAge(10);
        try {
            user2 = userRepository.save(user2);
        } catch (DataIntegrityViolationException e) {
        }

        assertNull(userRepository.findOne(user2.getId()));
    }

    @Test
    @Transactional("productTransactionManager")
    public void whenCreatingProduct_thenCreated() {
        Product product = new Product();
        product.setName("Book");
        product.setId(2);
        product.setPrice(20);
        product = productRepository.save(product);

        assertNotNull(productRepository.findOne(product.getId()));
    }
}
```

## 6.Spring Boot 的多个数据库

Spring Boot 可以简化上面的配置。

默认情况下， **Spring Boot 会用前缀为`spring.datasource.*`** 的配置属性实例化其默认`DataSource`:

```java
spring.datasource.jdbcUrl = [url]
spring.datasource.username = [username]
spring.datasource.password = [password]
```

我们现在想继续使用相同的方式来配置第二个`DataSource`，但是使用不同的属性名称空间:

```java
spring.second-datasource.jdbcUrl = [url]
spring.second-datasource.username = [username]
spring.second-datasource.password = [password]
```

因为我们希望 Spring Boot 自动配置选择这些不同的属性(并实例化两个不同的`DataSources`)，我们将定义两个类似于前面章节的配置类:

```java
@Configuration
@PropertySource({"classpath:persistence-multiple-db-boot.properties"})
@EnableJpaRepositories(
  basePackages = "com.baeldung.multipledb.dao.user",
  entityManagerFactoryRef = "userEntityManager",
  transactionManagerRef = "userTransactionManager")
public class PersistenceUserAutoConfiguration {

    @Primary
    @Bean
    @ConfigurationProperties(prefix="spring.datasource")
    public DataSource userDataSource() {
        return DataSourceBuilder.create().build();
    }
    // userEntityManager bean 

    // userTransactionManager bean
}
```

```java
@Configuration
@PropertySource({"classpath:persistence-multiple-db-boot.properties"})
@EnableJpaRepositories(
  basePackages = "com.baeldung.multipledb.dao.product", 
  entityManagerFactoryRef = "productEntityManager", 
  transactionManagerRef = "productTransactionManager")
public class PersistenceProductAutoConfiguration {

    @Bean
    @ConfigurationProperties(prefix="spring.second-datasource")
    public DataSource productDataSource() {
        return DataSourceBuilder.create().build();
    }

    // productEntityManager bean 

    // productTransactionManager bean
} 
```

现在我们已经根据引导自动配置约定在`persistence-multiple-db-boot.properties` 中定义了数据源属性。

有趣的部分是**用** `**@ConfigurationProperties**` **标注数据源 bean 创建方法。我们只需要在这个方法中指定相应的配置前缀`.`，我们使用了一个`DataSourceBuilder`，Spring Boot 会自动完成剩下的工作。**

但是如何将配置的属性注入到`DataSource`配置中呢？

当调用`DataSourceBuilder`上的`build()`方法时，它将调用其私有的`bind()`方法:

```java
public T build() {
    Class<? extends DataSource> type = getType();
    DataSource result = BeanUtils.instantiateClass(type);
    maybeGetDriverClassName();
    bind(result);
    return (T) result;
}
```

这个私有方法执行许多自动配置的魔术，将解析的配置绑定到实际的`DataSource`实例:

```java
private void bind(DataSource result) {
    ConfigurationPropertySource source = new MapConfigurationPropertySource(this.properties);
    ConfigurationPropertyNameAliases aliases = new ConfigurationPropertyNameAliases();
    aliases.addAliases("url", "jdbc-url");
    aliases.addAliases("username", "user");
    Binder binder = new Binder(source.withAliases(aliases));
    binder.bind(ConfigurationPropertyName.EMPTY, Bindable.ofInstance(result));
}
```

虽然我们自己不需要接触这些代码，但是了解 Spring Boot 自动配置的情况仍然很有用。

除此之外，事务管理器和实体管理器 beans 的配置与标准的 Spring 应用程序相同。

## 7。结论

本文是关于如何配置我们的 Spring Data JPA 项目以使用多个数据库的实用概述。

**的完整实现**可以在[的 GitHub 项目](https://web.archive.org/web/20220523145709/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-data-jpa-enterprise-2 "The Full Example Project on Github")中找到。这是一个基于 Maven 的项目，因此应该很容易导入和运行。