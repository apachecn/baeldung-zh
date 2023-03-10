# 使用 Spring 的 JPA 指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/the-persistence-layer-with-spring-and-jpa>

## 1。概述

本教程**展示了如何用 JPA** 设置 Spring，使用 Hibernate 作为持久性提供者。

关于使用基于 Java 的配置和项目的基本 Maven pom 设置 Spring 上下文的分步介绍，请参见本文。

我们将从在 Spring Boot 项目中设置 JPA 开始。然后，如果我们有一个标准的 Spring 项目，我们将研究我们需要的完整配置。

## 延伸阅读:

## [定义 JPA 实体](/web/20220813062407/https://www.baeldung.com/jpa-entities)

Learn how to define entities and customize them using the Java Persistence API.[Read more](/web/20220813062407/https://www.baeldung.com/jpa-entities) →

## [Spring Boot 与冬眠](/web/20220813062407/https://www.baeldung.com/spring-boot-hibernate)

A quick, practical intro to integrating Spring Boot and Hibernate/JPA.[Read more](/web/20220813062407/https://www.baeldung.com/spring-boot-hibernate) →

这里有一个用 Spring 4 设置 Hibernate 4 的视频(我们建议看完整的 1080p):

[https://web.archive.org/web/20220813062407if_/https://www.youtube.com/embed/7jdFEdrfh9Q?feature=oembed](https://web.archive.org/web/20220813062407if_/https://www.youtube.com/embed/7jdFEdrfh9Q?feature=oembed)

## 2。Spring Boot 的 JPA

Spring Boot 项目旨在让创建 Spring 应用程序变得更快更容易。这是通过使用启动器和各种 Spring 功能的自动配置来实现的，JPA 就是其中之一。

### 2.1.Maven 依赖性

为了在 Spring Boot 应用程序中启用 JPA，我们需要 `[spring-boot-starter](https://web.archive.org/web/20220813062407/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22spring-boot-starter%22%20AND%20g%3A%22org.springframework.boot%22)`和 `[spring-boot-starter-data-jpa](https://web.archive.org/web/20220813062407/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22spring-boot-starter-data-jpa%22)` 依赖关系:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter</artifactId>
    <version>2.2.6.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
    <version>2.2.6.RELEASE</version>
</dependency>
```

`spring-boot-starter`包含 Spring JPA 必需的自动配置。另外，`spring-boot-starter-jpa`项目引用了所有必要的依赖项，比如`hibernate-core`。

### 2.2.配置

**Spring Boot 将`Hibernate`配置为默认的 JPA 提供者**，所以不再需要定义`entityManagerFactory` bean，除非我们想要定制它。

**根据我们使用的数据库，Spring Boot 还可以自动配置`dataSource` bean。**对于类型为`H2`、`HSQLDB`和`Apache Derby`的内存数据库，如果类路径中存在相应的数据库依赖关系，Boot 会自动配置`DataSource`。

例如，如果我们想在 Spring Boot JPA 应用程序中使用内存中的`H2`数据库，我们只需要将 [`h2`](https://web.archive.org/web/20220813062407/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22h2%22%20AND%20g%3A%22com.h2database%22) 依赖项添加到`pom.xml`文件中:

```java
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <version>1.4.200</version>
</dependency>
```

这样，我们不需要定义`dataSource` bean，但是如果我们想定制它，我们可以这样做。

如果我们想对数据库使用 JPA，我们需要依赖关系。我们还需要定义`DataSource`配置。

我们可以在一个`@Configuration`类中或者通过使用标准的 Spring Boot 属性来做到这一点。

Java 配置看起来与它在标准 Spring 项目中的配置一样:

```java
@Bean
public DataSource dataSource() {
    DriverManagerDataSource dataSource = new DriverManagerDataSource();

    dataSource.setDriverClassName("com.mysql.cj.jdbc.Driver");
    dataSource.setUsername("mysqluser");
    dataSource.setPassword("mysqlpass");
    dataSource.setUrl(
      "jdbc:mysql://localhost:3306/myDb?createDatabaseIfNotExist=true"); 

    return dataSource;
}
```

**要使用属性文件配置数据源，我们必须设置前缀为`spring.datasource`** 的属性:

```java
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.datasource.username=mysqluser
spring.datasource.password=mysqlpass
spring.datasource.url=
  jdbc:mysql://localhost:3306/myDb?createDatabaseIfNotExist=true
```

Spring Boot 将根据这些属性自动配置数据源。

同样在 Spring Boot 1 中，默认的连接池是`Tomcat`，但是在 Spring Boot 2 中已经被改成了`HikariCP`。

在 GitHub 项目中，我们有更多在 Spring Boot 配置 JPA 的例子。

正如我们所看到的，如果我们使用 Spring Boot，基本的 JPA 配置相当简单。

然而，如果我们有一个标准的 Spring 项目，我们需要更明确的配置，使用 Java 或 XML。这是我们将在接下来的章节中重点关注的内容。

## 3。在非引导项目中用 Java 配置 JPA Spring

为了在 Spring 项目中使用 JPA，**我们需要设置`EntityManager`。**

这是配置的主要部分，我们可以通过 Spring factory bean 来完成。这可以是更简单的`LocalEntityManagerFactoryBean`或**更灵活的`LocalContainerEntityManagerFactoryBean`。**

让我们看看如何使用后一个选项:

```java
@Configuration
@EnableTransactionManagement
public class PersistenceJPAConfig{

   @Bean
   public LocalContainerEntityManagerFactoryBean entityManagerFactory() {
      LocalContainerEntityManagerFactoryBean em 
        = new LocalContainerEntityManagerFactoryBean();
      em.setDataSource(dataSource());
      em.setPackagesToScan(new String[] { "com.baeldung.persistence.model" });

      JpaVendorAdapter vendorAdapter = new HibernateJpaVendorAdapter();
      em.setJpaVendorAdapter(vendorAdapter);
      em.setJpaProperties(additionalProperties());

      return em;
   }

   // ...

}
```

**我们还需要显式定义上面使用的`DataSource` bean** :

```java
@Bean
public DataSource dataSource(){
    DriverManagerDataSource dataSource = new DriverManagerDataSource();
    dataSource.setDriverClassName("com.mysql.cj.jdbc.Driver");
    dataSource.setUrl("jdbc:mysql://localhost:3306/spring_jpa");
    dataSource.setUsername( "tutorialuser" );
    dataSource.setPassword( "tutorialmy5ql" );
    return dataSource;
}
```

配置的最后一部分是附加的 Hibernate 属性以及`TransactionManager`和`exceptionTranslation`bean:

```java
@Bean
public PlatformTransactionManager transactionManager() {
    JpaTransactionManager transactionManager = new JpaTransactionManager();
    transactionManager.setEntityManagerFactory(entityManagerFactory().getObject());

    return transactionManager;
}

@Bean
public PersistenceExceptionTranslationPostProcessor exceptionTranslation(){
    return new PersistenceExceptionTranslationPostProcessor();
}

Properties additionalProperties() {
    Properties properties = new Properties();
    properties.setProperty("hibernate.hbm2ddl.auto", "create-drop");
    properties.setProperty("hibernate.dialect", "org.hibernate.dialect.MySQL5Dialect");

    return properties;
}
```

## 4。带有 XML 的 JPA Spring 配置

接下来，让我们看看同样的带有 XML 的 Spring 配置:

```java
<bean id="myEmf" 
  class="org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean">
    <property name="dataSource" ref="dataSource" />
    <property name="packagesToScan" value="com.baeldung.persistence.model" />
    <property name="jpaVendorAdapter">
        <bean class="org.springframework.orm.jpa.vendor.HibernateJpaVendorAdapter" />
    </property>
    <property name="jpaProperties">
        <props>
            <prop key="hibernate.hbm2ddl.auto">create-drop</prop>
            <prop key="hibernate.dialect">org.hibernate.dialect.MySQL5Dialect</prop>
        </props>
    </property>
</bean>

<bean id="dataSource" 
  class="org.springframework.jdbc.datasource.DriverManagerDataSource">
    <property name="driverClassName" value="com.mysql.cj.jdbc.Driver" />
    <property name="url" value="jdbc:mysql://localhost:3306/spring_jpa" />
    <property name="username" value="tutorialuser" />
    <property name="password" value="tutorialmy5ql" />
</bean>

<bean id="transactionManager" class="org.springframework.orm.jpa.JpaTransactionManager">
    <property name="entityManagerFactory" ref="myEmf" />
</bean>
<tx:annotation-driven />

<bean id="persistenceExceptionTranslationPostProcessor" class=
  "org.springframework.dao.annotation.PersistenceExceptionTranslationPostProcessor" />
```

XML 和新的基于 Java 的配置之间的差别相对较小。也就是说，在 XML 中，对另一个 bean 的引用可以指向该 bean 或该 bean 的 bean 工厂。

但是在 Java 中，由于类型不同，编译器不允许这样做，所以首先从 bean 工厂中检索`EntityManagerFactory` ,然后传递给事务管理器:

```java
transactionManager.setEntityManagerFactory(entityManagerFactory().getObject());
```

## 5。走向完全无 XML

通常，JPA 通过`META-INF/persistence.xml`文件定义一个持久化单元。**从弹簧 3.1 开始，不再需要`persistence.xml`。**`LocalContainerEntityManagerFactoryBean`现在支持一个`packagesToScan`属性，其中可以指定扫描`@Entity`类的包。

这个文件是我们需要删除的最后一段 XML。我们现在可以完全不用 XML 来设置 JPA。

我们通常会在`persistence.xml`文件中指定 JPA 属性。

或者，我们可以将属性直接添加到实体管理器工厂 bean 中:

```java
factoryBean.setJpaProperties(this.additionalProperties());
```

顺便提一下，如果 Hibernate 是持久性提供者，这也是指定特定于 Hibernate 的属性的方法。

## 6。Maven 配置

除了 Spring 核心和持久性依赖——在 [Spring with Maven 教程](/web/20220813062407/https://www.baeldung.com/spring-with-maven "Spring Maven dependencies")中有详细介绍——我们还需要在项目中定义 JPA 和 Hibernate 以及 MySQL 连接器:

```java
<dependency>
   <groupId>org.hibernate</groupId>
   <artifactId>hibernate-core</artifactId>
   <version>5.2.17.Final</version>
   <scope>runtime</scope>
</dependency>

<dependency>
   <groupId>mysql</groupId>
   <artifactId>mysql-connector-java</artifactId>
   <version>8.0.19</version>
   <scope>runtime</scope>
</dependency>
```

请注意，MySQL 依赖项在这里是作为一个例子包含在内的。我们需要一个驱动程序来配置数据源，但是任何支持 Hibernate 的数据库都可以。

## 7。结论

本教程演示了如何在 Spring Boot 和标准 Spring 应用程序中使用 Spring 中的 Hibernate 来配置 **JPA。**

和往常一样，本文中的代码可以从 GitHub 上的[处获得。](https://web.archive.org/web/20220813062407/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-jpa-2)