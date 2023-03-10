# Spring Boot 对 jOOQ 的支持

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-support-for-jooq>

## 1。概述

本教程是 jOOQ 与 Spring 简介文章的后续，涵盖了 jOOQ 在 Spring Boot 应用程序中的使用方式。

如果您还没有阅读过该教程，请阅读一下，并按照第 2 节中关于 Maven 依赖性和第 3 节中关于代码生成的说明进行操作。这将为表示示例数据库中的表的 Java 类生成源代码，包括`Author`、`Book`和`AuthorBook`。

## 2。Maven 配置

除了上一篇教程中的依赖项和插件，Maven POM 文件中还需要包含其他几个组件，以使 jOOQ 能够与 Spring Boot 一起工作。

### 2.1。依赖性管理

使用 Spring Boot 最常见的方式是通过在`parent`元素中声明来继承`spring-boot-starter-parent`项目。然而，这种方法并不总是合适的，因为它强加了一个继承链，这在许多情况下可能不是用户想要的。

本教程使用另一种方法:将依赖性管理委托给 Spring Boot。要实现这一点，只需将下面的`dependencyManagement`元素添加到 POM 文件中:

```java
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-dependencies</artifactId>
            <version>2.2.6.RELEASE</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

### 2.2。依赖性

为了让 Spring Boot 控制 jOOQ，需要声明对`spring-boot-starter-jooq`工件的依赖:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jooq</artifactId>
    <version>2.2.6.RELEASE</version>
</dependency>
```

请注意，本文关注的是 jOOQ 的开源发行版。如果你想使用商业发行版，可以在官方博客上查看使用 jOOQ 商业发行版和 Spring Boot 的指南。

## 3。Spring Boot 配置

### 3.1.初始引导配置

在我们得到 jOOQ 的支持之前，我们要开始准备 Spring Boot 的事情。

首先，我们将利用引导中的持久性支持和改进，以及标准`application.properties`文件中的数据访问信息。这样，我们可以跳过定义 beans 并通过单独的属性文件进行配置。

我们将在这里添加 URL 和凭证来定义我们的嵌入式 H2 数据库:

```java
spring.datasource.url=jdbc:h2:~/jooq
spring.datasource.username=sa
spring.datasource.password=
```

我们还将定义一个简单的引导应用程序:

```java
@SpringBootApplication
@EnableTransactionManagement
public class Application {

}
```

我们将让这个简单的空的，我们将在另一个配置类中定义所有其他的 bean 声明-`InitialConfiguration`。

### 3.2。Bean 配置

现在让我们定义这个`InitialConfiguration`类:

```java
@Configuration
public class InitialConfiguration {
    // Other declarations
}
```

Spring Boot 已经基于`application.properties`文件中设置的属性自动生成并配置了`dataSource` bean，因此我们不需要手动注册它。下面的代码让自动配置的`DataSource` bean 注入到一个字段中，并展示了如何使用这个 bean:

```java
@Autowired
private DataSource dataSource;

@Bean
public DataSourceConnectionProvider connectionProvider() {
    return new DataSourceConnectionProvider
      (new TransactionAwareDataSourceProxy(dataSource));
}
```

由于名为`transactionManager`的 bean 已经由 Spring Boot 自动创建和配置，我们不需要像在前面的教程中那样声明任何其他的`DataSourceTransactionManager`类型的 bean 来利用 Spring 事务支持。

创建`DSLContext` bean 的方式与前面教程的`PersistenceContext`类相同:

```java
@Bean
public DefaultDSLContext dsl() {
    return new DefaultDSLContext(configuration());
}
```

最后，需要向`DSLContext`提供一个`Configuration`实现。由于 Spring Boot 能够通过类路径上存在的 H2 工件来识别正在使用的 SQL 方言，因此不再需要方言配置:

```java
public DefaultConfiguration configuration() {
    DefaultConfiguration jooqConfiguration = new DefaultConfiguration();
    jooqConfiguration.set(connectionProvider());
    jooqConfiguration
      .set(new DefaultExecuteListenerProvider(exceptionTransformer()));

    return jooqConfiguration;
}
```

## 4。通过 jOOQ 使用 Spring Boot

为了使 Spring Boot 对 jOOQ 支持的演示更容易理解，本教程前传中的测试用例被重用，对其类级注释做了一点修改:

```java
@SpringApplicationConfiguration(Application.class)
@Transactional("transactionManager")
@RunWith(SpringJUnit4ClassRunner.class)
public class SpringBootTest {
    // Other declarations
}
```

很明显，Spring Boot 没有采用`@ContextConfiguration`注释，而是使用`@SpringApplicationConfiguration`来利用`SpringApplicationContextLoader`上下文加载器来测试应用程序。

插入、更新和删除数据的测试方法与前面的教程完全相同。请查看那篇文章的第 5 部分，了解更多信息。所有的测试都应该在新的配置下成功执行，证明 jOOQ 完全受 Spring Boot 支持。

## 5。结论

本教程深入探讨了 jOOQ 与 Spring 的结合使用。它介绍了 Spring Boot 应用程序利用 jOOQ 以类型安全的方式与数据库交互的方法。

所有这些例子和代码片段的实现都可以在 GitHub 项目中找到。