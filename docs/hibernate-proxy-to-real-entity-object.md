# 如何将 Hibernate 代理转换成真实的实体对象

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/hibernate-proxy-to-real-entity-object>

## 1.概观

在本教程中，我们将学习如何将一个 [Hibernate 代理](/web/20220525124241/https://www.baeldung.com/hibernate-proxy-load-method)转换成一个真实的实体对象。在此之前，我们将了解 Hibernate 何时创建代理对象。然后，我们将讨论 Hibernate 代理为什么有用。最后，我们将模拟一个需要取消对象代理的场景。

## 2.Hibernate 什么时候创建代理对象？

**Hibernate 使用代理对象来允许[延迟加载](/web/20220525124241/https://www.baeldung.com/hibernate-lazy-eager-loading)。**为了更好地形象化这个场景，让我们看看*支付收据*和*支付*实体:

```java
@Entity
public class PaymentReceipt {
    ...
    @OneToOne(fetch = FetchType.LAZY)
    private Payment payment;
    ...
}
```

```java
@Entity
@Inheritance(strategy = InheritanceType.TABLE_PER_CLASS)
public abstract class Payment {
    ...
    @ManyToOne(fetch = FetchType.LAZY)
    protected WebUser webUser;
    ...
}
```

例如，加载这些实体将导致 **Hibernate 为与`FetchType.LAZY`相关联的字段创建一个代理对象。**

为了演示，让我们创建并运行一个集成测试:

```java
@Test
public void givenPaymentReceipt_whenAccessingPayment_thenVerifyType() {
    PaymentReceipt paymentReceipt = entityManager.find(PaymentReceipt.class, 3L);
    Assert.assertTrue(paymentReceipt.getPayment() instanceof HibernateProxy);
}
```

从测试中，我们已经加载了一个*支付收据*，并验证了**的*支付*对象不是`CreditCardPayment`** — **的实例，而是一个`HibernateProxy`对象**。

相反，如果没有惰性加载，前面的测试将会失败，因为返回的*支付*对象将会是*信用卡支付*的一个实例。

另外，值得一提的是 Hibernate 使用字节码 **插装** 来创建代理对象。

为了验证这一点，我们可以在集成测试的断言语句行上添加一个断点，并在调试模式下运行它。现在，让我们看看调试器显示了什么:

```java
paymentReceipt = {[[email protected]](/web/20220525124241/https://www.baeldung.com/cdn-cgi/l/email-protection)} 
 payment = {[[email protected]](/web/20220525124241/https://www.baeldung.com/cdn-cgi/l/email-protection)} "[[email protected]](/web/20220525124241/https://www.baeldung.com/cdn-cgi/l/email-protection)"
  $_hibernate_interceptor = {[[email protected]](/web/20220525124241/https://www.baeldung.com/cdn-cgi/l/email-protection)} 
```

从调试器中，我们可以看到 Hibernate 正在使用 [Byte Buddy](/web/20220525124241/https://www.baeldung.com/byte-buddy) ，这是一个在运行时动态生成 Java 类的库。

## 3.Hibernate 代理为什么有用？

### 3.1.用于延迟加载的 Hibernate 代理

我们之前已经了解了一些。为了让它更有意义，让我们试着从 *PaymentReceipt* 和 *Payment* 实体中移除惰性加载机制:

```java
public class PaymentReceipt {
    ...
    @OneToOne
    private Payment payment;
    ...
}
```

```java
public abstract class Payment {
    ...
    @ManyToOne
    protected WebUser webUser;
    ...
}
```

现在，让我们快速检索一个 *PaymentReceipt* 并从日志中检查生成的 SQL:

```java
select
    paymentrec0_.id as id1_2_0_,
    paymentrec0_.payment_id as payment_3_2_0_,
    paymentrec0_.transactionNumber as transact2_2_0_,
    payment1_.id as id1_1_1_,
    payment1_.amount as amount2_1_1_,
    payment1_.webUser_id as webuser_3_1_1_,
    payment1_.cardNumber as cardnumb1_0_1_,
    payment1_.clazz_ as clazz_1_,
    webuser2_.id as id1_3_2_,
    webuser2_.name as name2_3_2_ 
from
    PaymentReceipt paymentrec0_ 
left outer join
    (
        select
            id,
            amount,
            webUser_id,
            cardNumber,
            1 as clazz_ 
        from
            CreditCardPayment 
    ) payment1_ 
        on paymentrec0_.payment_id=payment1_.id 
left outer join
    WebUser webuser2_ 
        on payment1_.webUser_id=webuser2_.id 
where
    paymentrec0_.id=?
```

从日志中我们可以看到， **对 `PaymentReceipt` 的查询包含了多个 join 语句。**

现在，让我们通过适当的延迟加载来运行它:

```java
select
    paymentrec0_.id as id1_2_0_,
    paymentrec0_.payment_id as payment_3_2_0_,
    paymentrec0_.transactionNumber as transact2_2_0_ 
from
    PaymentReceipt paymentrec0_ 
where
    paymentrec0_.id=?
```

显然，通过省略所有不必要的连接语句，生成的 SQL 得到了简化。

### 3.2。用于写数据的休眠代理

为了举例说明，让我们用它来创建一个*付款*，并为其分配一个*web 用户*。如果不使用代理，这将产生两条 SQL 语句:一条 *SELECT* 语句检索 *WebUser* 和一条`INSERT`语句创建 *Payment* 。

让我们使用代理创建一个测试:

```java
@Test
public void givenWebUserProxy_whenCreatingPayment_thenExecuteSingleStatement() {
    entityManager.getTransaction().begin();

    WebUser webUser = entityManager.getReference(WebUser.class, 1L);
    Payment payment = new CreditCardPayment(new BigDecimal(100), webUser, "CN-1234");
    entityManager.persist(payment);

    entityManager.getTransaction().commit();
    Assert.assertTrue(webUser instanceof HibernateProxy);
}
```

值得强调的是，我们使用`entityManager.getReference(…)`来获得一个代理对象。

接下来，让我们运行测试并检查日志:

```java
insert 
into
    CreditCardPayment
    (amount, webUser_id, cardNumber, id) 
values
    (?, ?, ?, ?)
```

**在这里，我们可以看到，在使用代理时，Hibernate 只执行了一条 `INSERT ` 语句，用于 `Payment`**  **的创建。**

## 4.场景:解除压力的需要

给定我们的域模型，让我们假设我们正在检索一个 *PaymentReceipt。*正如我们已经知道的，它与一个*支付*实体相关联，该实体具有一个继承策略 *[每类表](/web/20220525124241/https://www.baeldung.com/hibernate-inheritance#table-per-class)* 和一个惰性获取类型*。*

在我们的例子中，基于填充的数据，*付款收据*的关联*付款*属于*信用卡付款类型。*然而，由于我们使用的是延迟加载，它将是一个代理对象。

现在，我们来看看 `CreditCardPayment` 实体:

```java
@Entity
public class CreditCardPayment extends Payment {

    private String cardNumber;
    ...
}
```

事实上，如果不解除 `payment` 对象的绑定，就不可能从 `CreditCardPayment` 类中检索到`c` `ardNumber` 字段。不管怎样，让我们试着将 `payment` 对象转换成 `CreditCardPayment` ，看看会发生什么:

```java
@Test
public void givenPaymentReceipt_whenCastingPaymentToConcreteClass_thenThrowClassCastException() {
    PaymentReceipt paymentReceipt = entityManager.find(PaymentReceipt.class, 3L);
    assertThrows(ClassCastException.class, () -> {
        CreditCardPayment creditCardPayment = (CreditCardPayment) paymentReceipt.getPayment();
    });
}
```

从测试中，我们看到了需要将 `payment` 对象转换成 `CreditCardPayment` 。但是，**因为****`payment`对象仍然是 Hibernate 代理对象，我们已经遇到了 `ClassCastException`** 。

## 5.实体对象的休眠代理

从 Hibernate 5.2.10 开始，我们可以使用内置的静态方法来释放 Hibernate 实体:

```java
Hibernate.unproxy(paymentReceipt.getPayment());
```

让我们使用这种方法创建一个最终的集成测试:  

```java
@Test
public void givenPaymentReceipt_whenPaymentIsUnproxied_thenReturnRealEntityObject() {
    PaymentReceipt paymentReceipt = entityManager.find(PaymentReceipt.class, 3L);
    Assert.assertTrue(Hibernate.unproxy(paymentReceipt.getPayment()) instanceof CreditCardPayment);
}
```

从测试中，我们可以看到我们已经成功地将一个 Hibernate 代理转换成了一个真实的实体对象。

另一方面，Hibernate 5.2.10 之前有一个解决方案:

```java
HibernateProxy hibernateProxy = (HibernateProxy) paymentReceipt.getPayment();
LazyInitializer initializer = hibernateProxy.getHibernateLazyInitializer();
CreditCardPayment unproxiedEntity = (CreditCardPayment) initializer.getImplementation();
```

## 6.结论

在本教程中，我们学习了如何将 Hibernate 代理转换成真实的实体对象。除此之外，我们还讨论了 Hibernate 代理是如何工作的，以及它为什么有用。然后，我们模拟了一种需要取消对象代理的情况。

最后，我们运行了几个集成测试来演示我们的示例并验证我们的解决方案。

和往常一样，GitHub 上的[提供了这篇文章的完整源代码。](https://web.archive.org/web/20220525124241/https://github.com/eugenp/tutorials/tree/master/persistence-modules/java-jpa-3)