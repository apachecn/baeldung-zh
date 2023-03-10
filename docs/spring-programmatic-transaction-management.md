# Spring 中的程序化事务管理

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-programmatic-transaction-management>

## 1.概观

Spring 的 [`@Transactional`](/web/20220926190313/https://www.baeldung.com/transaction-configuration-with-jpa-and-spring) 注释提供了一个很好的声明式 API 来标记事务边界。

在幕后，方面负责创建和维护事务，因为它们是在每次出现`@Transactional`注释时定义的。这种方法很容易将我们的核心业务逻辑从横切关注点中分离出来，比如事务管理。

在本教程中，我们将看到这并不总是最好的方法。我们将探索 Spring 提供了哪些编程选择，比如`TransactionTemplate`，以及我们使用它们的原因。

## 2.天堂的烦恼

假设我们在一个简单的服务中混合了两种不同类型的 I/O:

```java
@Transactional
public void initialPayment(PaymentRequest request) {
    savePaymentRequest(request); // DB
    callThePaymentProviderApi(request); // API
    updatePaymentState(request); // DB
    saveHistoryForAuditing(request); // DB
}
```

这里我们有一些数据库调用和一个可能很昂贵的 REST API 调用。乍一看，让整个方法成为事务性的可能是有意义的，因为我们可能希望使用一个`EntityManager `来原子地执行整个操作。

然而，如果由于某种原因，外部 API 需要比平常更长的时间来响应，我们可能很快就会耗尽数据库连接！

### 2.1.现实的残酷性

下面是我们调用`initialPayment `方法时发生的情况:

1.  事务方面创建了一个新的`EntityManager` 并启动了一个新的事务，因此它从连接池中借用了一个`Connection `。
2.  在第一次数据库调用之后，它调用外部 API，同时保留借用的`Connection`。
3.  最后，它使用那个`Connection `来执行剩余的数据库调用。

**如果 API 调用在一段时间内响应非常慢，这个方法会在等待响应的时候占用借用的`Connection `。**

想象一下，在此期间，我们收到了对`initialPayment `方法的大量调用。在这种情况下，所有的`Connections `都可以等待 API 调用的响应。**这就是为什么我们可能会因为缓慢的后端服务而耗尽数据库连接！**

在事务环境中混合数据库 I/O 和其他类型的 I/O 不是一个好主意。因此，这类问题的第一个解决方案是将这些类型的 I/O 完全分开。如果出于某种原因我们不能将它们分开，我们仍然可以使用 Spring APIs 来手动管理事务。

## 3.使用`TransactionTemplate`

`[TransactionTemplate](https://web.archive.org/web/20220926190313/https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/transaction/support/TransactionTemplate.html) `提供一组基于回调的 API 来手动管理事务。为了使用它，我们应该首先用一个`PlatformTransactionManager`初始化它。

我们可以使用依赖注入来设置这个模板:

```java
// test annotations
class ManualTransactionIntegrationTest {

    @Autowired
    private PlatformTransactionManager transactionManager;

    private TransactionTemplate transactionTemplate;

    @BeforeEach
    void setUp() {
        transactionTemplate = new TransactionTemplate(transactionManager);
    }

    // omitted
}
```

`PlatformTransactionManager `帮助模板创建、提交或回滚事务。

当使用 Spring Boot 时，会自动注册一个合适的类型为`PlatformTransactionManager `的 bean，所以我们只需要简单地注入它。否则，我们应该[手动注册](/web/20220926190313/https://www.baeldung.com/transaction-configuration-with-jpa-and-spring)一个`PlatformTransactionManager ` bean。

### 3.1.样本域模型

从现在开始，为了便于演示，我们将使用一个简化的支付域模型。

在这个简单的域中，我们有一个`Payment `实体来封装每笔支付的详细信息:

```java
@Entity
public class Payment {

    @Id
    @GeneratedValue
    private Long id;

    private Long amount;

    @Column(unique = true)
    private String referenceNumber;

    @Enumerated(EnumType.STRING)
    private State state;

    // getters and setters

    public enum State {
        STARTED, FAILED, SUCCESSFUL
    }
}
```

此外，我们将运行测试类中的所有测试，在每个测试用例之前，使用 [Testcontainers](/web/20220926190313/https://www.baeldung.com/spring-boot-testcontainers-integration-test) 库运行 PostgreSQL 实例:

```java
@DataJpaTest
@Testcontainers
@ActiveProfiles("test")
@AutoConfigureTestDatabase(replace = NONE)
@Transactional(propagation = NOT_SUPPORTED) // we're going to handle transactions manually
public class ManualTransactionIntegrationTest {

    @Autowired 
    private PlatformTransactionManager transactionManager;

    @Autowired 
    private EntityManager entityManager;

    @Container
    private static PostgreSQLContainer<?> pg = initPostgres();

    private TransactionTemplate transactionTemplate;

    @BeforeEach
    public void setUp() {
        transactionTemplate = new TransactionTemplate(transactionManager);
    }

    // tests

    private static PostgreSQLContainer<?> initPostgres() {
        PostgreSQLContainer<?> pg = new PostgreSQLContainer<>("postgres:11.1")
                .withDatabaseName("baeldung")
                .withUsername("test")
                .withPassword("test");
        pg.setPortBindings(singletonList("54320:5432"));

        return pg;
    }
}
```

### 3.2.有结果的交易

`TransactionTemplate `提供了一个名为`[execute](https://web.archive.org/web/20220926190313/https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/transaction/support/TransactionTemplate.html#execute-org.springframework.transaction.support.TransactionCallback-)`的方法，它可以在一个事务中运行任何给定的代码块，然后返回一些结果:

```java
@Test
void givenAPayment_WhenNotDuplicate_ThenShouldCommit() {
    Long id = transactionTemplate.execute(status -> {
        Payment payment = new Payment();
        payment.setAmount(1000L);
        payment.setReferenceNumber("Ref-1");
        payment.setState(Payment.State.SUCCESSFUL);

        entityManager.persist(payment);

        return payment.getId();
    });

    Payment payment = entityManager.find(Payment.class, id);
    assertThat(payment).isNotNull();
}
```

这里我们将新的`Payment `实例持久化到数据库中，然后返回它自动生成的 id。

与声明性方法类似，**模板可以为我们保证原子性**。

如果事务中的一个操作未能完成，它会回滚所有操作:

```java
@Test
void givenTwoPayments_WhenRefIsDuplicate_ThenShouldRollback() {
    try {
        transactionTemplate.execute(status -> {
            Payment first = new Payment();
            first.setAmount(1000L);
            first.setReferenceNumber("Ref-1");
            first.setState(Payment.State.SUCCESSFUL);

            Payment second = new Payment();
            second.setAmount(2000L);
            second.setReferenceNumber("Ref-1"); // same reference number
            second.setState(Payment.State.SUCCESSFUL);

            entityManager.persist(first); // ok
            entityManager.persist(second); // fails

            return "Ref-1";
        });
    } catch (Exception ignored) {}

    assertThat(entityManager.createQuery("select p from Payment p").getResultList()).isEmpty();
}
```

**由于第二个`referenceNumber `是重复的，数据库拒绝第二个持久化操作，导致整个事务回滚。**因此，数据库不包含交易后的任何支付。

也可以通过调用`[TransactionStatus](https://web.archive.org/web/20220926190313/https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/transaction/TransactionStatus.html)`上的`setRollbackOnly() `来手动触发回滚:

```java
@Test
void givenAPayment_WhenMarkAsRollback_ThenShouldRollback() {
    transactionTemplate.execute(status -> {
        Payment payment = new Payment();
        payment.setAmount(1000L);
        payment.setReferenceNumber("Ref-1");
        payment.setState(Payment.State.SUCCESSFUL);

        entityManager.persist(payment);
        status.setRollbackOnly();

        return payment.getId();
    });

    assertThat(entityManager.createQuery("select p from Payment p").getResultList()).isEmpty();
}
```

### 3.3.没有结果的交易

如果我们不打算从事务中返回任何东西，我们可以使用 [`TransactionCallbackWithoutResult`](https://web.archive.org/web/20220926190313/https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/transaction/support/TransactionCallbackWithoutResult.html) 回调类:

```java
@Test
void givenAPayment_WhenNotExpectingAnyResult_ThenShouldCommit() {
    transactionTemplate.execute(new TransactionCallbackWithoutResult() {
        @Override
        protected void doInTransactionWithoutResult(TransactionStatus status) {
            Payment payment = new Payment();
            payment.setReferenceNumber("Ref-1");
            payment.setState(Payment.State.SUCCESSFUL);

            entityManager.persist(payment);
        }
    });

    assertThat(entityManager.createQuery("select p from Payment p").getResultList()).hasSize(1);
}
```

### 3.4.自定义交易配置

到目前为止，我们使用默认配置的`TransactionTemplate `。尽管这个缺省值在大多数情况下已经足够了，但是仍然可以更改配置设置。

让我们设置[事务隔离级别](https://web.archive.org/web/20220926190313/https://en.wikipedia.org/wiki/Isolation_(database_systems)):

```java
transactionTemplate = new TransactionTemplate(transactionManager);
transactionTemplate.setIsolationLevel(TransactionDefinition.ISOLATION_REPEATABLE_READ);
```

同样，我们可以更改事务传播行为:

```java
transactionTemplate.setPropagationBehavior(TransactionDefinition.PROPAGATION_REQUIRES_NEW);
```

或者我们可以为事务设置一个超时时间，以秒为单位:

```java
transactionTemplate.setTimeout(1000);
```

甚至可以从只读事务的优化中获益:

```java
transactionTemplate.setReadOnly(true);
```

一旦我们用一个配置创建了一个`TransactionTemplate `，所有的事务都将使用这个配置来执行。所以，**如果我们需要多个配置，我们应该创建多个模板实例。**

## 4.使用`PlatformTransactionManager`

除了`TransactionTemplate, `之外，我们可以使用一个更低级的 API，比如 [`PlatformTransactionManager`](https://web.archive.org/web/20220926190313/https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/transaction/PlatformTransactionManager.html) 来手动管理事务。非常有趣的是，`@Transactional `和`TransactionTemplate `都使用这个 API 来管理他们的内部事务。

### 4.1.配置交易

在使用这个 API 之前，我们应该定义我们的事务将会是什么样子。

让我们用可重复读取事务隔离级别设置一个三秒钟的超时:

```java
DefaultTransactionDefinition definition = new DefaultTransactionDefinition();
definition.setIsolationLevel(TransactionDefinition.ISOLATION_REPEATABLE_READ);
definition.setTimeout(3); 
```

交易定义类似于`TransactionTemplate `配置。然而，**我们可以用一个** `**PlatformTransactionManager**` **来使用多个定义。**

### 4.2.维护交易

在配置了我们的事务之后，我们可以以编程方式管理事务:

```java
@Test
void givenAPayment_WhenUsingTxManager_ThenShouldCommit() {

    // transaction definition

    TransactionStatus status = transactionManager.getTransaction(definition);
    try {
        Payment payment = new Payment();
        payment.setReferenceNumber("Ref-1");
        payment.setState(Payment.State.SUCCESSFUL);

        entityManager.persist(payment);
        transactionManager.commit(status);
    } catch (Exception ex) {
        transactionManager.rollback(status);
    }

    assertThat(entityManager.createQuery("select p from Payment p").getResultList()).hasSize(1);
}
```

## 5.结论

在本文中，我们首先看到了何时应该选择编程式事务管理而不是声明式方法。

然后，通过引入两种不同的 API，我们学习了如何手动创建、提交或回滚任何给定的事务。

像往常一样，GitHub 上的示例代码[可用。](https://web.archive.org/web/20220926190313/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-data-jpa-annotations)