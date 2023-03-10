# 用 Java 以编程方式引导 JPA

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-bootstrap-jpa>

## 1。概述

大多数 JPA 驱动的应用程序大量使用`“persistence.xml”`文件来获得 JPA 实现，例如 [Hibernate](https://web.archive.org/web/20221128051852/http://hibernate.org/) 或 [OpenJPA](https://web.archive.org/web/20221128051852/https://openjpa.apache.org/) 。

我们这里的方法**提供了一个集中的机制，用于配置一个或多个[持久单元](https://web.archive.org/web/20221128051852/https://docs.oracle.com/cd/E19798-01/821-1841/bnbrj/index.html)** 和相关的[持久上下文](https://web.archive.org/web/20221128051852/https://docs.jboss.org/hibernate/orm/4.0/devguide/en-US/html/ch03.html)。

虽然这种方法本身并没有错，但它不适合需要单独测试使用不同持久性单元的应用程序组件的用例。

从好的方面来看，**完全不借助于`“persistence.xml”`文件，只使用普通的 Java** 就可以引导 JPA 实现。

在本教程中，我们将看到如何用 Hibernate 来完成这个任务。

## 2。实现`PersistenceUnitInfo`接口

在典型的“基于 xml”的 JPA 配置中，JPA 实现自动负责实现 `[PersistenceUnitInfo](https://web.archive.org/web/20221128051852/https://docs.oracle.com/javaee/7/api/javax/persistence/spi/PersistenceUnitInfo.html)`接口。

使用通过解析`“persistence.xml”`文件收集的所有数据，持久性提供者使用这个实现来创建一个实体管理器工厂。从这个工厂，我们可以获得一个实体管理器。

**既然我们不会依赖 `“persistence.xml”` 文件，t 他我们需要做的第一件事就是提供我们自己的`PersistenceUnitInfo`实现。**我们将使用 Hibernate 作为我们的持久性提供者:

```java
public class HibernatePersistenceUnitInfo implements PersistenceUnitInfo {

    public static String JPA_VERSION = "2.1";
    private String persistenceUnitName;
    private PersistenceUnitTransactionType transactionType
      = PersistenceUnitTransactionType.RESOURCE_LOCAL;
    private List<String> managedClassNames;
    private List<String> mappingFileNames = new ArrayList<>();
    private Properties properties;
    private DataSource jtaDataSource;
    private DataSource nonjtaDataSource;
    private List<ClassTransformer> transformers = new ArrayList<>();

    public HibernatePersistenceUnitInfo(
      String persistenceUnitName, List<String> managedClassNames, Properties properties) {
        this.persistenceUnitName = persistenceUnitName;
        this.managedClassNames = managedClassNames;
        this.properties = properties;
    }

    // standard setters / getters   
}
```

简而言之，`HibernatePersistenceUnitInfo`类只是一个普通的数据容器，它存储绑定到特定持久性单元的配置参数。这包括持久性单元名称、受管实体类名、事务类型、JTA 和非 JTA 数据源等等。

## 3。用 Hibernate 的`EntityManagerFactoryBuilderImpl`类创建实体管理器工厂

既然我们已经有了一个定制的`PersistenceUnitInfo`实现，我们需要做的最后一件事就是获得一个实体管理器工厂。

**Hibernate 通过它的 [`EntityManagerFactoryBuilderImpl`](https://web.archive.org/web/20221128051852/https://docs.jboss.org/hibernate/orm/5.0/javadocs/org/hibernate/jpa/boot/internal/EntityManagerFactoryBuilderImpl.html) (一个构建器模式的简洁实现)**让这个过程变得轻而易举。

为了提供更高层次的抽象，让我们创建一个封装了`EntityManagerFactoryBuilderImpl.`功能的类

首先，让我们展示使用`Hibernate's EntityManagerFactoryBuilderImpl`类和我们的`HibernatePersistenceUnitInf` o 类创建**实体管理器工厂和实体管理器**的方法:

```java
public class JpaEntityManagerFactory {
    private String DB_URL = "jdbc:mysql://databaseurl";
    private String DB_USER_NAME = "username";
    private String DB_PASSWORD = "password";
    private Class[] entityClasses;

    public JpaEntityManagerFactory(Class[] entityClasses) {
        this.entityClasses = entityClasses;
    }

    public EntityManager getEntityManager() {
        return getEntityManagerFactory().createEntityManager();
    }

    protected EntityManagerFactory getEntityManagerFactory() {
        PersistenceUnitInfo persistenceUnitInfo = getPersistenceUnitInfo(
          getClass().getSimpleName());
        Map<String, Object> configuration = new HashMap<>();
        return new EntityManagerFactoryBuilderImpl(
          new PersistenceUnitInfoDescriptor(persistenceUnitInfo), configuration)
          .build();
    }

    protected HibernatePersistenceUnitInfo getPersistenceUnitInfo(String name) {
        return new HibernatePersistenceUnitInfo(name, getEntityClassNames(), getProperties());
    }

    // additional methods
} 
```

接下来，我们来看看提供`EntityManagerFactoryBuilderImpl`和`HibernatePersistenceUnitInf` o 所需参数的**方法。**

这些参数包括托管实体类、实体类名、Hibernate 的配置属性和一个`MysqlDataSource`对象:

```java
public class JpaEntityManagerFactory {
    //...

    protected List<String> getEntityClassNames() {
        return Arrays.asList(getEntities())
          .stream()
          .map(Class::getName)
          .collect(Collectors.toList());
    }

    protected Properties getProperties() {
        Properties properties = new Properties();
        properties.put("hibernate.dialect", "org.hibernate.dialect.MySQLDialect");
        properties.put("hibernate.id.new_generator_mappings", false);
        properties.put("hibernate.connection.datasource", getMysqlDataSource());
        return properties;
    }

    protected Class[] getEntities() {
        return entityClasses;
    }

    protected DataSource getMysqlDataSource() {
        MysqlDataSource mysqlDataSource = new MysqlDataSource();
        mysqlDataSource.setURL(DB_URL);
        mysqlDataSource.setUser(DB_USER_NAME);
        mysqlDataSource.setPassword(DB_PASSWORD);
        return mysqlDataSource;
    }
} 
```

为了简单起见，我们将数据库连接参数硬编码在`JpaEntityManagerFactory`类中。但是，在生产中，我们应该将它们存储在一个单独的属性文件中。

此外，`getMysqlDataSource()`方法返回一个完全初始化的`MysqlDataSource`对象。

我们这样做只是为了让事情容易跟踪。在一个更现实、松耦合的设计中，我们将使用`EntityManagerFactoryBuilderImpl'` s [`withDataSource()`](https://web.archive.org/web/20221128051852/https://docs.jboss.org/hibernate/orm/5.0/javadocs/org/hibernate/jpa/boot/internal/EntityManagerFactoryBuilderImpl.html#withDataSource-javax.sql.DataSource-) 方法**注入一个`DataSource`对象，而不是在类**中创建它。

## 4。用实体管理器执行 CRUD 操作

最后，让我们看看如何使用一个`JpaEntityManagerFactory`实例来获取 JPA 实体管理器并执行 CRUD 操作。(注意，为了简洁起见，我们省略了`User`类):

```java
public static void main(String[] args) {
    EntityManager entityManager = getJpaEntityManager();
    User user = entityManager.find(User.class, 1);

    entityManager.getTransaction().begin();
    user.setName("John");
    user.setEmail("[[email protected]](/web/20221128051852/https://www.baeldung.com/cdn-cgi/l/email-protection)");
    entityManager.merge(user);
    entityManager.getTransaction().commit();

    entityManager.getTransaction().begin();
    entityManager.persist(new User("Monica", "[[email protected]](/web/20221128051852/https://www.baeldung.com/cdn-cgi/l/email-protection)"));
    entityManager.getTransaction().commit();

    // additional CRUD operations
}

private static class EntityManagerHolder {
    private static final EntityManager ENTITY_MANAGER = new JpaEntityManagerFactory(
      new Class[]{User.class})
      .getEntityManager();
}

public static EntityManager getJpaEntityManager() {
    return EntityManagerHolder.ENTITY_MANAGER;
}
```

## 5。结论

在本文中，我们展示了如何使用 JPA 的 `PersistenceUnitInfo`接口和 Hibernate 的`EntityManagerFactoryBuilderImpl`类的自定义实现，以编程方式引导 JPA 实体管理器，而不必依赖传统的 `“persistence.xml”`文件。

像往常一样，本文中显示的所有代码示例都可以在 GitHub 上获得。