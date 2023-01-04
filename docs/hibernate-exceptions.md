# 常见休眠异常

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/hibernate-exceptions>

## 1.介绍

在本教程中，我们将讨论一些在使用 Hibernate 时会遇到的常见异常。

我们将回顾它们的目的和一些常见原因。此外，我们将研究他们的解决方案。

## 2.休眠异常概述

使用 Hibernate 时，许多情况都会导致异常。这些错误可能是映射错误、基础结构问题、SQL 错误、数据完整性违规、会话问题和事务错误。

**这些例外大多从`[HibernateException](https://web.archive.org/web/20220703143024/https://docs.jboss.org/hibernate/orm/6.0/javadocs/org/hibernate/HibernateException.html)`开始延续。**然而，如果我们使用 Hibernate 作为 JPA 持久性提供者，这些异常可能会被打包到 [`PersistenceException`](https://web.archive.org/web/20220703143024/https://docs.oracle.com/javaee/7/api/javax/persistence/PersistenceException.html) 中。

这两个基类都从`RuntimeException`扩展而来。因此，它们都没有被选中。因此，我们不需要在每个使用它们的地方捕捉或声明它们。

此外，这些大部分是不可恢复的。因此，重试该操作不会有任何帮助。这意味着我们必须在遇到它们时放弃当前会话。

现在，让我们一次看一个。

## 3.映射错误

对象关系映射是 Hibernate 的一个主要优点。具体来说，它将我们从手工编写 SQL 语句中解放出来。

同时，它要求我们指定 Java 对象和数据库表之间的映射。因此，我们使用注释或通过映射文档来指定它们。这些映射可以手动编码。或者，我们可以使用工具来生成它们。

在指定这些映射时，我们可能会出错。这些可能在映射规范中。或者，Java 对象和相应的数据库表之间可能不匹配。

这种映射错误会产生异常。在最初的开发过程中，我们经常会遇到它们。此外，在跨环境迁移变更时，我们可能会遇到这些问题。

让我们通过一些例子来研究这些错误。

### 3.1.`MappingException`

**对象关系映射的问题导致抛出`MappingException`**:

```java
public void whenQueryExecutedWithUnmappedEntity_thenMappingException() {
    thrown.expectCause(isA(MappingException.class));
    thrown.expectMessage("Unknown entity: java.lang.String");

    Session session = sessionFactory.getCurrentSession();
    NativeQuery<String> query = session
      .createNativeQuery("select name from PRODUCT", String.class);
    query.getResultList();
}
```

在上面的代码中，`createNativeQuery`方法试图将查询结果映射到指定的 Java 类型`String.` ，它使用来自`[Metamodel](https://web.archive.org/web/20220703143024/https://docs.oracle.com/javaee/6/api/javax/persistence/metamodel/Metamodel.html)` 的`String`类的隐式映射来进行映射。

然而，`String`类没有指定任何映射。因此，Hibernate 不知道如何将`name`列映射到`String`，并抛出异常。

有关可能原因和解决方案的详细分析，请查看 [Hibernate 映射异常-未知实体](/web/20220703143024/https://www.baeldung.com/hibernate-mappingexception-unknown-entity)。

同样，其他错误也可能导致此异常:

*   混合字段和方法的注释
*   无法为`@ManyToMany`关联指定`@JoinTable`
*   映射类的默认构造函数在映射处理过程中引发异常

此外， **`MappingException`有几个子类可以表示特定的映射问题:**

*   `AnnotationException – `注释的问题
*   `DuplicateMappingException –` 类别、表格或属性名称的重复映射
*   `InvalidMappingException – `映射无效
*   `MappingNotFoundException – `找不到映射资源
*   `PropertyNotFoundException – `在类上找不到预期的 getter 或 setter 方法

因此，**如果我们遇到这个异常，我们应该首先验证我们的映射**。

### 3.2.`AnnotationException`

为了理解`AnnotationException,` ,让我们创建一个在任何字段或属性上没有标识符注释的实体:

```java
@Entity
public class EntityWithNoId {
    private int id;
    public int getId() {
        return id;
    }

    // standard setter
}
```

因为 **Hibernate 期望每个实体都有一个[标识符](/web/20220703143024/https://www.baeldung.com/hibernate-identifiers)** ，所以当我们使用实体时，我们将得到一个`AnnotationException`:

```java
public void givenEntityWithoutId_whenSessionFactoryCreated_thenAnnotationException() {
    thrown.expect(AnnotationException.class);
    thrown.expectMessage("No identifier specified for entity");

    Configuration cfg = getConfiguration();
    cfg.addAnnotatedClass(EntityWithNoId.class);
    cfg.buildSessionFactory();
}
```

此外，其他一些可能的原因有:

*   在`@GeneratedValue`注释中使用了未知的序列发生器
*   与 Java 8 `Date` / `Time`类一起使用的`@Temporal`注释
*   `@ManyToOne`或`@OneToMany`的目标实体缺失或不存在
*   [与关系注释`@OneToMany`或`@ManyToMany`一起使用的原始集合类](/web/20220703143024/https://www.baeldung.com/java-diamond-operator)
*   Hibernate 期望集合接口的集合注释`@OneToMany`、`@ManyToMany`或`@ElementCollection`使用的具体类

要解决这个异常，我们应该首先检查错误消息中提到的特定注释。

## 4.架构管理错误

自动数据库模式管理是 Hibernate 的另一个好处。例如，它可以生成 DDL 语句来创建或验证数据库对象。

要使用这个特性，我们需要适当地设置`hibernate.hbm2ddl.auto `属性。

如果在执行模式管理时出现问题，我们会得到一个异常。让我们检查这些错误。

### 4.1.`SchemaManagementException`

在执行模式管理时，任何与基础设施相关的问题都会导致`SchemaManagementException`。

为了演示，让我们指示 Hibernate 验证数据库模式:

```java
public void givenMissingTable_whenSchemaValidated_thenSchemaManagementException() {
    thrown.expect(SchemaManagementException.class);
    thrown.expectMessage("Schema-validation: missing table");

    Configuration cfg = getConfiguration();
    cfg.setProperty(AvailableSettings.HBM2DDL_AUTO, "validate");
    cfg.addAnnotatedClass(Product.class);
    cfg.buildSessionFactory();
}
```

由于对应于`Product`的表不在数据库中，我们在构建 S `essionFactory`时得到了模式验证异常。

此外，此异常还有其他可能的情况:

*   无法连接到数据库来执行模式管理任务
*   数据库中不存在该架构

### 4.2.`CommandAcceptanceException`

执行与特定模式管理命令相对应的 DDL 的任何问题都可能导致`CommandAcceptanceException`。

例如，让我们在设置`SessionFactory`时指定错误的方言:

```java
public void whenWrongDialectSpecified_thenCommandAcceptanceException() {
    thrown.expect(SchemaManagementException.class);

    thrown.expectCause(isA(CommandAcceptanceException.class));
    thrown.expectMessage("Halting on error : Error executing DDL");

    Configuration cfg = getConfiguration();
    cfg.setProperty(AvailableSettings.DIALECT,
      "org.hibernate.dialect.MySQLDialect");
    cfg.setProperty(AvailableSettings.HBM2DDL_AUTO, "update");
    cfg.setProperty(AvailableSettings.HBM2DDL_HALT_ON_ERROR,"true");
    cfg.getProperties()
      .put(AvailableSettings.HBM2DDL_HALT_ON_ERROR, true);

    cfg.addAnnotatedClass(Product.class);
    cfg.buildSessionFactory();
}
```

这里，我们指定了错误的方言:`MySQLDialect. `此外，我们还指示 Hibernate 更新模式对象。因此，Hibernate 执行的更新 H2 数据库的 DDL 语句会失败，我们会得到一个异常。

默认情况下，Hibernate 会默默地记录这个异常并继续运行。当我们稍后使用`SessionFactory, `时，我们得到了异常。

为了确保这个错误会引发异常，我们将属性`HBM2DDL_HALT_ON_ERROR`设置为`true`。

同样，以下是导致此错误的一些其他常见原因:

*   映射和数据库之间的列名不匹配
*   两个类映射到同一个表
*   用于类或表的名称是数据库中的保留字，例如`USER`
*   用于连接数据库的用户没有所需的权限

## 5.SQL 执行错误

**当我们使用 Hibernate 插入、更新、删除或查询数据时，它使用 JDBC** 对数据库执行 DML 语句。如果操作导致错误或警告，这个 API 会引发一个`SQLException`。

Hibernate 将这个异常转换成`JDBCException`或者它的一个合适的子类:

*   `ConstraintViolationException`
*   `DataException`
*   `JDBCConnectionException`
*   `LockAcquisitionException`
*   `PessimisticLockException`
*   `QueryTimeoutException`
*   `SQLGrammarException`
*   `GenericJDBCException`

我们来讨论一下常见的错误。

### 5.1.`JDBCException`

`JDBCException`总是由特定的 SQL 语句引起的。我们可以调用`getSQL`方法来获取有问题的 SQL 语句。

此外，我们可以用`getSQLException`方法检索底层的`SQLException`。

### 5.2.`SQLGrammarException`

`SQLGrammarException`表示发送到数据库的 SQL 无效。这可能是由于语法错误或无效的对象引用。

例如，**在查询数据**时，一个缺失的表会导致此错误:

```java
public void givenMissingTable_whenQueryExecuted_thenSQLGrammarException() {
    thrown.expect(isA(PersistenceException.class));
    thrown.expectCause(isA(SQLGrammarException.class));
    thrown.expectMessage("SQLGrammarException: could not prepare statement");

    Session session = sessionFactory.getCurrentSession();
    NativeQuery<Product> query = session.createNativeQuery(
      "select * from NON_EXISTING_TABLE", Product.class);
    query.getResultList();
}
```

此外，如果表丢失，我们在保存数据时也会出现此错误:

```java
public void givenMissingTable_whenEntitySaved_thenSQLGrammarException() {
    thrown.expect(isA(PersistenceException.class));
    thrown.expectCause(isA(SQLGrammarException.class));
    thrown
      .expectMessage("SQLGrammarException: could not prepare statement");

    Configuration cfg = getConfiguration();
    cfg.addAnnotatedClass(Product.class);

    SessionFactory sessionFactory = cfg.buildSessionFactory();
    Session session = null;
    Transaction transaction = null;
    try {
        session = sessionFactory.openSession();
        transaction = session.beginTransaction();
        Product product = new Product();
        product.setId(1);
        product.setName("Product 1");
        session.save(product);
        transaction.commit();
    } catch (Exception e) {
        rollbackTransactionQuietly(transaction);
        throw (e);
    } finally {
        closeSessionQuietly(session);
        closeSessionFactoryQuietly(sessionFactory);
    }
}
```

其他一些可能的原因有:

*   使用的命名策略没有将类映射到正确的表
*   `@JoinColumn`中指定的列不存在

### 5.3.`ConstraintViolationException`

`ConstraintViolationException`表示所请求的 DML 操作导致完整性约束被违反。我们可以通过调用`getConstraintName`方法来获得这个约束的名称。

此异常的一个常见原因是试图保存重复记录:

```java
public void whenDuplicateIdSaved_thenConstraintViolationException() {
    thrown.expect(isA(PersistenceException.class));
    thrown.expectCause(isA(ConstraintViolationException.class));
    thrown.expectMessage(
      "ConstraintViolationException: could not execute statement");

    Session session = null;
    Transaction transaction = null;

    for (int i = 1; i <= 2; i++) {
        try {
            session = sessionFactory.openSession();
            transaction = session.beginTransaction();
            Product product = new Product();
            product.setId(1);
            product.setName("Product " + i);
            session.save(product);
            transaction.commit();
        } catch (Exception e) {
            rollbackTransactionQuietly(transaction);
            throw (e);
        } finally {
            closeSessionQuietly(session);
        }
    }
}
```

此外，将一个`null`值保存到数据库中的一个`NOT NULL`列也会引发这个错误。

为了解决这个错误，**我们应该在业务层**执行所有验证。此外，数据库约束不应该用于应用程序验证。

### 5.4.`DataException`

`DataException`表示 SQL 语句的评估导致了一些非法操作、类型不匹配或基数不正确。

例如，对数值列使用字符数据会导致以下错误:

```java
public void givenQueryWithDataTypeMismatch_WhenQueryExecuted_thenDataException() {
    thrown.expectCause(isA(DataException.class));
    thrown.expectMessage(
      "org.hibernate.exception.DataException: could not prepare statement");

    Session session = sessionFactory.getCurrentSession();
    NativeQuery<Product> query = session.createNativeQuery(
      "select * from PRODUCT where id='wrongTypeId'", Product.class);
    query.getResultList();
}
```

为了修复这个错误，**我们应该确保应用程序代码和数据库**之间的数据类型和长度匹配。

### 5.5.`JDBCConnectionException`

A `JDBCConectionException`表示与数据库的通信出现问题。

例如，数据库或网络出现故障可能会引发此异常。

此外，不正确的数据库设置也会导致此异常。其中一种情况是数据库连接被服务器关闭，因为它长时间处于空闲状态。如果我们正在使用[连接池](/web/20220703143024/https://www.baeldung.com/java-connection-pooling)并且池上的空闲超时设置大于数据库中的连接超时值，就会发生这种情况。

要解决这个问题，我们应该首先确保数据库主机存在并且已经启动。然后，我们应该验证数据库连接使用了正确的身份验证。最后，我们应该检查连接池上的超时值设置是否正确。

### 5.6.`QueryTimeoutException`

当数据库查询超时时，我们会得到这个异常。我们还可以看到其他错误，比如表空间变满。

这是少数可恢复的错误之一，这意味着我们可以在同一个事务中重试该语句。

为了解决这个问题，**我们可以通过多种方式增加长时间运行的查询**的查询超时:

*   在`@NamedQuery`或`@NamedNativeQuery`标注中设置 [`timeout`](https://web.archive.org/web/20220703143024/https://docs.jboss.org/hibernate/orm/6.0/javadocs/org/hibernate/annotations/NamedQuery.html#timeout--) 元素
*   调用`the Query`接口的` [setHint](https://web.archive.org/web/20220703143024/https://javaee.github.io/javaee-spec/javadocs/javax/persistence/Query.html#setHint-java.lang.String-java.lang.Object-)`方法
*   调用`Transaction`接口的 [`setTimeout`](https://web.archive.org/web/20220703143024/http://docs.jboss.org/hibernate/orm/6.0/javadocs/org/hibernate/Transaction.html#setTimeout-int-) 方法
*   调用 `Query`接口的 [`setTimeout`](https://web.archive.org/web/20220703143024/http://docs.jboss.org/hibernate/orm/6.0/javadocs/org/hibernate/query/Query.html#setTimeout-int-) 方法

## 6.与会话状态相关的错误

现在让我们来看看由于 Hibernate 会话使用错误而导致的错误。

### 6.1.`NonUniqueObjectException`

Hibernate 不允许两个对象在一个会话中有相同的标识符。

如果我们试图在一个会话中将同一个 Java 类的两个实例与同一个标识符关联起来，我们会得到一个`NonUniqueObjectException`。我们可以通过调用`getEntityName()`和`getIdentifier()`方法来获得实体的名称和标识符。

要重现此错误，让我们尝试用会话保存两个具有相同 id 的`Product`实例:

```java
public void 
givenSessionContainingAnId_whenIdAssociatedAgain_thenNonUniqueObjectException() {
    thrown.expect(isA(NonUniqueObjectException.class));
    thrown.expectMessage(
      "A different object with the same identifier value was already associated with the session");

    Session session = null;
    Transaction transaction = null;

    try {
        session = sessionFactory.openSession();
        transaction = session.beginTransaction();

        Product product = new Product();
        product.setId(1);
        product.setName("Product 1");
        session.save(product);

        product = new Product();
        product.setId(1);
        product.setName("Product 2");
        session.save(product);

        transaction.commit();
    } catch (Exception e) {
        rollbackTransactionQuietly(transaction);
        throw (e);
    } finally {
        closeSessionQuietly(session);
    }
}
```

我们会像预期的那样得到一个`NonUniqueObjectException, `。

**当通过调用`update`方法用会话重新附加一个分离的对象时，这个异常经常发生。**如果会话加载了另一个具有相同标识符的实例，那么我们会得到这个错误。为了解决这个问题，**我们可以使用`merge`方法**来重新连接分离的对象。

### 6.2.`StaleStateException`

当版本号或时间戳检查失败时，Hibernate 抛出 *StaleStateException* s。它表示会话包含过时数据。

有时这被打包成一个`OptimisticLockException`。

此错误通常发生在使用具有版本控制的长时间运行的事务时。

此外，如果相应的数据库行不存在，在尝试更新或删除实体时也会发生这种情况:

```java
public void whenUpdatingNonExistingObject_thenStaleStateException() {
    thrown.expect(isA(OptimisticLockException.class));
    thrown.expectMessage(
      "Batch update returned unexpected row count from update");
    thrown.expectCause(isA(StaleStateException.class));

    Session session = null;
    Transaction transaction = null;

    try {
        session = sessionFactory.openSession();
        transaction = session.beginTransaction();

        Product product = new Product();
        product.setId(15);
        product.setName("Product1");
        session.update(product);
        transaction.commit();
    } catch (Exception e) {
        rollbackTransactionQuietly(transaction);
        throw (e);
    } finally {
        closeSessionQuietly(session);
    }
}
```

其他一些可能的情况有:

*   我们没有为实体指定适当的未保存值策略
*   两个用户几乎同时尝试删除同一行
*   我们在自动生成的 ID 或版本字段中手动设置一个值

## 7.惰性初始化错误

为了提高应用程序的性能，我们通常将关联配置为延迟加载。关联只有在第一次使用时才会被提取。

然而，Hibernate 需要一个活动会话来获取数据。如果当我们试图访问一个未初始化的关联时会话已经关闭，我们会得到一个异常。

让我们来看看这个异常和修复它的各种方法。

### 7.1.`LazyInitializationException`

**`LazyInitializationException`表示试图在活动会话之外加载未初始化的数据。**我们可以在很多场景中得到这个错误。

首先，我们可以在访问表示层中的惰性关系时得到这个异常。原因是实体在业务层中被部分加载，并且会话被关闭。

其次，如果我们使用`getOne`方法，我们可以用[弹簧数据](/web/20220703143024/https://www.baeldung.com/the-persistence-layer-with-spring-data-jpa)得到这个误差。该方法延迟获取实例。

有很多方法可以解决这个异常。

首先，我们可以让所有的关系都热切地加载。但是，这将影响应用程序的性能，因为我们将加载不会被使用的数据。

其次，我们可以保持会话打开，直到视图被呈现。这被称为视图 中的 [**开放会话，这是一种反模式。我们应该避免这样做，因为它有几个缺点。**](https://web.archive.org/web/20220703143024/https://vladmihalcea.com/the-open-session-in-view-anti-pattern/)

第三，我们可以打开另一个会话并重新附加实体，以获取关系。我们可以通过在会话中使用`merge`方法来做到这一点。

最后，我们可以在业务层中初始化所需的关联。我们将在下一节讨论这一点。

### 7.2.在业务层初始化相关的惰性关系

有许多方法可以初始化懒惰关系。

一种选择是通过在实体上调用相应的方法来初始化它们。在这种情况下，Hibernate 将发出多个数据库查询，导致性能下降。我们称之为“N+1 选择”问题。

其次，我们可以使用`Fetch Join`在单个查询中获取数据。然而，我们需要编写自定义代码来实现这一点。

最后，**我们可以使用[实体图](/web/20220703143024/https://www.baeldung.com/jpa-entity-graph)来定义所有要提取的属性**。我们可以使用注释`@NamedEntityGraph, @NamedAttributeNode`和`@NamedEntitySubgraph`来声明性地定义实体图。我们也可以用 JPA API 以编程方式定义它们。然后，**我们通过在获取操作**中指定它，在单个调用中检索整个图。

## 8.交易问题

[事务](https://web.archive.org/web/20220703143024/http://docs.jboss.org/hibernate/orm/6.0/userguide/html_single/Hibernate_User_Guide.html#transactions)定义并发活动之间的工作单元和隔离。我们可以用两种不同的方式来区分它们。首先，我们可以使用注释来声明性地定义它们。其次，我们可以使用 [Hibernate 事务 API](https://web.archive.org/web/20220703143024/http://docs.jboss.org/hibernate/orm/6.0/userguide/html_single/Hibernate_User_Guide.html#transactions-api) 以编程方式管理它们。

此外，Hibernate 将事务管理委托给事务管理器。如果某个事务由于某种原因无法启动、提交或回滚，Hibernate 会抛出一个异常。

**根据事务管理器的不同，我们通常会得到一个`TransactionException `或一个`IllegalArgumentException`。**

作为示例，让我们尝试提交一个已标记为回滚的事务:

```java
public void 
givenTxnMarkedRollbackOnly_whenCommitted_thenTransactionException() {
    thrown.expect(isA(TransactionException.class));
    thrown.expectMessage(
        "Transaction was marked for rollback only; cannot commit");

    Session session = null;
    Transaction transaction = null;
    try {
        session = sessionFactory.openSession();
        transaction = session.beginTransaction();

        Product product = new Product();
        product.setId(15);
        product.setName("Product1");
        session.save(product);
        transaction.setRollbackOnly();

        transaction.commit();
    } catch (Exception e) {
        rollbackTransactionQuietly(transaction);
        throw (e);
    } finally {
        closeSessionQuietly(session);
    }
}
```

同样，其他错误也可能导致异常:

*   混合声明性和编程性事务
*   当会话中已有一个事务处于活动状态时，试图启动该事务
*   尝试在不启动事务的情况下提交或回滚
*   多次尝试提交或回滚事务

## 9.并发问题

Hibernate 支持两种[锁定](https://web.archive.org/web/20220703143024/http://docs.jboss.org/hibernate/orm/6.0/userguide/html_single/Hibernate_User_Guide.html#locking)策略，以防止并发事务导致的数据库不一致——[乐观](/web/20220703143024/https://www.baeldung.com/jpa-optimistic-locking)和[悲观](/web/20220703143024/https://www.baeldung.com/jpa-pessimistic-locking)。在锁定冲突的情况下，它们都会引发异常。

为了支持高并发性和高可伸缩性，我们通常使用带有版本检查的乐观并发控制。这使用版本号或时间戳来检测冲突的更新。

抛出 **`OptimisticLockingException`表示乐观锁定冲突**。例如，如果我们对同一个实体执行两次更新或删除操作，而没有在第一次操作后刷新它，就会出现此错误:

```java
public void whenDeletingADeletedObject_thenOptimisticLockException() {
    thrown.expect(isA(OptimisticLockException.class));
    thrown.expectMessage(
        "Batch update returned unexpected row count from update");
    thrown.expectCause(isA(StaleStateException.class));

    Session session = null;
    Transaction transaction = null;

    try {
        session = sessionFactory.openSession();
        transaction = session.beginTransaction();

        Product product = new Product();
        product.setId(12);
        product.setName("Product 12");
        session.save(product1);
        transaction.commit();
        session.close();

        session = sessionFactory.openSession();
        transaction = session.beginTransaction();
        product = session.get(Product.class, 12);
        session.createNativeQuery("delete from Product where id=12")
          .executeUpdate();
        // We need to refresh to fix the error.
        // session.refresh(product);
        session.delete(product);
        transaction.commit();
    } catch (Exception e) {
        rollbackTransactionQuietly(transaction);
        throw (e);
    } finally {
        closeSessionQuietly(session);
    }
}
```

同样，如果两个用户试图几乎同时更新同一个实体，我们也会得到这个错误。在这种情况下，第一个可能会成功，第二个会引发此错误。

因此，**如果不引入悲观锁定**，我们无法完全避免这个错误。但是，我们可以通过执行以下操作来最小化其发生的概率:

*   尽可能缩短更新操作
*   尽可能频繁地更新客户端中的实体表示
*   不要缓存实体或代表它的任何值对象
*   更新后总是刷新客户端上的实体表示

## 10.结论

在本文中，我们研究了使用 Hibernate 时遇到的一些常见异常。此外，我们调查了它们的可能原因和解决方案。

像往常一样，完整的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220703143024/https://github.com/eugenp/tutorials/tree/master/persistence-modules/hibernate-enterprise)