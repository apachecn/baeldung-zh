# 与 Spring 和 JPA 的交易

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/transaction-configuration-with-jpa-and-spring>

## 1。概述

本教程将讨论**配置 Spring 事务**的正确方法，如何使用`@Transactional`注释，以及常见的陷阱。

关于核心持久性配置的更深入的讨论，请查看带有 JPA 教程的[Spring](/web/20220617075734/https://www.baeldung.com/the-persistence-layer-with-spring-and-jpa "Spring 3 and JPA with Hibernate")。

基本上，有两种不同的方式来配置事务，注释和 AOP，每种方式都有自己的优点。我们将在这里讨论更常见的注释配置。

## 延伸阅读:

## [为测试配置单独的 Spring 数据源](/web/20220617075734/https://www.baeldung.com/spring-testing-separate-data-source)

A quick, practical tutorial on how to configure a separate data source for testing in a Spring application.[Read more](/web/20220617075734/https://www.baeldung.com/spring-testing-separate-data-source) →

## [使用 Spring Boot 加载初始数据的快速指南](/web/20220617075734/https://www.baeldung.com/spring-boot-data-sql-and-schema-sql)

A quick and practical example of using data.sql and schema.sql files in Spring Boot.[Read more](/web/20220617075734/https://www.baeldung.com/spring-boot-data-sql-and-schema-sql) →

## [显示来自 Spring Boot 的 Hibernate/JPA SQL 语句](/web/20220617075734/https://www.baeldung.com/sql-logging-spring-boot)

Learn how you can configure logging of the generated SQL statements in your Spring Boot application.[Read more](/web/20220617075734/https://www.baeldung.com/sql-logging-spring-boot) →

## 2。配置交易

Spring 3.1 引入了**的`@EnableTransactionManagement`注释**，我们可以在`@Configuration`类中使用它来实现事务支持:

```java
@Configuration
@EnableTransactionManagement
public class PersistenceJPAConfig{

   @Bean
   public LocalContainerEntityManagerFactoryBean
     entityManagerFactoryBean(){
      //...
   }

   @Bean
   public PlatformTransactionManager transactionManager(){
      JpaTransactionManager transactionManager
        = new JpaTransactionManager();
      transactionManager.setEntityManagerFactory(
        entityManagerFactoryBean().getObject() );
      return transactionManager;
   }
}
```

然而，**如果我们正在使用一个 Spring Boot 项目，并且在类路径上有一个 spring-data-*或 spring-tx 依赖项，那么事务管理将默认启用**。

## 3。用 XML 配置交易

对于 3.1 之前的版本，或者如果 Java 不是一个选项，下面是使用`annotation-driven`和名称空间支持的 XML 配置:

```java
<bean id="txManager" class="org.springframework.orm.jpa.JpaTransactionManager">
   <property name="entityManagerFactory" ref="myEmf" />
</bean>
<tx:annotation-driven transaction-manager="txManager" />
```

## 4。`@Transactional`注解

配置好事务后，我们现在可以在类或方法级别用`@Transactional`注释 bean:

```java
@Service
@Transactional
public class FooService {
    //...
}
```

注释还支持**进一步配置**:

*   交易的`Propagation Type`
*   交易的`Isolation Level`
*   事务包装的操作的一个`Timeout`
*   一个`readOnly flag`–提示持久化提供者事务应该是只读的
*   `Rollback`交易规则

请注意，默认情况下，回滚只在运行时发生，未检查的异常除外。**被检查的异常不触发事务的回滚**。当然，我们可以用`rollbackFor`和`noRollbackFor`注释参数来配置这种行为。

## 5。潜在陷阱

### 5.1。交易和代理

在高层次上， **Spring 为所有用`@Transactional`** 标注的类创建代理，或者在类上，或者在任何方法上。代理允许框架在运行方法之前和之后注入事务逻辑，主要用于启动和提交事务。

需要记住的重要一点是，如果事务性 bean 实现了一个接口，默认情况下代理将是一个 Java 动态代理。这意味着只有通过代理进入的外部方法调用才会被拦截。**任何自调用调用都不会启动任何事务，**，即使该方法有`@Transactional`注释。

使用代理的另一个警告是**只有公共方法应该用`@Transactional.`** 注释，任何其他可见性的方法将简单地忽略注释，因为它们没有被代理。

### 5.2。改变隔离等级

```java
courseDao.createWithRuntimeException(course);
```

我们还可以更改事务隔离级别:

```java
@Transactional(isolation = Isolation.SERIALIZABLE)
```

注意这实际上已经在 Spring 4.1 中[引入了](https://web.archive.org/web/20220617075734/https://jira.spring.io/browse/SPR-5012)；如果我们在 Spring 4.1 之前运行上面的例子，它将导致:

> `org.springframework.transaction.InvalidIsolationLevelException`:标准 JPA 不支持定制隔离级别——为您的 JPA 实现使用一个特殊的`JpaDialect`

### 5.3。只读交易

`readOnly`标志通常会产生混淆，尤其是在使用 JPA 时。从 Javadoc:

> 这只是作为实际事务子系统的一个提示；它将`not necessarily`导致写访问尝试失败。不能解释只读提示的事务管理器将在被请求只读事务时抛出一个异常。

事实是**当`readOnly`标志被设置时，我们不能确定插入或更新不会发生。**这种行为依赖于供应商，而 JPA 是独立于供应商的。

理解**`readOnly`标志只与事务内部相关也很重要。**如果一个操作发生在事务上下文之外，这个标志就会被忽略。一个简单的例子是调用一个用以下内容注释的方法:

```java
@Transactional( propagation = Propagation.SUPPORTS,readOnly = true )
```

在非事务上下文中，不会创建事务，并且会忽略`readOnly`标志。

### 5.4。交易记录

理解事务相关问题的一个有用方法是微调事务包中的日志记录。Spring 中相关的包是“`org.springframework.transaction”`，应该配置一个 `TRACE.` 的日志级别

### 5.5.事务回滚

`@Transactional`注释是指定方法上的事务语义的元数据。我们有两种回滚事务的方法:声明式和编程式。

在**声明性方法中，我们用`@`** `**Transactional**` **注释**来注释方法。`@Transactional`注释使用属性`rollbackFor`或`rollbackForClassName`回滚事务，使用属性`noRollbackFor`或`noRollbackForClassName`避免回滚列出的异常。

声明性方法中的默认回滚行为将在运行时异常时回滚。

让我们看一个简单的例子，使用声明性方法回滚运行时异常或错误的事务:

```java
@Transactional
public void createCourseDeclarativeWithRuntimeException(Course course) {
    courseDao.create(course);
    throw new DataIntegrityViolationException("Throwing exception for demoing Rollback!!!");
}
```

接下来，我们将使用声明性方法为列出的已检查异常回滚事务。中的 回滚 中的 我们的 例子 是 上的 `SQLException`:

```java
@Transactional(rollbackFor = { SQLException.class })
public void createCourseDeclarativeWithCheckedException(Course course) throws SQLException {
    courseDao.create(course);
    throw new SQLException("Throwing exception for demoing rollback");
}
```

让我们看看属性`noRollbackFor`在声明性方法中的简单使用，以防止所列异常的事务回滚:

```java
@Transactional(noRollbackFor = { SQLException.class })
public void createCourseDeclarativeWithNoRollBack(Course course) throws SQLException {
    courseDao.create(course);
    throw new SQLException("Throwing exception for demoing rollback");
}
```

在**编程方式** **中，我们使用`TransactionAspectSupport`** 回滚事务:

```java
public void createCourseDefaultRatingProgramatic(Course course) {
    try {
       courseDao.create(course);
    } catch (Exception e) {
       TransactionAspectSupport.currentTransactionStatus().setRollbackOnly();
    }
}
```

The **d****eclarative rollback strategy should be favored over the programmatic rollback strategy**.

## 6。结论

在本文中，我们介绍了使用 Java 和 XML 的事务语义的基本配置。我们还学习了如何使用`@Transactional,`和交易策略的最佳实践。

和往常一样，本文中的代码可以从 Github 上的[处获得。](https://web.archive.org/web/20220617075734/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-jpa-2)