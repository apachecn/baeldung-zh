# 休眠映射异常-未知实体

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/hibernate-mappingexception-unknown-entity>

## 1。问题

本文将讨论针对 Hibernate 以及 Spring 和 Hibernate 环境的 **`org.hibernate.MappingException`:未知实体**问题和解决方案。

## 延伸阅读:

## [用 Spring 引导 Hibernate 5](/web/20221012100323/https://www.baeldung.com/hibernate-5-spring)

A quick and practical guide to integrating Hibernate 5 with Spring.[Read more](/web/20221012100323/https://www.baeldung.com/hibernate-5-spring) →

## [@在 Hibernate 中不可变](/web/20221012100323/https://www.baeldung.com/hibernate-immutable)

A quick and practical guide to @Immutable annotation in Hibernate[Read more](/web/20221012100323/https://www.baeldung.com/hibernate-immutable) →

## [Hibernate exception:Hibernate 3](/web/20221012100323/https://www.baeldung.com/no-hibernate-session-bound-to-thread-exception)中没有绑定到线程的 Hibernate 会话

Discover when "No Hibernate Session Bound to Thread" exception gets thrown and how to deal with it.[Read more](/web/20221012100323/https://www.baeldung.com/no-hibernate-session-bound-to-thread-exception) →

## 2。`@Entity`标注缺失或无效

映射异常最常见的原因是实体类**缺少`@Entity`注释**:

```java
public class Foo implements Serializable {

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private long id;

    public Foo() {
        super();
    }

    public long getId() {
        return id;
    }
    public void setId(long id) {
        this.id = id;
    }
}
```

另一种可能是，它可能有**错误类型的`@Entity`注解**:

```java
import org.hibernate.annotations.Entity;

@Entity
public class Foo implements Serializable {
    ...
```

不赞成使用的`org.hibernate.annotations.Entity`是错误的实体类型——我们需要的**是`javax.persistence.Entity`** :

```java
import javax.persistence.Entity;

@Entity
public class Foo implements Serializable {
    ...
```

## 3。`MappingException`同春

Spring 中 Hibernate 的[配置包括通过`LocalSessionFactoryBean`从注释扫描中引导`SessionFactory`:](/web/20221012100323/https://www.baeldung.com/hibernate-4-spring "Hibernate 4 with Spring")

```java
@Bean
public LocalSessionFactoryBean sessionFactory() {
    LocalSessionFactoryBean sessionFactory = new LocalSessionFactoryBean();
    sessionFactory.setDataSource(restDataSource());
    ...
    return sessionFactory;
}
```

会话工厂 Bean 的这个简单配置缺少一个关键要素，尝试使用`SessionFactory`的测试将会失败:

```java
...
@Autowired
private SessionFactory sessionFactory;

@Test(expected = MappingException.class)
@Transactional
public void givenEntityIsPersisted_thenException() {
    sessionFactory.getCurrentSession().saveOrUpdate(new Foo());
}
```

不出所料，例外是`MappingException: Unknown entity`:

```java
org.hibernate.MappingException: Unknown entity: 
com.baeldung.ex.mappingexception.persistence.model.Foo
    at o.h.i.SessionFactoryImpl.getEntityPersister(SessionFactoryImpl.java:1141)
```

现在，这个问题有两个**解决方案——两种方式告诉`LocalSessionFactoryBean`关于`Foo`实体类。**

我们可以指定在类路径中搜索实体类的**包:**

```java
sessionFactory.setPackagesToScan(
  new String[] { "com.baeldung.ex.mappingexception.persistence.model" });
```

或者我们可以简单地**将实体类直接**注册到会话工厂:

```java
sessionFactory.setAnnotatedClasses(new Class[] { Foo.class });
```

使用这些附加的配置行中的任何一行，测试现在都将正确运行并通过。

## 4。`MappingException`同冬眠

现在让我们看看仅使用 Hibernate 时的错误:

```java
public class Cause4MappingExceptionIntegrationTest {

    @Test
    public void givenEntityIsPersisted_thenException() throws IOException {
        SessionFactory sessionFactory = configureSessionFactory();

        Session session = sessionFactory.openSession();
        session.beginTransaction();
        session.saveOrUpdate(new Foo());
        session.getTransaction().commit();
    }

    private SessionFactory configureSessionFactory() throws IOException {
        Configuration configuration = new Configuration();
        InputStream inputStream = this.getClass().getClassLoader().
          getResourceAsStream("hibernate-mysql.properties");
        Properties hibernateProperties = new Properties();
        hibernateProperties.load(inputStream);
        configuration.setProperties(hibernateProperties);

        // configuration.addAnnotatedClass(Foo.class);

        ServiceRegistry serviceRegistry = new ServiceRegistryBuilder().
          applySettings(configuration.getProperties()).buildServiceRegistry();
        SessionFactory sessionFactory = configuration.buildSessionFactory(serviceRegistry);
        return sessionFactory;
    }
}
```

`hibernate-mysql.properties`文件包含**休眠配置属性**:

```java
hibernate.connection.username=tutorialuser
hibernate.connection.password=tutorialmy5ql
hibernate.connection.driver_class=com.mysql.jdbc.Driver
hibernate.dialect=org.hibernate.dialect.MySQL5Dialect
hibernate.connection.url=jdbc:mysql://localhost:3306/spring_hibernate4_exceptions
hibernate.show_sql=false
hibernate.hbm2ddl.auto=create
```

运行该测试将导致相同的映射异常:

```java
org.hibernate.MappingException: 
  Unknown entity: com.baeldung.ex.mappingexception.persistence.model.Foo
    at o.h.i.SessionFactoryImpl.getEntityPersister(SessionFactoryImpl.java:1141)
```

从上面的例子中可能已经很清楚了，配置中缺少的是**向配置**中添加实体类`Foo`的元数据:

```java
configuration.addAnnotatedClass(Foo.class);
```

这修复了测试——现在可以持久化 Foo 实体了。

## 5。结论

本文阐述了未知实体映射异常可能发生的原因，以及当它发生时如何解决问题，首先是在实体级别，然后是 Spring 和 Hibernate，最后是 Hibernate 本身。

在 github 项目中可以找到所有异常示例的实现——这是一个基于 Eclipse 的项目，因此应该很容易导入和运行。