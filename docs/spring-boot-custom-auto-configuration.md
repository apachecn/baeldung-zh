# 使用 Spring Boot 创建自定义自动配置

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-custom-auto-configuration>

## 1。概述

简而言之，Spring Boot 自动配置帮助我们基于类路径上存在的依赖项自动配置 Spring 应用程序。

通过消除定义自动配置类中包含的某些 beans 的需要，这可以使开发更快更容易。

在接下来的部分，我们将看一下**创建我们的自定义 Spring Boot 自动配置。**

## 2。Maven 依赖关系

让我们从依赖关系开始:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
    <version>2.4.0</version>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>8.0.19</version>
</dependency>
```

最新版本的[spring-boot-starter-data-JPA](https://web.archive.org/web/20220908123937/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22spring-boot-starter-data-jpa%22)和 [mysql-connector-java](https://web.archive.org/web/20220908123937/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22mysql-connector-java%22%20AND%20g%3A%22mysql%22) 可以从 Maven Central 下载。

## 3。创建自定义自动配置

**为了创建一个定制的自动配置，我们需要创建一个标注为`@Configuration`的类并注册它。**

让我们为一个`MySQL`数据源创建一个定制配置:

```java
@Configuration
public class MySQLAutoconfiguration {
    //...
}
```

接下来，我们需要将该类注册为自动配置候选。

我们通过在标准文件`resources/META-INF/spring.factories`中的键`org.springframework.boot.autoconfigure.EnableAutoConfiguration`下添加类名来做到这一点:

```java
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
com.baeldung.autoconfiguration.MySQLAutoconfiguration
```

如果我们希望我们的自动配置类优先于其他候选类，我们可以添加`@AutoConfigureOrder(Ordered.HIGHEST_PRECEDENCE)`注释。

我们使用标有`@Conditional`注释的类和 beans 来设计自动配置，以便我们可以替换自动配置或它的特定部分。

**注意，只有当我们没有在应用程序中定义自动配置的 beans 时，自动配置才有效。如果我们定义我们的 bean，它将覆盖默认的 bean。**

### 3.1。上课条件

类条件允许我们**指定，如果使用`@ConditionalOnClass`注释**指定的类存在**，或者如果使用`@ConditionalOnMissingClass`注释指定的类不存在**，我们希望包含一个配置 bean。

让我们指定只有当类`DataSource`存在时，我们的`MySQLConfiguration`才会加载，在这种情况下，我们可以假设应用程序将使用数据库:

```java
@Configuration
@ConditionalOnClass(DataSource.class)
public class MySQLAutoconfiguration {
    //...
}
```

### 3.2。Bean 条件

如果我们希望**仅在指定的 bean 存在或不存在的情况下包含一个 bean**，我们可以使用`@ConditionalOnBean`和`@ConditionalOnMissingBean`注释。

为此，让我们向配置类添加一个`entityManagerFactory` bean。

首先，我们将指定，如果名为`dataSource`的 bean 存在，并且名为`entityManagerFactory`的 bean 尚未定义，我们只希望创建这个 bean:

```java
@Bean
@ConditionalOnBean(name = "dataSource")
@ConditionalOnMissingBean
public LocalContainerEntityManagerFactoryBean entityManagerFactory() {
    LocalContainerEntityManagerFactoryBean em
      = new LocalContainerEntityManagerFactoryBean();
    em.setDataSource(dataSource());
    em.setPackagesToScan("com.baeldung.autoconfiguration.example");
    em.setJpaVendorAdapter(new HibernateJpaVendorAdapter());
    if (additionalProperties() != null) {
        em.setJpaProperties(additionalProperties());
    }
    return em;
}
```

让我们也配置一个`transactionManager` bean，只有当我们还没有定义一个类型为`JpaTransactionManager`的 bean 时，它才会加载:

```java
@Bean
@ConditionalOnMissingBean(type = "JpaTransactionManager")
JpaTransactionManager transactionManager(EntityManagerFactory entityManagerFactory) {
    JpaTransactionManager transactionManager = new JpaTransactionManager();
    transactionManager.setEntityManagerFactory(entityManagerFactory);
    return transactionManager;
}
```

### 3.3。房产条件

我们使用`@ConditionalOnProperty`注释来**指定是否基于 Spring 环境属性的存在和值来加载配置。**

首先，让我们为我们的配置添加一个属性源文件，它将决定从哪里读取属性:

```java
@PropertySource("classpath:mysql.properties")
public class MySQLAutoconfiguration {
    //...
}
```

我们可以配置主`DataSource` bean，我们将使用它来创建到数据库的连接，这样只有当名为`usemysql`的属性存在时，它才会加载。

我们可以使用属性`havingValue`来指定必须匹配的`usemysql`属性的某些值。

现在让我们用默认值定义`dataSource` bean，如果我们将`usemysql`属性设置为`local`，它将连接到名为`myDb`的本地数据库:

```java
@Bean
@ConditionalOnProperty(
  name = "usemysql", 
  havingValue = "local")
@ConditionalOnMissingBean
public DataSource dataSource() {
    DriverManagerDataSource dataSource = new DriverManagerDataSource();

    dataSource.setDriverClassName("com.mysql.cj.jdbc.Driver");
    dataSource.setUrl("jdbc:mysql://localhost:3306/myDb?createDatabaseIfNotExist=true");
    dataSource.setUsername("mysqluser");
    dataSource.setPassword("mysqlpass");

    return dataSource;
}
```

如果我们将`usemysql`属性设置为`custom`，我们将使用数据库 URL、用户和密码的自定义属性值来配置`dataSource` bean:

```java
@Bean(name = "dataSource")
@ConditionalOnProperty(
  name = "usemysql", 
  havingValue = "custom")
@ConditionalOnMissingBean
public DataSource dataSource2() {
    DriverManagerDataSource dataSource = new DriverManagerDataSource();

    dataSource.setDriverClassName("com.mysql.cj.jdbc.Driver");
    dataSource.setUrl(env.getProperty("mysql.url"));
    dataSource.setUsername(env.getProperty("mysql.user") != null 
      ? env.getProperty("mysql.user") : "");
    dataSource.setPassword(env.getProperty("mysql.pass") != null 
      ? env.getProperty("mysql.pass") : "");

    return dataSource;
}
```

`mysql.properties`文件将包含`usemysql`属性:

```java
usemysql=local
```

使用`MySQLAutoconfiguration`的应用程序可能需要覆盖默认属性。在这种情况下，只需要为`mysql.properties`文件中的`mysql.url`、`mysql.user`、`mysql.pass`属性和`usemysql=custom`行添加不同的值即可。

### 3.4。资源条件

添加`@ConditionalOnResource`注释意味着**配置仅在指定资源存在时加载。**

让我们定义一个名为`additionalProperties()`的方法，该方法将返回一个包含特定于 Hibernate 的属性的`Properties`对象，供`entityManagerFactory` bean 使用，前提是资源文件`mysql.properties`存在:

```java
@ConditionalOnResource(
  resources = "classpath:mysql.properties")
@Conditional(HibernateCondition.class)
Properties additionalProperties() {
    Properties hibernateProperties = new Properties();

    hibernateProperties.setProperty("hibernate.hbm2ddl.auto", 
      env.getProperty("mysql-hibernate.hbm2ddl.auto"));
    hibernateProperties.setProperty("hibernate.dialect", 
      env.getProperty("mysql-hibernate.dialect"));
    hibernateProperties.setProperty("hibernate.show_sql", 
      env.getProperty("mysql-hibernate.show_sql") != null 
      ? env.getProperty("mysql-hibernate.show_sql") : "false");
    return hibernateProperties;
}
```

我们可以将特定于 Hibernate 的属性添加到`mysql.properties`文件中:

```java
mysql-hibernate.dialect=org.hibernate.dialect.MySQLDialect
mysql-hibernate.show_sql=true
mysql-hibernate.hbm2ddl.auto=create-drop
```

### 3.5。自定义条件

假设我们不想使用 Spring Boot 的任何可用条件。

我们还可以通过扩展`SpringBootCondition`类和覆盖`getMatchOutcome()`方法来**定义自定义条件。**

让我们为我们的`additionalProperties()`方法创建一个名为`HibernateCondition`的条件，它将验证一个`HibernateEntityManager`类是否存在于类路径中:

```java
static class HibernateCondition extends SpringBootCondition {

    private static String[] CLASS_NAMES
      = { "org.hibernate.ejb.HibernateEntityManager", 
          "org.hibernate.jpa.HibernateEntityManager" };

    @Override
    public ConditionOutcome getMatchOutcome(ConditionContext context, 
      AnnotatedTypeMetadata metadata) {

        ConditionMessage.Builder message
          = ConditionMessage.forCondition("Hibernate");
        return Arrays.stream(CLASS_NAMES)
          .filter(className -> ClassUtils.isPresent(className, context.getClassLoader()))
          .map(className -> ConditionOutcome
            .match(message.found("class")
            .items(Style.NORMAL, className)))
          .findAny()
          .orElseGet(() -> ConditionOutcome
            .noMatch(message.didNotFind("class", "classes")
            .items(Style.NORMAL, Arrays.asList(CLASS_NAMES))));
    }
}
```

然后我们可以将条件添加到`additionalProperties()`方法中:

```java
@Conditional(HibernateCondition.class)
Properties additionalProperties() {
  //...
}
```

### 3.6。申请条件

我们还可以**指定配置只能在 web 上下文内部/外部加载。**为了做到这一点，我们可以添加`@ConditionalOnWebApplication`或`@ConditionalOnNotWebApplication`注释。

## 4。测试自动配置

让我们创建一个非常简单的例子来测试我们的自动配置。

我们将使用 Spring 数据创建一个名为`MyUser`的实体类和一个`MyUserRepository`接口:

```java
@Entity
public class MyUser {
    @Id
    private String email;

    // standard constructor, getters, setters
}
```

```java
public interface MyUserRepository 
  extends JpaRepository<MyUser, String> { }
```

为了启用自动配置，我们可以使用`@SpringBootApplication`或`@EnableAutoConfiguration`注释之一:

```java
@SpringBootApplication
public class AutoconfigurationApplication {
    public static void main(String[] args) {
        SpringApplication.run(AutoconfigurationApplication.class, args);
    }
}
```

接下来，让我们编写一个保存一个`MyUser`实体的`JUnit`测试:

```java
@RunWith(SpringJUnit4ClassRunner.class)
@SpringBootTest(
  classes = AutoconfigurationApplication.class)
@EnableJpaRepositories(
  basePackages = { "com.baeldung.autoconfiguration.example" })
public class AutoconfigurationLiveTest {

    @Autowired
    private MyUserRepository userRepository;

    @Test
    public void whenSaveUser_thenOk() {
        MyUser user = new MyUser("[[email protected]](/web/20220908123937/https://www.baeldung.com/cdn-cgi/l/email-protection)");
        userRepository.save(user);
    }
}
```

由于我们没有定义我们的`DataSource`配置，应用程序将使用我们创建的自动配置连接到一个名为`myDb`的`MySQL`数据库。

**连接字符串包含`createDatabaseIfNotExist=true`属性，所以数据库不需要存在。然而，需要创建用户`mysqluser`，或者通过`mysql.user`属性指定的用户(如果存在的话)。**

我们可以检查应用程序日志，看看我们使用的是`MySQL`数据源:

```java
web - 2017-04-12 00:01:33,956 [main] INFO  o.s.j.d.DriverManagerDataSource - Loaded JDBC driver: com.mysql.cj.jdbc.Driver
```

## 5。禁用自动配置类

假设我们想要**从加载中排除自动配置。**

我们可以将带有`exclude`或`excludeName`属性的`@EnableAutoConfiguration`注释添加到配置类中:

```java
@Configuration
@EnableAutoConfiguration(
  exclude={MySQLAutoconfiguration.class})
public class AutoconfigurationApplication {
    //...
}
```

我们还可以设置`spring.autoconfigure.exclude`属性:

```java
spring.autoconfigure.exclude=com.baeldung.autoconfiguration.MySQLAutoconfiguration
```

## 6。结论

在本文中，我们展示了如何创建一个定制的 Spring Boot 自动配置。

这个例子的完整源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220908123937/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-autoconfiguration)

JUnit 测试可以使用`autoconfiguration`概要文件`mvn clean install -Pautoconfiguration`来运行。