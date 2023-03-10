# Hibernate exception:Hibernate 3 中没有绑定到线程的 Hibernate 会话

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/no-hibernate-session-bound-to-thread-exception>

## 1。简介

在这个简短的教程中，**我们将阐明什么时候抛出“没有绑定到线程的 Hibernate 会话”异常，以及如何解决它。**

我们将重点关注两种不同的场景:

1.  使用 `LocalSessionFactoryBean`
2.  使用`AnnotationSessionFactoryBean`

## 2。起因

在版本 3 中，Hibernate 引入了上下文会话的概念，并将`getCurrentSession()` 方法添加到了`SessionFactory`类中。关于上下文会话的更多信息可以在[这里](https://web.archive.org/web/20221208143856/https://docs.jboss.org/hibernate/stable/core.old/reference/en/html/architecture-current-session.html)找到。

Spring 有自己的`org.hibernate.context.CurrentSessionContext`接口实现—`org.springframework.orm.hibernate3.SpringSessionContext` (在 Spring Hibernate 3 的情况下)。**这个实现需要将会话绑定到一个事务。**

自然，调用`getCurrentSession()` 方法的类应该在类级别或方法级别用`@Transactional` 进行注释。否则，`org.hibernate.HibernateException: No Hibernate Session Bound to Thread` 将被抛出。

让我们快速看一个例子。

## 3。`LocalFactorySessionBean`

他是我们在本文中要考虑的第一个场景。

我们将用`LocalSessionFactoryBean`定义一个 Java Spring 配置类:

```java
@Configuration
@EnableTransactionManagement
@PropertySource(
  { "classpath:persistence-h2.properties" }
)
@ComponentScan(
  { "com.baeldung.persistence.dao", "com.baeldung.persistence.service" }
)
public class PersistenceConfigHibernate3 {   
    // ...    
    @Bean
    public LocalSessionFactoryBean sessionFactory() {
        LocalSessionFactoryBean sessionFactory 
          = new LocalSessionFactoryBean();
        Resource config = new ClassPathResource("exceptionDemo.cfg.xml");
        sessionFactory.setDataSource(dataSource());
        sessionFactory.setConfigLocation(config);
        sessionFactory.setHibernateProperties(hibernateProperties());

        return sessionFactory;
    }    
    // ...
}
```

注意，为了映射模型类，我们在这里使用了一个 Hibernate 配置文件(`exceptionDemo.cfg.xml`)。这是因为**`org.springframework.orm.hibernate3.LocalSessionFactoryBean`没有为映射模型类提供属性** `**packagesToScan**,` 。

这是我们的简单服务:

```java
@Service
@Transactional
public class EventService {

    @Autowired
    private IEventDao dao;

    public void create(Event entity) {
        dao.create(entity);
    }
}
```

```java
@Entity
@Table(name = "EVENTS")
public class Event implements Serializable {
    @Id
    @GeneratedValue
    private Long id;
    private String description;

    // ...
 }
```

正如我们在下面的代码片段中看到的，使用了`SessionFactory` 类的`getCurrentSession()` 方法来获得 Hibernate 会话:

```java
public abstract class AbstractHibernateDao<T extends Serializable> 
  implements IOperations<T> {
    private Class<T> clazz;
    @Autowired
    private SessionFactory sessionFactory;
    // ...

    @Override
    public void create(T entity) {
        Preconditions.checkNotNull(entity);
        getCurrentSession().persist(entity);
    }

    protected Session getCurrentSession() {
        return sessionFactory.getCurrentSession();
    }
}
```

下面的测试通过了，演示了当包含服务方法的类`EventService`没有用`@Transactional` 注释进行注释时，将如何抛出异常:

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(
  classes = { PersistenceConfigHibernate3.class }, 
  loader = AnnotationConfigContextLoader.class
)
public class HibernateExceptionScen1MainIntegrationTest {
    @Autowired
    EventService service;

    @Rule
    public ExpectedException expectedEx = ExpectedException.none();

    @Test
    public void whenNoTransBoundToSession_thenException() {
        expectedEx.expectCause(
          IsInstanceOf.<Throwable>instanceOf(HibernateException.class));
        expectedEx.expectMessage("No Hibernate Session bound to thread, "
          + "and configuration does not allow creation "
          + "of non-transactional one here");
        service.create(new Event("from LocalSessionFactoryBean"));
    }
}
```

这个测试展示了当用`@Transactional` 注释对`EventService`类进行注释时，服务方法是如何成功执行的:

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(
  classes = { PersistenceConfigHibernate3.class }, 
  loader = AnnotationConfigContextLoader.class
)
public class HibernateExceptionScen1MainIntegrationTest {
    @Autowired
    EventService service;

    @Rule
    public ExpectedException expectedEx = ExpectedException.none();

    @Test
    public void whenEntityIsCreated_thenNoExceptions() {
        service.create(new Event("from LocalSessionFactoryBean"));
        List<Event> events = service.findAll();
    }
}
```

## 4。`AnnotationSessionFactoryBean`

当我们在 Spring 应用程序中使用`org.springframework.orm.hibernate3.annotation.AnnotationSessionFactoryBean` 创建`SessionFactory`时，也会出现这种异常。

让我们来看一些演示这一点的示例代码。为此，我们用`AnnotationSessionFactoryBean`定义了一个 Java Spring 配置类:

```java
@Configuration
@EnableTransactionManagement
@PropertySource(
  { "classpath:persistence-h2.properties" }
)
@ComponentScan(
  { "com.baeldung.persistence.dao", "com.baeldung.persistence.service" }
)
public class PersistenceConfig {
    //...
    @Bean
    public AnnotationSessionFactoryBean sessionFactory() {
        AnnotationSessionFactoryBean sessionFactory 
          = new AnnotationSessionFactoryBean();
        sessionFactory.setDataSource(dataSource());
        sessionFactory.setPackagesToScan(
          new String[] { "com.baeldung.persistence.model" });
        sessionFactory.setHibernateProperties(hibernateProperties());

        return sessionFactory;
    }
    // ...
}
```

对于上一节中的同一组 DAO、服务和模型类，我们会遇到如上所述的异常:

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(
  classes = { PersistenceConfig.class }, 
  loader = AnnotationConfigContextLoader.class
)
public class HibernateExceptionScen2MainIntegrationTest {
    @Autowired
    EventService service;

    @Rule
    public ExpectedException expectedEx = ExpectedException.none();

    @Test
    public void whenNoTransBoundToSession_thenException() {
        expectedEx.expectCause(
          IsInstanceOf.<Throwable>instanceOf(HibernateException.class));
        expectedEx.expectMessage("No Hibernate Session bound to thread, "
          + "and configuration does not allow creation "
          + "of non-transactional one here");
        service.create(new Event("from AnnotationSessionFactoryBean"));
    }
}
```

如果我们用一个`@Transactional` 注释来注释服务类，那么服务方法会像预期的那样工作，并且下面显示的测试会通过:

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(
  classes = { PersistenceConfig.class }, 
  loader = AnnotationConfigContextLoader.class
)
public class HibernateExceptionScen2MainIntegrationTest {
    @Autowired
    EventService service;

    @Rule
    public ExpectedException expectedEx = ExpectedException.none();

    @Test
    public void whenEntityIsCreated_thenNoExceptions() {
        service.create(new Event("from AnnotationSessionFactoryBean"));
        List<Event> events = service.findAll();
    }
}
```

## 5。解决方案

很明显，从 Spring 获得的`SessionFactory` 的`getCurrentSession()` 方法需要从一个打开的事务中调用。因此，**解决方案是确保我们的 DAO/服务方法/类用`@Transactional`标注正确地标注。**

应该注意的是，在 Hibernate 4 和更高版本中，由于同样的原因抛出的异常消息的措辞是不同的。我们得到的不是“`**No Hibernate Session Bound to Thread”,**` ”，而是“**无法获得当前线程的事务同步会话”。**

还有一点很重要。除了`org.hibernate.context.CurrentSessionContext` 接口，Hibernate 还引入了一个属性`hibernate.current_session_context_class` ，该属性可以设置为实现当前会话上下文的类。

如前所述，Spring 自带这个接口的实现:默认情况下，它将`SpringSessionContext.` 属性设置为等于这个类。

因此，如果我们显式地将这个属性设置为其他属性，就会破坏 Spring 管理 Hibernate 会话和事务的能力。这也导致了一个异常，但与正在考虑的异常不同。

总之，重要的是要记住，当我们使用 Spring 管理 Hibernate 会话时，我们不应该显式地设置`hibernate.current_session_context_class`。

## 6。结论

在本文中，我们研究了 Hibernate 3 中异常 `org.hibernate.HibernateException: No Hibernate Session Bound to Thread` 抛出的原因以及一些示例代码，以及如何轻松解决这个问题。

这篇文章的代码可以在 Github 上找到[。](https://web.archive.org/web/20221208143856/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-hibernate-3)