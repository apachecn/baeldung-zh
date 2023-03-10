# Spring @Transactional 中的事务传播和隔离

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-transactional-propagation-isolation>

## 1.介绍

在本教程中，我们将介绍`@Transactional` 注释，以及它的`isolation` 和`propagation`设置。

## 延伸阅读:

## [Java 和 Spring 中的事务介绍](/web/20221002120122/https://www.baeldung.com/java-transactions)

A quick and practical guide to transactions in Java and Spring.[Read more](/web/20221002120122/https://www.baeldung.com/java-transactions) →

## [春季程序化交易管理](/web/20221002120122/https://www.baeldung.com/spring-programmatic-transaction-management)

Learn to manage transactions programmatically in Spring and why this approach is sometimes better than simply using the declarative Transactional annotation.[Read more](/web/20221002120122/https://www.baeldung.com/spring-programmatic-transaction-management) →

## [检测 Spring 事务是否活动](/web/20221002120122/https://www.baeldung.com/spring-transaction-active)

Let's look at how we can find if a Spring transaction is active.[Read more](/web/20221002120122/https://www.baeldung.com/spring-transaction-active) →

## 2.什么是`@Transactional?`

我们可以使用`[@Transactional](/web/20221002120122/https://www.baeldung.com/transaction-configuration-with-jpa-and-spring)` 在数据库事务中包装一个方法。

它允许我们为事务设置传播、隔离、超时、只读和回滚条件。我们还可以通过 指定事务经理。

### 2.1.`@Transactional` 实施细节

Spring 创建一个代理，或者操纵类字节码，来管理事务的创建、提交和回滚。在代理的情况下，Spring 忽略内部方法调用中的 `@Transactional` 。

简而言之，如果我们有一个类似于`callMethod`的方法，并且我们将它标记为`@Transactional,` ，Spring 将在调用的`@Transactional`方法周围包装一些事务管理代码:

```java
createTransactionIfNecessary();
try {
    callMethod();
    commitTransactionAfterReturning();
} catch (exception) {
    completeTransactionAfterThrowing();
    throw exception;
}
```

### 2.2.如何使用`@Transactional`

我们可以将注释放在接口、类的定义上，或者直接放在方法上。它们根据优先级顺序相互覆盖；从最低到最高我们有:接口，超类，类，接口方法，超类方法，和类方法。

**Spring 将类级注释应用于我们没有用 `@Transactional` 注释的这个类的所有公共方法。**

然而，如果我们将注释放在私有或受保护的方法上，Spring 将会忽略它而不会出错。

让我们从一个接口示例开始:

```java
@Transactional
public interface TransferService {
    void transfer(String user1, String user2, double val);
} 
```

通常不建议在界面上设置`@Transactional`；但是对于像`@Repository`这样有 Spring 数据的情况是可以接受的。我们可以将注释放在类定义上，以覆盖接口/超类的事务设置:

```java
@Service
@Transactional
public class TransferServiceImpl implements TransferService {
    @Override
    public void transfer(String user1, String user2, double val) {
        // ...
    }
}
```

现在让我们通过直接在方法上设置注释来覆盖它:

```java
@Transactional
public void transfer(String user1, String user2, double val) {
    // ...
}
```

## 3.事务传播

传播定义了我们的业务逻辑的事务边界。Spring 设法根据我们的`propagation`设置启动和暂停一个事务。

Spring 调用`TransactionManager::getTransaction`来根据传播获取或创建事务。它支持所有类型的`TransactionManager`的一些传播，但是有一些只被`TransactionManager`的特定实现支持。

让我们看看不同的传播方式以及它们是如何工作的。

### 3.1. **`REQUIRED` 传播**

`REQUIRED`是默认传播。Spring 检查是否有活动的事务，如果没有，它就创建一个新的事务。否则，业务逻辑会向当前活动的事务追加:

```java
@Transactional(propagation = Propagation.REQUIRED)
public void requiredExample(String user) { 
    // ... 
}
```

此外，由于`REQUIRED`是默认传播，我们可以通过删除它来简化代码:

```java
@Transactional
public void requiredExample(String user) { 
    // ... 
}
```

让我们看看事务创建如何为`REQUIRED`传播工作的伪代码:

```java
if (isExistingTransaction()) {
    if (isValidateExistingTransaction()) {
        validateExisitingAndThrowExceptionIfNotValid();
    }
    return existing;
}
return createNewTransaction();
```

### 3.2. **`SUPPORTS` 传播**

对于`SUPPORTS`，Spring 首先检查是否存在活动事务。如果存在事务，则将使用现有的事务。如果没有事务，它将以非事务方式执行:

```java
@Transactional(propagation = Propagation.SUPPORTS)
public void supportsExample(String user) { 
    // ... 
}
```

让我们看看`SUPPORTS`的事务创建的伪代码:

```java
if (isExistingTransaction()) {
    if (isValidateExistingTransaction()) {
        validateExisitingAndThrowExceptionIfNotValid();
    }
    return existing;
}
return emptyTransaction;
```

### 3.3. **`MANDATORY` 传播**

当传播为`MANDATORY`时，如果有一个活动的事务，那么它将被使用。如果没有活动的事务，那么 Spring 抛出一个异常:

```java
@Transactional(propagation = Propagation.MANDATORY)
public void mandatoryExample(String user) { 
    // ... 
}
```

让我们再次看看伪代码:

```java
if (isExistingTransaction()) {
    if (isValidateExistingTransaction()) {
        validateExisitingAndThrowExceptionIfNotValid();
    }
    return existing;
}
throw IllegalTransactionStateException;
```

### 3.4. **`NEVER` 传播**

对于带有`NEVER`传播的事务逻辑，如果有一个活动的事务，Spring 会抛出一个异常:

```java
@Transactional(propagation = Propagation.NEVER)
public void neverExample(String user) { 
    // ... 
}
```

让我们看看事务创建如何为`NEVER`传播工作的伪代码:

```java
if (isExistingTransaction()) {
    throw IllegalTransactionStateException;
}
return emptyTransaction;
```

### 3.5. **`NOT_SUPPORTED` 传播**

如果存在当前事务，首先 Spring 会暂停它，然后在没有事务的情况下执行业务逻辑:

```java
@Transactional(propagation = Propagation.NOT_SUPPORTED)
public void notSupportedExample(String user) { 
    // ... 
}
```

**`JTATransactionManager`支持开箱即用的真实交易暂停。其他人通过持有对现有线程的引用，然后从线程上下文中清除它来模拟挂起**

### 3.6. **`REQUIRES_NEW` 传播**

当传播为`REQUIRES_NEW`时，Spring 暂停当前的事务(如果它存在的话)，然后创建一个新的事务:

```java
@Transactional(propagation = Propagation.REQUIRES_NEW)
public void requiresNewExample(String user) { 
    // ... 
}
```

**类似于`NOT_SUPPORTED`，我们需要`JTATransactionManager`来进行实际的交易暂停。**

伪代码看起来像这样:

```java
if (isExistingTransaction()) {
    suspend(existing);
    try {
        return createNewTransaction();
    } catch (exception) {
        resumeAfterBeginException();
        throw exception;
    }
}
return createNewTransaction();
```

### 3.7. **`NESTED` 传播**

对于`NESTED`传播，Spring 检查事务是否存在，如果存在，它标记一个保存点。这意味着如果我们的业务逻辑执行抛出一个异常，那么事务回滚到这个保存点。如果没有活动的事务，它就像 `REQUIRED` 一样工作。

**`DataSourceTransactionManager`支持这种开箱即用的传播。`JTATransactionManager`的一些实现可能也支持这一点。**

**[`JpaTransactionManager`](https://web.archive.org/web/20221002120122/https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/orm/jpa/JpaTransactionManager.html) 仅支持 `NESTED`用于 JDBC 连接。然而，如果我们将`nestedTransactionAllowed`标志设置为`true`，如果我们的 JDBC 驱动程序支持保存点，它也适用于 JPA 交易中的 JDBC 访问代码。** 

最后，让我们将`propagation`设置为`NESTED`:

```java
@Transactional(propagation = Propagation.NESTED)
public void nestedExample(String user) { 
    // ... 
}
```

## 4.事务隔离

隔离是常见的酸属性之一:原子性、一致性、隔离性和持久性。隔离描述了并发事务所应用的更改如何彼此可见。

每个隔离级别都可以防止对事务产生零个或多个并发副作用:

*   **脏读:**读取并发事务的未提交更改
*   **不可重复读取**:如果并发事务更新同一行并提交，则在重新读取该行时获得不同的值
*   **幻像读取:**如果另一个事务在范围中添加或删除了一些行并提交，则在重新执行范围查询后获得不同的行

我们可以通过`@Transactional::isolation.` 来设置一个事务的隔离级别，它在 Spring 中有这五种枚举:`DEFAULT`、`READ_UNCOMMITTED`、`READ_COMMITTED`、`REPEATABLE_READ`、`SERIALIZABLE.`

### 4.1.春季隔离管理

默认隔离级别是`DEFAULT`。因此，当 Spring 创建一个新事务时，隔离级别将是 RDBMS 的默认隔离。因此，如果我们改变数据库，我们应该小心。

我们还应该考虑在正常流程中调用具有不同隔离的方法链的情况，隔离仅在创建新事务时适用。因此，如果出于某种原因我们不想让一个方法在不同的隔离中执行，我们必须将`TransactionManager::setValidateExistingTransaction`设置为 true。

那么交易验证的伪代码将是:

```java
if (isolationLevel != ISOLATION_DEFAULT) {
    if (currentTransactionIsolationLevel() != isolationLevel) {
        throw IllegalTransactionStateException
    }
}
```

现在让我们深入了解不同的隔离级别及其影响。

### 4.2.`READ_UNCOMMITTED` 隔离

`READ_UNCOMMITTED` 是最低的隔离级别，允许最多的并发访问。

结果，它遭受了所有三个提到的并发副作用。具有这种隔离的事务读取其他并发事务的未提交数据。此外，不可重复读取和幻像读取都可能发生。因此，我们可以在重新读取行或重新执行范围查询时获得不同的结果。

我们可以为一个方法或类设置`isolation`级别:

```java
@Transactional(isolation = Isolation.READ_UNCOMMITTED)
public void log(String message) {
    // ...
}
```

**Postgres 不支持`READ_UNCOMMITTED`隔离，回退到`READ_COMMITED` 代替。** **还有，甲骨文不支持也不允许`READ_UNCOMMITTED`。** 

### 4.3.`READ_COMMITTED` 隔离

第二级隔离， `READ_COMMITTED,` 防止脏读。

其余的并发副作用仍然可能发生。所以并发事务中未提交的更改对我们没有影响，但是如果一个事务提交了它的更改，我们的结果可能会因为重新查询而改变。

这里我们设置`isolation`等级:

```java
@Transactional(isolation = Isolation.READ_COMMITTED)
public void log(String message){
    // ...
}
```

**`READ_COMMITTED`是 Postgres、SQL Server、Oracle 的默认级别。**

### 4.4.`REPEATABLE_READ` 隔离

第三级隔离， `REPEATABLE_READ,` 防止脏的和不可重复的读取。因此我们不会受到并发事务中未提交更改的影响。

另外，当我们重新查询一行时，我们不会得到不同的结果。然而，在重新执行范围查询时，我们可能会得到新添加或删除的行。

此外，这是防止更新丢失所需的最低级别。当两个或多个并发事务读取和更新同一行时，会发生更新丢失。`REPEATABLE_READ` 根本不允许同时访问一行。因此，丢失的更新不会发生。


以下是如何设置方法的`isolation`级别:

```java
@Transactional(isolation = Isolation.REPEATABLE_READ) 
public void log(String message){
    // ...
}
```

**`REPEATABLE_READ`是 Mysql 中的默认级别。** **甲骨文不支持 `REPEATABLE_READ`。**

### 4.5.`SERIALIZABLE` 隔离

`SERIALIZABLE` 是最高级别的隔离。它防止了所有提到的并发副作用，但可能导致最低的并发访问率，因为它按顺序执行并发调用。

换句话说，一组可串行化事务的并发执行与串行执行具有相同的结果。

现在让我们看看如何将`SERIALIZABLE`设置为`isolation`电平:

```java
@Transactional(isolation = Isolation.SERIALIZABLE)
public void log(String message){
    // ...
}
```

## 5.结论

在本文中，我们详细探讨了`@Transaction`的传播特性。然后，我们了解了并发的副作用和隔离级别。

和往常一样，本文的完整代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20221002120122/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-persistence-simple)