# 在 JPA 中使用惰性元素集合

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-jpa-lazy-collections>

 ![announcement - icon](img/a1e2b788f68b6fad7e7525b6de9d42ea.png)

JPA can behave very differently depending on the exact circumstances under which it is used. Code that works in our local environment or in staging performs very poorly (or even flat out fails) when thrown against real-scale databases in production environments.

Debugging these JPA issues in production is pretty difficult - existing APMs don’t provide enough granular insights at the code level, and tracking every single place someone queried entities one by one instead of in bulk can be a grueling, time-consuming task.

**Lightrun** is a new approach to debugging in production. Using Lightrun’s Logs and Snapshots, you can now get debugger-level granularity in production without opening inbound ports, redeploying, restarting, or even stropping the running application.

In addition, instrumenting Lightrun Metrics at runtime allows you to track down persistence issues securely and in real-time. Want to see it in action? **Check out our 2-minute tutorial** for debugging JPA performance issues in production using Lightrun:

[>> Debugging Spring Persistence and JPA Issues Using Lightrun](/web/20220525125951/https://www.baeldung.com/lightrun-n-jpa)

## 1.概观

JPA 规范提供了两种不同的获取策略:渴望和懒惰。虽然惰性方法有助于避免不必要地加载我们不需要的数据，但有时我们需要读取最初没有加载到封闭的持久化上下文[中的数据](/web/20220525125951/https://www.baeldung.com/jpa-hibernate-persistence-context)。此外，在封闭的持久化上下文中访问惰性元素集合是一个常见的问题。

在本教程中，我们将关注如何从惰性元素集合中加载数据。我们将探索三种不同的解决方案:一种涉及 JPA 查询语言，另一种使用实体图，最后一种使用事务传播。

## 2.元素收集问题

**默认情况下，JPA 在`@ElementCollection`类型的关联中使用延迟获取策略。**因此，在一个封闭的持久性上下文中对集合的任何访问都会导致一个异常。

为了理解这个问题，让我们根据雇员和它的电话列表之间的关系定义一个域模型:

```
@Entity
public class Employee {
    @Id
    private int id;
    private String name;
    @ElementCollection
    @CollectionTable(name = "employee_phone", joinColumns = @JoinColumn(name = "employee_id"))
    private List phones;

    // standard constructors, getters, and setters
}

@Embeddable
public class Phone {
    private String type;
    private String areaCode;
    private String number;

    // standard constructors, getters, and setters
}
```

我们的模型指定一个雇员可以有许多电话。电话列表是一个可嵌入类型 的 **[集合。让我们在这个模型中使用 Spring 存储库:](/web/20220525125951/https://www.baeldung.com/jpa-tagging-advanced)**

```
@Repository
public class EmployeeRepository {

    public Employee findById(int id) {
        return em.find(Employee.class, id);
    }

    // additional properties and auxiliary methods
} 
```

现在，让我们用一个简单的 JUnit 测试用例来重现这个问题:

```
public class ElementCollectionIntegrationTest {

    @Before
    public void init() {
        Employee employee = new Employee(1, "Fred");
        employee.setPhones(
          Arrays.asList(new Phone("work", "+55", "99999-9999"), new Phone("home", "+55", "98888-8888")));
        employeeRepository.save(employee);
    }

    @After
    public void clean() {
        employeeRepository.remove(1);
    }

    @Test(expected = org.hibernate.LazyInitializationException.class)
    public void whenAccessLazyCollection_thenThrowLazyInitializationException() {
        Employee employee = employeeRepository.findById(1);

        assertThat(employee.getPhones().size(), is(2));
    }
} 
```

当我们试图访问电话列表时，这个测试**抛出一个异常，因为持久化上下文是关闭的**。

我们可以通过**改变`@ElementCollection`的获取策略来使用急切方法**来解决这个问题。然而，急切地获取数据并不一定是最好的解决方案，因为无论我们是否需要，手机数据总是会被加载。

## 3.用 JPA 查询语言加载数据

**JPA 查询语言允许我们定制投影信息。**因此，我们可以在我们的`EmployeeRepository`中定义一个新方法来选择员工及其电话:

```
public Employee findByJPQL(int id) {
    return em.createQuery("SELECT u FROM Employee AS u JOIN FETCH u.phones WHERE u.id=:id", Employee.class)
        .setParameter("id", id).getSingleResult();
} 
```

上面的查询**使用一个内部连接操作来获取返回的每个雇员的电话列表**。

## 4.用实体图加载数据

另一个可能的解决方案是使用 JPA 的[实体图特性](/web/20220525125951/https://www.baeldung.com/jpa-entity-graph)。**实体图使我们能够选择 JPA 查询将投影哪些字段。**让我们在存储库中再定义一个方法:

```
public Employee findByEntityGraph(int id) {
    EntityGraph entityGraph = em.createEntityGraph(Employee.class);
    entityGraph.addAttributeNodes("name", "phones");
    Map<String, Object> properties = new HashMap<>();
    properties.put("javax.persistence.fetchgraph", entityGraph);
    return em.find(Employee.class, id, properties);
} 
```

我们可以看到**我们的实体图包括两个属性:姓名和电话**。因此，当 JPA 将其转换为 SQL 时，它会投影相关的列。

## 5.在事务范围内加载数据

最后，我们将探索最后一个解决方案。到目前为止，我们已经看到这个问题与持久性上下文生命周期有关。

所发生的是**我们的持久化上下文是[事务范围的](/web/20220525125951/https://www.baeldung.com/jpa-hibernate-persistence-context#transaction_persistence_context)，并且将保持打开直到事务完成**。事务生命周期从存储库方法执行的开始到结束。

因此，让我们创建另一个测试用例，并将我们的持久性上下文配置为绑定到由我们的测试方法启动的事务。我们将保持持久性上下文开放，直到测试结束:

```
@Test
@Transactional
public void whenUseTransaction_thenFetchResult() {
    Employee employee = employeeRepository.findById(1);
    assertThat(employee.getPhones().size(), is(2));
} 
```

**`@Transactional`注释围绕相关测试类的实例配置一个事务代理。**此外，事务与执行它的线程相关联。考虑到默认的事务传播设置，从该方法创建的每个持久性上下文都加入到同一个事务中。因此，事务持久性上下文被绑定到测试方法的事务范围。

## 6.结论

在本教程中，**我们评估了三种不同的解决方案，以解决在封闭的持久性上下文中从惰性关联读取数据的问题**。

首先，我们使用 JPA 查询语言来获取元素集合。接下来，我们定义了一个实体图来检索必要的数据。

并且，在最终的解决方案中，我们使用 Spring 事务来保持持久性上下文打开并读取所需的数据。

和往常一样，本教程的示例代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20220525125951/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-data-jpa-enterprise)