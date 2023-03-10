# 春季数据 JPA 批量插入

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-data-jpa-batch-inserts>

## 1.概观

去数据库是昂贵的。我们可以通过将多个插入批处理为一个来提高性能和一致性。

在本教程中，我们将看看如何使用 [Spring 数据 JPA](/web/20220926183355/https://www.baeldung.com/the-persistence-layer-with-spring-data-jpa) 来实现这一点。

## 2.Spring JPA 存储库

首先，我们需要一个简单的实体。姑且称之为`Customer`:

```java
@Entity
public class Customer {

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;
    private String firstName;
    private String lastName;

    // constructor, getters, setters 
}
```

然后，我们需要我们的存储库:

```java
public interface CustomerRepository extends CrudRepository<Customer, Long> {
} 
```

**这为我们公开了一个`saveAll`方法，它将几个插入批处理成一个。**

因此，让我们在控制器中利用这一点:

```java
@RestController
public class CustomerController {   
    @Autowired
    CustomerRepository customerRepository;   

    @PostMapping("/customers")
    public ResponseEntity<String> insertCustomers() {        
        Customer c1 = new Customer("James", "Gosling");
        Customer c2 = new Customer("Doug", "Lea");
        Customer c3 = new Customer("Martin", "Fowler");
        Customer c4 = new Customer("Brian", "Goetz");
        List<Customer> customers = Arrays.asList(c1, c2, c3, c4);
        customerRepository.saveAll(customers);
        return ResponseEntity.created("/customers");
    }

    // ... @GetMapping to read customers
} 
```

## 3.测试我们的端点

用`MockMvc`测试我们的代码很简单:

```java
@Autowired
private MockMvc mockMvc;

@Test 
public void whenInsertingCustomers_thenCustomersAreCreated() throws Exception {
    this.mockMvc.perform(post("/customers"))
      .andExpect(status().isCreated()));
} 
```

## 4.我们确定要分批吗？

**所以，实际上，还有一点配置要做**——让我们做一个快速演示来说明不同之处。

首先，让我们将下面的属性添加到`application.properties `来查看一些统计数据:

```java
spring.jpa.properties.hibernate.generate_statistics=true 
```

此时，如果我们运行测试，我们将看到如下所示的统计信息:

```java
11232586 nanoseconds spent preparing 4 JDBC statements;
4076610 nanoseconds spent executing 4 JDBC statements;
0 nanoseconds spent executing 0 JDBC batches; 
```

因此，我们创建了四个客户，这很好，**但是请注意，他们都不在一个批处理中。**

**原因是在某些情况下，默认情况下批处理是不打开的。**

在我们的例子中，这是因为我们使用了 id 自动生成。因此，**默认情况下，`saveAll` 分别插入。**

所以，让我们打开它:

```java
spring.jpa.properties.hibernate.jdbc.batch_size=4
spring.jpa.properties.hibernate.order_inserts=true 
```

第一个属性告诉 Hibernate 以四个为一批收集插入。属性告诉 Hibernate 花时间按实体对插入进行分组，创建更大的批处理。

**因此，第二次运行测试时，我们会看到插入被分批:**

```java
16577314 nanoseconds spent preparing 4 JDBC statements;
2207548 nanoseconds spent executing 4 JDBC statements;
2003005 nanoseconds spent executing 1 JDBC batches; 
```

我们可以对删除和更新应用相同的方法(**记住 Hibernate 也有一个`order_updates `属性**)。

## 5.结论

有了批量插入的能力，我们可以看到一些性能提升。

当然，我们需要意识到批处理在某些情况下是自动禁用的，我们应该在发布之前检查并计划好。

请务必在 GitHub 上查看所有这些代码片段[。](https://web.archive.org/web/20220926183355/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-data-jpa-crud)