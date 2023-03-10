# 用春天冬眠 3

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/hibernate3-spring>

## 1。概述

本文将关注如何用 Spring 设置**Hibernate 3——我们将看看如何使用 XML 和 Java 配置来用 Hibernate 3 和 MySQL 设置 Spring。**

更新:本文主要关注 Hibernate 3。如果你正在寻找 Hibernate 的当前版本-[,这就是本文关注的内容](/web/20220815044744/https://www.baeldung.com/hibernate-5-spring)。

## 2。Java **Spring 配置为 Hibernate 3**

用 Spring 和 Java config 设置 Hibernate 3 很简单:

```java
import java.util.Properties;
import javax.sql.DataSource;
import org.apache.tomcat.dbcp.dbcp.BasicDataSource;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.PropertySource;
import org.springframework.core.env.Environment;
import org.springframework.dao.annotation.PersistenceExceptionTranslationPostProcessor;
import org.springframework.orm.hibernate3.HibernateTransactionManager;
import org.springframework.orm.hibernate3.annotation.AnnotationSessionFactoryBean;
import org.springframework.transaction.annotation.EnableTransactionManagement;
import com.google.common.base.Preconditions;

@Configuration
@EnableTransactionManagement
@PropertySource({ "classpath:persistence-mysql.properties" })
@ComponentScan({ "com.baeldung.spring.persistence" })
public class PersistenceConfig {

   @Autowired
   private Environment env;

   @Bean
   public AnnotationSessionFactoryBean sessionFactory() {
      AnnotationSessionFactoryBean sessionFactory = new AnnotationSessionFactoryBean();
      sessionFactory.setDataSource(restDataSource());
      sessionFactory.setPackagesToScan(new String[] { "com.baeldung.spring.persistence.model" });
      sessionFactory.setHibernateProperties(hibernateProperties());

      return sessionFactory;
   }

   @Bean
   public DataSource restDataSource() {
      BasicDataSource dataSource = new BasicDataSource();
      dataSource.setDriverClassName(env.getProperty("jdbc.driverClassName"));
      dataSource.setUrl(env.getProperty("jdbc.url"));
      dataSource.setUsername(env.getProperty("jdbc.user"));
      dataSource.setPassword(env.getProperty("jdbc.pass"));

      return dataSource;
   }

   @Bean
   @Autowired
   public HibernateTransactionManager transactionManager(SessionFactory sessionFactory) {
      HibernateTransactionManager txManager = new HibernateTransactionManager();
      txManager.setSessionFactory(sessionFactory);

      return txManager;
   }

   @Bean
   public PersistenceExceptionTranslationPostProcessor exceptionTranslation() {
      return new PersistenceExceptionTranslationPostProcessor();
   }

   Properties hibernateProperties() {
      return new Properties() {
         {
            setProperty("hibernate.hbm2ddl.auto", env.getProperty("hibernate.hbm2ddl.auto"));
            setProperty("hibernate.dialect", env.getProperty("hibernate.dialect"));
         }
      };
   }
}
```

与下面描述的 XML 配置相比，配置中的一个 bean 访问另一个 bean 的方式略有不同。在 XML 中，**指向一个 bean 或者指向一个能够创建那个 bean** 的 bean 工厂没有区别。由于 Java 配置是类型安全的——直接指向 bean 工厂不再是一种选择——我们需要手动从 bean 工厂中检索 bean:

```java
txManager.setSessionFactory(sessionFactory().getObject());
```

## 3。Hibernate 3 的 XML Spring 配置

类似地，我们也可以用 XML 配置来设置 **Hibernate 3:**

```java
<context:property-placeholder location="classpath:persistence-mysql.properties" />

<bean id="sessionFactory" 
  class="org.springframework.orm.hibernate3.annotation.AnnotationSessionFactoryBean">
    <property name="dataSource" ref="dataSource" />
    <property name="packagesToScan" value="com.baeldung.spring.persistence.model" />
    <property name="hibernateProperties">
        <props>
            <prop key="hibernate.hbm2ddl.auto">${hibernate.hbm2ddl.auto}</prop>
            <prop key="hibernate.dialect">${hibernate.dialect}</prop>
        </props>
    </property>
</bean>

<bean id="dataSource" 
  class="org.apache.tomcat.dbcp.dbcp.BasicDataSource">
    <property name="driverClassName" value="${jdbc.driverClassName}" />
    <property name="url" value="${jdbc.url}" />
    <property name="username" value="${jdbc.user}" />
    <property name="password" value="${jdbc.pass}" />
</bean>

<bean id="txManager" 
  class="org.springframework.orm.hibernate3.HibernateTransactionManager">
    <property name="sessionFactory" ref="sessionFactory" />
</bean>

<bean id="persistenceExceptionTranslationPostProcessor" 
  class="org.springframework.dao.annotation.PersistenceExceptionTranslationPostProcessor"/>
```

然后，使用一个`@Configuration`类将这个 XML 文件引导到 Spring 上下文中:

```java
@Configuration
@EnableTransactionManagement
@ImportResource({ "classpath:persistenceConfig.xml" })
public class PersistenceXmlConfig {
   //
}
```

对于这两种类型的配置，特定于 JDBC 和 Hibernate 的属性都存储在属性文件中:

```java
# jdbc.X
jdbc.driverClassName=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/spring_hibernate_dev?createDatabaseIfNotExist=true
jdbc.user=tutorialuser
jdbc.pass=tutorialmy5ql
# hibernate.X
hibernate.dialect=org.hibernate.dialect.MySQL5Dialect
hibernate.show_sql=false
hibernate.hbm2ddl.auto=create-drop
```

## 4。Spring、Hibernate 和 MySQL

上面的例子使用 MySQL 5 作为用 Hibernate 配置的底层数据库——然而，Hibernate 支持几个底层的 [SQL 数据库](https://web.archive.org/web/20220815044744/https://developer.jboss.org/docs/DOC-13921 "Hibernate Supported Databases")。

### 4.1。司机

驱动程序类名通过提供给数据源的**`jdbc.driverClassName`属性**来配置。

在上面的例子中，它从我们在 pom 中定义的`mysql-connector-java`依赖项被设置为`com.mysql.jdbc.Driver`，在文章的开始。

### 4.2。方言

方言是通过**提供给休眠`SessionFactory`的`hibernate.dialect`属性**来配置的。

在上面的例子中，它被设置为`org.hibernate.dialect.MySQL5Dialect`，因为我们使用 MySQL 5 作为底层数据库。支持 MySQL 的还有几种**其他方言:**

*   `**org.hibernate.dialect.MySQL5InnoDBDialect**`–适用于采用 InnoDB 存储引擎的 MySQL 5.x
*   `**org.hibernate.dialect.MySQLDialect**`–适用于 5.x 之前的 MySQL
*   `**org.hibernate.dialect.MySQLInnoDBDialect**`–适用于使用 InnoDB 存储引擎的 MySQL 5 . x 之前版本
*   `**org.hibernate.dialect.MySQLMyISAMDialect**`–适用于所有采用 ISAM 存储引擎的 MySQL 版本

Hibernate [支持每一个支持的数据库的 SQL 方言](https://web.archive.org/web/20220815044744/http://docs.jboss.org/hibernate/core/3.6/reference/en-US/html/session-configuration.html#configuration-optional-dialects "Hibernate Supported Dialects")。

## 5。用途

至此，Hibernate 3 已经完全配置了 Spring，我们可以在需要的时候直接**注入原始 Hibernate** `SessionFactory`:

```java
public abstract class FooHibernateDAO{

   @Autowired
   SessionFactory sessionFactory;

   ...

   protected Session getCurrentSession(){
      return sessionFactory.getCurrentSession();
   }
}
```

## 6 号。肚子

要将 Spring 持久性依赖项添加到 pom 中，请参见带有 Maven 示例的[Spring](/web/20220815044744/https://www.baeldung.com/spring-with-maven#persistence "Spring and Maven - Persistence")——我们需要定义`spring-context`和`spring-orm`。

继续 Hibernate 3，Maven 的依赖性很简单:

```java
<dependency>
   <groupId>org.hibernate</groupId>
   <artifactId>hibernate-core</artifactId>
   <version>3.6.10.Final</version>
</dependency>
```

然后，为了使 Hibernate 能够使用它的代理模型，我们还需要`javassist`:

```java
<dependency>
   <groupId>org.javassist</groupId>
   <artifactId>javassist</artifactId>
   <version>3.18.2-GA</version>
</dependency>
```

在本教程中，我们将使用 MySQL 作为我们的数据库，因此我们还需要:

```java
<dependency>
   <groupId>mysql</groupId>
   <artifactId>mysql-connector-java</artifactId>
   <version>5.1.32</version>
   <scope>runtime</scope>
</dependency>
```

最后，我们将不会使用 Spring 数据源实现——`DriverManagerDataSource`；相反，我们将使用一个生产就绪的连接池解决方案—Tomcat JDBC 连接池:

```java
<dependency>
    <groupId>org.apache.tomcat</groupId>
    <artifactId>tomcat-dbcp</artifactId>
    <version>7.0.55</version>
</dependency>
```

## 7。结论

在这个例子中，**我们用 Spring** 配置了 Hibernate 3——既有 Java 也有 XML 配置。这个简单项目的实现可以在[的 GitHub 项目](https://web.archive.org/web/20220815044744/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-hibernate-3 "Spring with Hibernate 3 Project on Github")中找到——这是一个基于 Maven 的项目，所以它应该很容易导入和运行。