# Spring、Hibernate 和一个 JNDI 数据源

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-persistence-jpa-jndi-datasource>

## 1。概述

在本文中，我们将使用 Hibernate/JPA 创建一个带有 [JNDI](https://web.archive.org/web/20220625173605/https://en.wikipedia.org/wiki/Java_Naming_and_Directory_Interface) 数据源的 Spring 应用程序。

如果你想重新发现 Spring 和 Hibernate 的基础知识，请查看本文。

## 2。声明数据源

### 2.1。系统

因为我们使用的是 JNDI 数据源，所以我们不会在应用程序中定义它，我们将在应用程序容器中定义它。

在这个例子中，我们将使用 8.5.x 版本的 [Tomcat](https://web.archive.org/web/20220625173605/https://tomcat.apache.org/) 和 9.5.x 版本的 [PostgreSQL](https://web.archive.org/web/20220625173605/https://www.postgresql.org/) 数据库。

您应该能够使用任何其他 Java 应用程序容器和您选择的数据库来复制相同的步骤(只要您有合适的 JDBC jar！).

### 2.2。在应用程序容器上声明数据源

我们将在`<GlobalNamingResources>`元素内的`<tomcat_home>/conf/` `server.xml`文件中声明我们的数据源。

假设数据库服务器与应用程序容器运行在同一台机器上，目标数据库名为`postgres`，用户名为`baeldung`，密码为`pass1234`，资源看起来如下:

```java
<Resource name="jdbc/BaeldungDatabase" 
  auth="Container"
  type="javax.sql.DataSource" 
  driverClassName="org.postgresql.Driver"
  url="jdbc:postgresql://localhost:5432/postgres"
  username="baeldung" 
  password="pass1234" 
  maxTotal="20" 
  maxIdle="10" 
  maxWaitMillis="-1"/>
```

请注意，我们已经将我们的资源命名为`jdbc/BaeldungDatabase`。这将是引用此数据源时使用的名称。

我们还必须指定它的类型和数据库驱动程序的类名。为了让它工作，您还必须将相应的 jar 放在`<tomcat_home>/lib/`中(在本例中，是 PostgreSQL 的 JDBC jar)。

剩余的配置参数包括:

*   `auth=”Container”` –表示容器将代表应用程序登录到资源管理器
*   `maxTotal, maxIdle,` 和`maxWaitMillis`–是池连接的配置参数

我们还必须在`<tomcat_home>/conf/context` `.xml,` 中的`<Context>`元素内定义一个`ResourceLink`,如下所示:

```java
<ResourceLink 
  name="jdbc/BaeldungDatabase" 
  global="jdbc/BaeldungDatabase" 
  type="javax.sql.DataSource"/>
```

请注意，我们在`server.xml`中使用了我们在`Resource`中定义的名称。

## 3。使用资源

### 3.1。设置应用程序

我们现在将使用纯 Java 配置定义一个简单的 Spring + JPA + Hibernate 应用程序。

我们将从定义 Spring 上下文的配置开始(请记住，我们这里关注的是 JNDI，并且假设您已经了解了 Spring 配置的基础):

```java
@Configuration
@EnableTransactionManagement
@PropertySource("classpath:persistence-jndi.properties")
@ComponentScan("com.baeldung.hibernate.cache")
@EnableJpaRepositories(basePackages = "com.baeldung.hibernate.cache.dao")
public class PersistenceJNDIConfig {

    @Autowired
    private Environment env;

    @Bean
    public LocalContainerEntityManagerFactoryBean entityManagerFactory() 
      throws NamingException {
        LocalContainerEntityManagerFactoryBean em 
          = new LocalContainerEntityManagerFactoryBean();
        em.setDataSource(dataSource());

        // rest of entity manager configuration
        return em;
    }

    @Bean
    public DataSource dataSource() throws NamingException {
        return (DataSource) new JndiTemplate().lookup(env.getProperty("jdbc.url"));
    }

    @Bean
    public PlatformTransactionManager transactionManager(EntityManagerFactory emf) {
        JpaTransactionManager transactionManager = new JpaTransactionManager();
        transactionManager.setEntityManagerFactory(emf);
        return transactionManager;
    }

    // rest of persistence configuration
}
```

注意，我们在 [Spring 4 和 JPA with Hibernate](/web/20220625173605/https://www.baeldung.com/the-persistence-layer-with-spring-and-jpa) 文章中有一个完整的配置示例。

为了创建我们的`dataSource` bean，我们需要寻找我们在应用程序容器中定义的 JNDI 资源。我们将把它存储在`persistence-jndi.properties`键中(在其他属性中):

```java
jdbc.url=java:comp/env/jdbc/BaeldungDatabase
```

注意，在`jdbc.url property`中，我们定义了一个要查找的根名称:`java:comp/env/`(这是默认值，对应于组件和环境)，然后是我们在`server.xml`中使用的相同名称:`jdbc/BaeldungDatabase`。

### 3.2。JPA 配置–型号、DAO 和服务

我们将使用一个带有`@Entity`注释的简单模型，并生成一个`id`和一个`name`:

```java
@Entity
public class Foo {

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    @Column(name = "ID")
    private Long id;

    @Column(name = "NAME")
    private String name;

    // default getters and setters
}
```

让我们定义一个简单的存储库:

```java
@Repository
public class FooDao {

    @PersistenceContext
    private EntityManager entityManager;

    public List<Foo> findAll() {
        return entityManager
          .createQuery("from " + Foo.class.getName()).getResultList();
    }
}
```

最后，让我们创建一个简单的服务:

```java
@Service
@Transactional
public class FooService {

    @Autowired
    private FooDao dao;

    public List<Foo> findAll() {
        return dao.findAll();
    }
}
```

这样，您就拥有了在 Spring 应用程序中使用 JNDI 数据源所需的一切。

## 4。结论

在本文中，我们用 JPA + Hibernate 设置创建了一个示例 Spring 应用程序，它使用 JNDI 数据源。

注意，最重要的部分是应用程序容器中资源的定义和配置中 JNDI 资源的查找。

和往常一样，完整的项目可以在 GitHub 上找到[。](https://web.archive.org/web/20220625173605/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-hibernate-5)