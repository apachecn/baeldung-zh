# 用 Spring 引导 Hibernate 5

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/hibernate-5-spring>

## 1。概述

在本文中，我们将讨论如何使用 Java 和 XML 配置来用 Spring 引导 Hibernate 5。

本文主要关注 Spring MVC。我们的文章 [Spring Boot 与 Hibernate](/web/20220707035940/https://www.baeldung.com/spring-boot-hibernate) 描述了如何在 Spring Boot 使用 Hibernate。

## 2。弹簧集成

用原生 Hibernate API 引导一个`SessionFactory`有点复杂，会占用我们相当多的代码行(如果你真的需要这么做，看看[官方文档](https://web.archive.org/web/20220707035940/http://docs.jboss.org/hibernate/orm/5.2/userguide/html_single/Hibernate_User_Guide.html#bootstrap-native))。

幸运的是， **Spring 支持引导`SessionFactory`** `–` ，因此我们只需要几行 Java 代码或 XML 配置。

## 3。Maven 依赖关系

让我们首先向我们的`pom.xml`添加必要的依赖项:

```java
<dependency>
    <groupId>org.hibernate</groupId>
    <artifactId>hibernate-core</artifactId>
    <version>5.4.2.Final</version>
</dependency>
```

[spring-orm 模块](https://web.archive.org/web/20220707035940/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.springframework%22%20AND%20a%3A%22spring-orm%22)提供了与 Hibernate 的 spring 集成:

```java
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-orm</artifactId>
    <version>5.1.6.RELEASE</version>
</dependency>
```

为了简单起见，我们将使用 [H2](https://web.archive.org/web/20220707035940/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22com.h2database%22%20AND%20a%3A%22h2%22) 作为数据库:

```java
<dependency>
    <groupId>com.h2database</groupId> 
    <artifactId>h2</artifactId>
    <version>1.4.197</version>
</dependency>
```

最后，我们将使用 [Tomcat JDBC 连接池](https://web.archive.org/web/20220707035940/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.apache.tomcat%22%20AND%20a%3A%22tomcat-dbcp%22)，它比 Spring 提供的`DriverManagerDataSource`更适合生产目的:

```java
<dependency>
    <groupId>org.apache.tomcat</groupId>
    <artifactId>tomcat-dbcp</artifactId>
    <version>9.0.1</version>
</dependency>
```

## 4。配置

如前所述，Spring 支持我们引导 Hibernate `SessionFactory`。

我们所要做的就是**定义一些 beans 以及一些参数**。

有了 Spring，我们有两种配置选择，一种是基于 Java 的，另一种是基于 XML 的。

### 4.1。使用 Java 配置

在 Spring 上使用 Hibernate 5，从 Hibernate 4 的[到现在变化不大:我们不得不使用包`org.springframework.orm.hibernate5`中的`LocalSessionFactoryBean`，而不是`org.springframework.orm.hibernate4`。](/web/20220707035940/https://www.baeldung.com/hibernate-4-spring)

与之前的 Hibernate 4 一样，我们必须为`LocalSessionFactoryBean`、`DataSource`和`PlatformTransactionManager`定义 beans，以及一些特定于 Hibernate 的属性。

让我们创建我们的`HibernateConfig`类来**用 Spring** 配置 Hibernate 5:

```java
@Configuration
@EnableTransactionManagement
public class HibernateConf {

    @Bean
    public LocalSessionFactoryBean sessionFactory() {
        LocalSessionFactoryBean sessionFactory = new LocalSessionFactoryBean();
        sessionFactory.setDataSource(dataSource());
        sessionFactory.setPackagesToScan(
          {"com.baeldung.hibernate.bootstrap.model" });
        sessionFactory.setHibernateProperties(hibernateProperties());

        return sessionFactory;
    }

    @Bean
    public DataSource dataSource() {
        BasicDataSource dataSource = new BasicDataSource();
        dataSource.setDriverClassName("org.h2.Driver");
        dataSource.setUrl("jdbc:h2:mem:db;DB_CLOSE_DELAY=-1");
        dataSource.setUsername("sa");
        dataSource.setPassword("sa");

        return dataSource;
    }

    @Bean
    public PlatformTransactionManager hibernateTransactionManager() {
        HibernateTransactionManager transactionManager
          = new HibernateTransactionManager();
        transactionManager.setSessionFactory(sessionFactory().getObject());
        return transactionManager;
    }

    private final Properties hibernateProperties() {
        Properties hibernateProperties = new Properties();
        hibernateProperties.setProperty(
          "hibernate.hbm2ddl.auto", "create-drop");
        hibernateProperties.setProperty(
          "hibernate.dialect", "org.hibernate.dialect.H2Dialect");

        return hibernateProperties;
    }
}
```

### 4.2。使用 XML 配置

作为第二个选择，我们还可以用基于 XML 的配置来配置 Hibernate 5:

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans >

    <bean id="sessionFactory" 
      class="org.springframework.orm.hibernate5.LocalSessionFactoryBean">
        <property name="dataSource" 
          ref="dataSource"/>
        <property name="packagesToScan" 
          value="com.baeldung.hibernate.bootstrap.model"/>
        <property name="hibernateProperties">
            <props>
                <prop key="hibernate.hbm2ddl.auto">
                    create-drop
                </prop>
                <prop key="hibernate.dialect">
                    org.hibernate.dialect.H2Dialect
                </prop>
            </props>
        </property>
    </bean>

    <bean id="dataSource" 
      class="org.apache.tomcat.dbcp.dbcp2.BasicDataSource">
        <property name="driverClassName" value="org.h2.Driver"/>
        <property name="url" value="jdbc:h2:mem:db;DB_CLOSE_DELAY=-1"/>
        <property name="username" value="sa"/>
        <property name="password" value="sa"/>
    </bean>

    <bean id="txManager" 
      class="org.springframework.orm.hibernate5.HibernateTransactionManager">
        <property name="sessionFactory" ref="sessionFactory"/>
    </bean>
</beans>
```

正如我们很容易看到的，我们定义了与前面基于 Java 的配置完全相同的 beans 和参数。

**为了将 XML 引导到 Spring 上下文**，如果应用程序配置了 Java 配置，我们可以使用一个简单的 Java 配置文件:

```java
@Configuration
@EnableTransactionManagement
@ImportResource({"classpath:hibernate5Configuration.xml"})
public class HibernateXMLConf {
    //
}
```

或者，如果整个配置是纯 XML 的，我们可以简单地将 XML 文件提供给 Spring 上下文。

## 5。用途

至此，Hibernate 5 已经完全配置了 Spring，我们可以在任何需要的时候直接**注入原始 Hibernate `SessionFactory`** :

```java
public abstract class BarHibernateDAO {

    @Autowired
    private SessionFactory sessionFactory;

    // ...
}
```

## 6。支持的数据库

不幸的是，Hibernate 项目并没有提供一个官方的数据库支持列表。

也就是说，**很容易看出特定的数据库类型是否可能被支持**，我们可以看一下受支持方言的[列表](https://web.archive.org/web/20220707035940/http://docs.jboss.org/hibernate/orm/5.2/userguide/html_single/Hibernate_User_Guide.html#database-dialect)。

## 7 .**。结论**

在这个快速教程中，**我们用 Hibernate 5** 配置了 Spring 用 Java 和 XML 配置。

与往常一样，GitHub 上的[提供了示例的完整源代码。](https://web.archive.org/web/20220707035940/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-hibernate-5)