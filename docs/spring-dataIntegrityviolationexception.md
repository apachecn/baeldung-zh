# spring data integrityviolationexception

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-dataIntegrityviolationexception>

## 1。概述

在本文中，我们将讨论 Spring`**org.springframework.dao.DataIntegrityViolationException**`——这是一个一般的数据异常，通常由 Spring 异常翻译机制在处理较低级别的持久性异常时抛出。本文将讨论这种异常最常见的原因以及针对每种原因的解决方案。

## 延伸阅读:

## [Spring Data Java 8 支持](/web/20221129000518/https://www.baeldung.com/spring-data-java-8)

A quick and practical guide to Java 8 support in Spring Data.[Read more](/web/20221129000518/https://www.baeldung.com/spring-data-java-8) →

## [弹簧数据标注](/web/20221129000518/https://www.baeldung.com/spring-data-annotations)

Learn about the most important annotations we need to handle persistence using the Spring Data project[Read more](/web/20221129000518/https://www.baeldung.com/spring-data-annotations) →

## [在 Spring Data REST 中处理关系](/web/20221129000518/https://www.baeldung.com/spring-data-rest-relationships)

A practical guide to working with entity relationships in Spring Data REST.[Read more](/web/20221129000518/https://www.baeldung.com/spring-data-rest-relationships) →

## 2。`DataIntegrityViolationException`和春异常翻译

Spring 异常翻译机制可以透明地应用于所有用`@Repository`注释的 bean——通过在上下文中定义异常翻译 bean 后处理器 bean:

```java
<bean id="persistenceExceptionTranslationPostProcessor" 
   class="org.springframework.dao.annotation.PersistenceExceptionTranslationPostProcessor" />
```

或者在 Java 中:

```java
@Configuration
public class PersistenceHibernateConfig{
   @Bean
   public PersistenceExceptionTranslationPostProcessor exceptionTranslation(){
      return new PersistenceExceptionTranslationPostProcessor();
   }
}
```

在 Spring 中可用的较旧的持久性模板上，异常转换机制也是默认启用的 HibernateTemplate、JpaTemplate 等。

## 3。`DataIntegrityViolationException`被扔在哪里

#### 3.1。`DataIntegrityViolationException`同冬眠

当 Spring 配置了 Hibernate 时，在 Spring 提供的异常翻译层中抛出`exception`–`SessionFactoryUtils – convertHibernateAccessException`。

有三种可能的休眠异常会导致`DataIntegrityViolationException`被抛出:

*   `org.hibernate.exception.ConstraintViolationException`
*   `org.hibernate.PropertyValueException`
*   `org.hibernate.exception.DataException`

#### 3.2。`DataIntegrityViolationException`与 JPA

当 Spring 被配置为 JPA 作为其持久性提供者时，类似于 Hibernate，在异常转换层中抛出`DataIntegrityViolationException`——即在`EntityManagerFactoryUtils – convertJpaAccessExceptionIfPossible`中。

有一个 JPA 异常可能会触发一个`DataIntegrityViolationException`被抛出，即`javax.persistence.EntityExistsException`。

## 4。`org.hibernate.exception.ConstraintViolationException`起因:

这是导致`DataIntegrityViolationException`被抛出的最常见原因——休眠`ConstraintViolationException`表明操作违反了数据库完整性约束。

考虑以下示例——对于通过`Parent`和`Child`实体之间的显式外键列的一对一映射，以下操作应该会失败:

```java
@Test(expected = DataIntegrityViolationException.class)
public void whenChildIsDeletedWhileParentStillHasForeignKeyToIt_thenDataException() {
   Child childEntity = new Child();
   childService.create(childEntity);

   Parent parentEntity = new Parent(childEntity);
   service.create(parentEntity);

   childService.delete(childEntity);
}
```

`Parent`实体有一个到`Child`实体的外键——因此删除子实体将打破父实体的外键约束——这将导致一个`ConstraintViolationException`——被 Spring 包装在`DataIntegrityViolationException`中:

```java
org.springframework.dao.DataIntegrityViolationException: 
could not execute statement; SQL [n/a]; constraint [null]; 
nested exception is org.hibernate.exception.ConstraintViolationException: could not execute statement
    at o.s.orm.h.SessionFactoryUtils.convertHibernateAccessException(SessionFactoryUtils.java:138)
Caused by: org.hibernate.exception.ConstraintViolationException: could not execute statement
```

为了解决这个问题，应该首先删除`Parent`:

```java
@Test
public void whenChildIsDeletedAfterTheParent_thenNoExceptions() {
   Child childEntity = new Child();
   childService.create(childEntity);

   Parent parentEntity = new Parent(childEntity);
   service.create(parentEntity);

   service.delete(parentEntity);
   childService.delete(childEntity);
}
```

## 5。`org.hibernate.PropertyValueException`起因:

这是`DataIntegrityViolationException`更常见的原因之一——在 Hibernate 中，这将归结为一个存在问题的实体。要么实体具有用 **`not-null`约束**定义的空属性，要么实体的关联可能引用未保存的**，瞬态实例**。

例如，以下实体有一个非空的`name`属性

```java
@Entity
public class Foo {
   ...

   @Column(nullable = false)
   private String name;

   ...
}
```

如果下面的测试试图持久化具有空值的实体`name`:

```java
@Test(expected = DataIntegrityViolationException.class)
public void whenInvalidEntityIsCreated_thenDataException() {
   fooService.create(new Foo());
}
```

违反了数据库完整性约束，因此抛出了`DataIntegrityViolationException`:

```java
org.springframework.dao.DataIntegrityViolationException: 
not-null property references a null or transient value: 
org.baeldung.spring.persistence.model.Foo.name; 
nested exception is org.hibernate.PropertyValueException: 
not-null property references a null or transient value: 
org.baeldung.spring.persistence.model.Foo.name
	at o.s.orm.h.SessionFactoryUtils.convertHibernateAccessException(SessionFactoryUtils.java:160)
...
Caused by: org.hibernate.PropertyValueException: 
not-null property references a null or transient value: 
org.baeldung.spring.persistence.model.Foo.name
	at o.h.e.i.Nullability.checkNullability(Nullability.java:103)
...
```

## 6。`org.hibernate.exception.DataException`起因:

Hibernate `DataException`表示无效的 SQL 语句——在特定的上下文中，语句或数据有问题。例如，使用之前的或`Foo`实体，下面将触发这个异常:

```java
@Test(expected = DataIntegrityViolationException.class)
public final void whenEntityWithLongNameIsCreated_thenDataException() {
   service.create(new Foo(randomAlphabetic(2048)));
}
```

持久化具有长`name`值的对象的实际例外是:

```java
org.springframework.dao.DataIntegrityViolationException: 
could not execute statement; SQL [n/a]; 
nested exception is org.hibernate.exception.DataException: could not execute statement
   at o.s.o.h.SessionFactoryUtils.convertHibernateAccessException(SessionFactoryUtils.java:143)
...
Caused by: org.hibernate.exception.DataException: could not execute statement
	at o.h.e.i.SQLExceptionTypeDelegate.convert(SQLExceptionTypeDelegate.java:71)
```

在这个特定的示例中，解决方案是指定名称的最大长度:

```java
@Column(nullable = false, length = 4096)
```

## 7。`javax.persistence.EntityExistsException`起因:

与 Hibernate 类似，`EntityExistsException` JPA 异常也将被 Spring 异常翻译包装成`DataIntegrityViolationException`。唯一的区别是 JPA 本身已经是高级的，这使得这个 JPA 异常成为违反数据完整性的唯一潜在原因。

## 8。潜在的`DataIntegrityViolationException`

在某些情况下，`DataIntegrityViolationException`可能是预期的，另一个异常可能被抛出——一个这样的情况是如果 JSR-303 验证器，比如`hibernate-validator` 4 或 5 存在于类路径中。

在这种情况下，如果下面的实体用空值`name`持久化，那么**将不再因持久层触发的数据完整性违规**而失败:

```java
@Entity
public class Foo {
    ...
    @Column(nullable = false)
    @NotNull
    private String name;

    ...
}
```

这是因为执行不会到达持久层——它会在此之前失败，并带有一个`javax.validation.ConstraintViolationException`:

```java
javax.validation.ConstraintViolationException: 
Validation failed for classes [org.baeldung.spring.persistence.model.Foo] 
during persist time for groups [javax.validation.groups.Default, ]
List of constraint violations:[ ConstraintViolationImpl{
    interpolatedMessage='may not be null', propertyPath=name, 
    rootBeanClass=class org.baeldung.spring.persistence.model.Foo, 
    messageTemplate='{javax.validation.constraints.NotNull.message}'}
]
    at o.h.c.b.BeanValidationEventListener.validate(BeanValidationEventListener.java:159)
    at o.h.c.b.BeanValidationEventListener.onPreInsert(BeanValidationEventListener.java:94)
```

## 9。结论

在这篇文章的最后，我们应该有一个清晰的地图来导航可能导致春天`DataIntegrityViolationException`的各种原因和问题，以及如何解决所有这些问题。

在 github 项目中可以找到所有异常示例的实现——这是一个基于 Eclipse 的项目，因此应该很容易导入和运行。