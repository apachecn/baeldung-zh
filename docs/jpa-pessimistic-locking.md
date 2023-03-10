# JPA 中的悲观锁定

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jpa-pessimistic-locking>

## 1.概观

在很多情况下，我们都想从数据库中检索数据。有时我们想为自己锁定它，以便进一步处理，这样就没有人能打断我们的操作。

我们可以想到两种允许我们这样做的并发控制机制:设置适当的事务隔离级别，或者对我们当前需要的数据设置锁。

事务隔离是为数据库连接定义的。我们可以将其配置为保留不同程度的锁定数据。

然而，**一旦创建了连接**，就设置了隔离级别，并且它会影响该连接中的每个语句。幸运的是，我们可以使用悲观锁定，它使用数据库机制来保留对数据的更细粒度的独占访问。

我们可以使用悲观锁来确保没有其他事务可以修改或删除保留的数据。

我们可以保留两种类型的锁:独占锁和共享锁。当其他人持有共享锁时，我们可以读取数据，但不能写入数据。为了修改或删除保留的数据，我们需要有一个排他锁。

我们可以使用'`SELECT … FOR UPDATE`'语句获得独占锁。

## 2.锁定模式

JPA 规范定义了我们将要讨论的三种悲观锁模式:

*   `PESSIMISTIC_READ`–允许我们获得共享锁，防止数据被更新或删除
*   `PESSIMISTIC_WRITE`–允许我们获得排他锁，防止数据被读取、更新或删除
*   `PESSIMISTIC_FORCE_INCREMENT`–类似于`PESSIMISTIC_WRITE`工作，它还增加了一个版本化实体的版本属性

它们都是`LockModeType`类的静态成员，允许事务获得数据库锁。它们都被保留，直到事务提交或回滚。

**值得注意的是，我们一次只能获得一个锁。如果不可能，抛出一个`PersistenceException`。**

### 2.1.`PESSIMISTIC_READ`

每当我们希望只读取数据而不遇到脏读取时，我们可以使用`PESSIMISTIC_READ`(共享锁)。**我们将无法进行任何更新或删除。**

有时我们使用的数据库不支持`PESSIMISTIC_READ` 锁，所以我们有可能获得`PESSIMISTIC_WRITE`锁。

### 2.2.`PESSIMISTIC_WRITE`

任何需要获得数据锁并对其进行更改的事务都应该获得`PESSIMISTIC_WRITE`锁。根据`JPA`规范，持有`PESSIMISTIC_WRITE`锁将阻止其他事务读取、更新或删除数据。

**请注意，一些数据库系统实现了[多版本并发控制](https://web.archive.org/web/20220629004106/https://en.wikipedia.org/wiki/Multiversion_concurrency_control)，允许读者读取已经被封锁的数据。**

### 2.3.`PESSIMISTIC_FORCE_INCREMENT`

这个锁的工作方式与`PESSIMISTIC_WRITE`类似，但是它是为了与版本化的实体合作而引入的——具有用`@Version`注释的属性的实体。

版本化实体的任何更新可以在获得`PESSIMISTIC_FORCE_INCREMENT`锁之前进行。**获取该锁会导致版本列的更新。**

由持久性提供者决定它是否支持未版本化实体的`PESSIMISTIC_FORCE_INCREMENT` 。如果没有，就抛出`PersistanceException` `.`

### 2.4.例外

知道在使用悲观锁时可能会发生哪个异常是很好的。`JPA`规范提供了不同类型的异常:

*   `PessimisticLockException`–表示获取锁或将共享锁转换为独占锁失败，并导致事务级回滚
*   `LockTimeoutException – `表示获取锁或将共享锁转换为独占锁超时，导致语句级回滚
*   `PersistanceException –` 表示出现了持久性问题。`PersistanceException` 及其子类型，除了`NoResultException`、`NonUniqueResultException,`、`QueryTimeoutException, `、**标记要回滚的活动事务。**

## 3.使用悲观锁

在单个记录或一组记录上配置悲观锁有几种可能的方法。我们来看看在 JPA 中是怎么做的。

### 3.1.发现

这可能是最直接的方法。将一个`LockModeType` 对象作为参数传递给`find`方法就足够了:

```java
entityManager.find(Student.class, studentId, LockModeType.PESSIMISTIC_READ);
```

### 3.2.询问

此外，我们还可以使用一个`Query`对象，并调用带有锁模式的`setLockMode` setter 作为参数:

```java
Query query = entityManager.createQuery("from Student where studentId = :studentId");
query.setParameter("studentId", studentId);
query.setLockMode(LockModeType.PESSIMISTIC_WRITE);
query.getResultList()
```

### 3.3.显式锁定

也可以手动锁定 find 方法检索的结果:

```java
Student resultStudent = entityManager.find(Student.class, studentId);
entityManager.lock(resultStudent, LockModeType.PESSIMISTIC_WRITE);
```

### 3.4.恢复精神

如果我们想通过`refresh `方法覆盖实体的状态，我们也可以设置一个锁:

```java
Student resultStudent = entityManager.find(Student.class, studentId);
entityManager.refresh(resultStudent, LockModeType.PESSIMISTIC_FORCE_INCREMENT);
```

### 3.5\. NamedQuery

`@NamedQuery`注释也允许我们设置锁定模式:

```java
@NamedQuery(name="lockStudent",
  query="SELECT s FROM Student s WHERE s.id LIKE :studentId",
  lockMode = PESSIMISTIC_READ)
```

## 4.锁定范围

**锁定范围参数定义如何处理被锁定实体的锁定关系。**可以只在查询中定义的单个实体上获得锁，或者另外阻塞它的关系。

要配置作用域，我们可以使用`PessimisticLockScope` enum。它包含两个值:`NORMAL`和`EXTENDED`。

我们可以通过将带有`PessimisticLockScope`值的参数“`javax.persistance.lock.scope`”作为参数传递给`EntityManager`、`Query`、`TypedQuery`或`NamedQuery`的适当方法来设置范围:

```java
Map<String, Object> properties = new HashMap<>();
map.put("javax.persistence.lock.scope", PessimisticLockScope.EXTENDED);

entityManager.find(
  Student.class, 1L, LockModeType.PESSIMISTIC_WRITE, properties); 
```

### 4.1.`PessimisticLockScope.NORMAL`

**要知道`PessimisticLockScope.NORMAL`是默认范围。**有了这个锁定范围，我们就锁定了实体本身。当与联合继承一起使用时，它也锁定祖先。

让我们看看包含两个实体的示例代码:

```java
@Entity
@Inheritance(strategy = InheritanceType.JOINED)
public class Person {

    @Id
    private Long id;
    private String name;
    private String lastName;

    // getters and setters
}

@Entity
public class Employee extends Person {

    private BigDecimal salary;

    // getters and setters
}
```

当我们想要获得对`Employee`的锁定时，我们可以观察跨越这两个实体的`SQL`查询:

```java
SELECT t0.ID, t0.DTYPE, t0.LASTNAME, t0.NAME, t1.ID, t1.SALARY 
FROM PERSON t0, EMPLOYEE t1 
WHERE ((t0.ID = ?) AND ((t1.ID = t0.ID) AND (t0.DTYPE = ?))) FOR UPDATE
```

### 4.2.`PessimisticLockScope.EXTENDED`

`EXTENDED`作用域覆盖了与`NORMAL` `.` 相同的功能，此外，**它能够阻塞连接表**中的相关实体。

简单地说，它适用于标注有`@ElementCollection`或`@OneToOne`、`@OneToMany`等的实体。用`@JoinTable`。

让我们看看带有`@ElementCollection`注释的示例代码:

```java
@Entity
public class Customer {

    @Id
    private Long customerId;
    private String name;
    private String lastName;
    @ElementCollection
    @CollectionTable(name = "customer_address")
    private List<Address> addressList;

    // getters and setters
}

@Embeddable
public class Address {

    private String country;
    private String city;

    // getters and setters
}
```

让我们分析搜索`Customer`实体时的一些查询:

```java
SELECT CUSTOMERID, LASTNAME, NAME 
FROM CUSTOMER WHERE (CUSTOMERID = ?) FOR UPDATE

SELECT CITY, COUNTRY, Customer_CUSTOMERID 
FROM customer_address 
WHERE (Customer_CUSTOMERID = ?) FOR UPDATE
```

我们可以看到有两个'`FOR UPDATE`'查询锁定了 customer 表中的一行以及 join 表中的一行。

我们应该知道的另一个有趣的事实是，并不是所有的持久性提供者都支持锁定范围。

## 5.设置锁定超时

除了设置锁定范围，我们还可以调整另一个锁定参数——超时。**超时值是在`LockTimeoutException`发生之前，我们希望等待获得锁的毫秒数。**

我们可以像锁定作用域一样，通过使用属性'`javax.persistence.lock.timeout'`,以适当的毫秒数来更改超时值。

还可以通过将超时值更改为零来指定“无等待”锁定。然而，我们应该记住，有些数据库驱动程序**不支持以这种方式设置超时值。**

```java
Map<String, Object> properties = new HashMap<>(); 
map.put("javax.persistence.lock.timeout", 1000L); 

entityManager.find(
  Student.class, 1L, LockModeType.PESSIMISTIC_READ, properties);
```

## 6.结论

当设置适当的隔离级别不足以应对并发事务时，JPA 给了我们悲观锁定。它使我们能够隔离和编排不同的事务，使它们不会同时访问相同的资源。

为了实现这一点，我们可以在所讨论的锁类型之间进行选择，并因此修改诸如它们的作用域或超时之类的参数。

另一方面，我们应该记住，理解数据库锁与理解底层数据库系统的机制一样重要。记住悲观锁的行为取决于我们使用的持久性提供者也很重要。

最后，本教程的源代码可以在 GitHub 上找到，用于 [hibernate](https://web.archive.org/web/20220629004106/https://github.com/eugenp/tutorials/tree/master/persistence-modules/hibernate-jpa) 和 [EclipseLink](https://web.archive.org/web/20220629004106/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-data-eclipselink) 。