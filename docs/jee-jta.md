# 雅加达 EE JTA 指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jee-jta>

## 1。概述

Java 事务 API，俗称 JTA，**是 Java 中管理事务的 API。**它允许我们以与资源无关的方式启动、提交和回滚事务。

JTA 的真正强大之处在于它能够在一个事务中管理多个资源(如数据库、消息服务)。

在本教程中，我们将从概念层面了解 JTA，并了解业务代码通常如何与 JTA 交互。

## 2。通用 API 和分布式事务

JTA 为业务代码提供了对事务控制(开始、提交和回滚)的抽象。

如果没有这种抽象，我们就必须处理每种资源类型的单独 API。

例如，我们需要像这样处理 JDBC 资源。同样，JMS 资源可能有一个[相似但不兼容的模型](https://web.archive.org/web/20220628094138/https://docs.oracle.com/cd/E19798-01/821-1841/bncgh/index.html)。

借助 JTA，我们能够以一致和协调的方式管理多种不同类型的资源。

作为一个 API，JTA 定义了由`transaction managers`实现的接口和语义。实现由库提供，如 [Narayana](https://web.archive.org/web/20220628094138/http://narayana.io/) 和 [Atomikos](/web/20220628094138/https://www.baeldung.com/java-atomikos) 。

## 3。示例项目设置

示例应用程序是一个非常简单的银行应用程序的后端服务。我们有两个服务，`BankAccountService `和`AuditService`使用两个不同的数据库**。** **这些独立的数据库需要在事务开始、提交或回滚时进行协调**。

首先，我们的示例项目使用 Spring Boot 来简化配置:

```java
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.6.6</version>
</parent>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jta-atomikos</artifactId>
</dependency>
```

最后，在每个测试方法之前，我们用空数据初始化`AUDIT_LOG`,用 2 行数据初始化数据库`ACCOUNT`:

```java
+-----------+----------------+
| ID        |  BALANCE       |
+-----------+----------------+
| a0000001  |  1000          |  
| a0000002  |  2000          |
+-----------+----------------+
```

## 4。声明性事务界定

在 JTA 处理事务的第一种方式是使用 [`@Transactional`](https://web.archive.org/web/20220628094138/https://docs.oracle.com/javaee/7/api/javax/transaction/Transactional.html) 注释。更详细的解释和配置见[这篇文章](/web/20220628094138/https://www.baeldung.com/transaction-configuration-with-jpa-and-spring)。

让我们用`@Transactional. `来注释门面服务方法`executeTranser()`，这指示`transaction manager`开始一个事务`:`

```java
@Transactional
public void executeTransfer(String fromAccontId, String toAccountId, BigDecimal amount) {
    bankAccountService.transfer(fromAccontId, toAccountId, amount);
    auditService.log(fromAccontId, toAccountId, amount);
    ...
}
```

这里方法`executeTranser()`调用两个不同的服务，`AccountService`和`AuditService.` ，这些服务使用两个不同的数据库。

当`executeTransfer()`返回时，**`transaction manager`认识到这是事务的结束，并将提交给两个数据库**:

```java
tellerService.executeTransfer("a0000001", "a0000002", BigDecimal.valueOf(500));
assertThat(accountService.balanceOf("a0000001"))
  .isEqualByComparingTo(BigDecimal.valueOf(500));        
assertThat(accountService.balanceOf("a0000002"))
  .isEqualByComparingTo(BigDecimal.valueOf(2500));

TransferLog lastTransferLog = auditService
  .lastTransferLog();
assertThat(lastTransferLog)
  .isNotNull();        
assertThat(lastTransferLog.getFromAccountId())
  .isEqualTo("a0000001");
assertThat(lastTransferLog.getToAccountId())
  .isEqualTo("a0000002"); 
assertThat(lastTransferLog.getAmount())
  .isEqualByComparingTo(BigDecimal.valueOf(500));
```

### 4.1。在声明性分界中回滚

在该方法结束时，`executeTransfer()`检查账户余额，如果源资金不足，则抛出`RuntimeException`:

```java
@Transactional
public void executeTransfer(String fromAccontId, String toAccountId, BigDecimal amount) {
    bankAccountService.transfer(fromAccontId, toAccountId, amount);
    auditService.log(fromAccontId, toAccountId, amount);
    BigDecimal balance = bankAccountService.balanceOf(fromAccontId);
    if(balance.compareTo(BigDecimal.ZERO) < 0) {
        throw new RuntimeException("Insufficient fund.");
    }
}
```

超过第一个`@Transactional`的未处理的**`RuntimeException`将把事务** **回滚到两个数据库**。实际上，执行金额大于余额的转账将导致回滚 **:**

```java
assertThatThrownBy(() -> {
    tellerService.executeTransfer("a0000002", "a0000001", BigDecimal.valueOf(10000));
}).hasMessage("Insufficient fund.");

assertThat(accountService.balanceOf("a0000001")).isEqualByComparingTo(BigDecimal.valueOf(1000));
assertThat(accountService.balanceOf("a0000002")).isEqualByComparingTo(BigDecimal.valueOf(2000));
assertThat(auditServie.lastTransferLog()).isNull();
```

## 5。程序化交易界定

控制 JTA 事务的另一种方式是通过 [`UserTransaction`](https://web.archive.org/web/20220628094138/https://javaee.github.io/javaee-spec/javadocs/javax/transaction/UserTransaction.html) 编程。

现在让我们修改`executeTransfer()`来手动处理事务:

```java
userTransaction.begin();

bankAccountService.transfer(fromAccontId, toAccountId, amount);
auditService.log(fromAccontId, toAccountId, amount);
BigDecimal balance = bankAccountService.balanceOf(fromAccontId);
if(balance.compareTo(BigDecimal.ZERO) < 0) {
    userTransaction.rollback();
    throw new RuntimeException("Insufficient fund.");
} else {
    userTransaction.commit();
}
```

在我们的例子中，`begin()`方法启动了一个新的事务。如果余额验证失败，我们调用`rollback()` ，这将回滚到两个数据库。否则，**对`commit()`** **的调用提交对两个数据库的更改**。

需要注意的是，`commit()`和`rollback()`都结束当前事务。

最终，使用编程划分给了我们细粒度事务控制的灵活性。

## 6。结论

在本文中，我们讨论了 JTA 试图解决的问题。代码示例**说明了用注释和编程方式**控制事务，涉及两个需要在单个事务中协调的事务资源。

像往常一样，代码示例可以在 GitHub 上找到[。](https://web.archive.org/web/20220628094138/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-persistence-simple)