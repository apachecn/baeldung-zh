# 用 Spring 数据更新部分数据

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-data-partial-update>

## 1.概观

Spring Data 的`CrudRespository#save` 无疑很简单，但是有一个特性可能是一个缺点:它更新表中的每一列。这就是 CRUD 中 U 的语义，但是如果我们想做一个补丁呢？

在本教程中，我们将介绍执行部分更新而不是完全更新的技术和方法。

## 2.问题

如前所述，`save()`将用提供的数据覆盖任何匹配的实体，这意味着我们不能提供部分数据。这可能会变得不方便，尤其是对于具有大量字段的较大对象。

如果我们看一下 ORM，会发现存在一些补丁:

*   Hibernate 的 [`@DynamicUpdat` e](/web/20220719012440/https://www.baeldung.com/spring-data-jpa-dynamicupdate) 注释，它动态地重写了更新查询
*   JPA 的`@Column`注释，因为我们可以使用`updatable`参数禁止更新特定的列

但是我们将带着特定的意图来处理这个问题:**我们的目的是在不依赖 ORM 的情况下为`save`方法准备我们的实体。**

## 3.我们的案子

首先，我们来构建一个 `Customer ` 实体:

```java
@Entity 
public class Customer {
    @Id 
    @GeneratedValue(strategy = GenerationType.AUTO)
    public long id;
    public String name;
    public String phone;
} 
```

然后我们定义一个简单的 CRUD 库:

```java
@Repository 
public interface CustomerRepository extends CrudRepository<Customer, Long> {
    Customer findById(long id);
}
```

最后，我们准备一个`CustomerService`:

```java
@Service 
public class CustomerService {
    @Autowired 
    CustomerRepository repo;

    public void addCustomer(String name) {
        Customer c = new Customer();
        c.name = name;
        repo.save(c);
    }	
}
```

## 4.加载和保存方法

让我们首先来看一种可能很熟悉的方法:从数据库中加载我们的实体，然后只更新我们需要的字段。这是我们可以使用的最简单的方法。

让我们在服务中添加一个方法来更新客户的联系数据。

```java
public void updateCustomerContacts(long id, String phone) {
    Customer myCustomer = repo.findById(id);
    myCustomer.phone = phone;
    repo.save(myCustomer);
}
```

我们将调用`findById`方法并检索匹配的实体。然后，我们继续更新所需的字段并保存数据。

当要更新的字段数量相对较少并且我们的实体相当简单时，这种基本技术是有效的。

如果有几十个字段需要更新，会发生什么情况？

### 4.1.映射策略

当我们的对象有大量具有不同访问级别的字段时，实现 T2 DTO 模式是很常见的。

现在假设我们的对象中有一百多个`phone`字段。像我们以前做的那样，编写一个方法将数据从 DTO 注入我们的实体，这可能很烦人，而且很难维护。

然而，我们可以使用映射策略来解决这个问题，特别是使用 [`MapStruct`](/web/20220719012440/https://www.baeldung.com/mapstruct) 实现。

让我们创建一个`CustomerDto`:

```java
public class CustomerDto {
    private long id;
    public String name;
    public String phone;
    //...
    private String phone99;
}
```

我们还将创建一个`CustomerMapper`:

```java
@Mapper(componentModel = "spring")
public interface CustomerMapper {
    void updateCustomerFromDto(CustomerDto dto, @MappingTarget Customer entity);
}
```

`@MappingTarget`注释让我们更新一个现有的对象，省去了我们写大量代码的痛苦。

`MapStruct`有一个`@BeanMapping`方法装饰器，让我们定义一个规则，在映射过程中跳过`null`值。

让我们将它添加到我们的`updateCustomerFromDto`方法接口中:

```java
@BeanMapping(nullValuePropertyMappingStrategy = NullValuePropertyMappingStrategy.IGNORE)
```

**这样，我们可以在调用 JPA `save`方法之前加载存储的实体并用 DTO 合并它们——事实上，我们将只更新修改后的值。**

因此，让我们为我们的服务添加一个方法，该方法将调用我们的映射器:

```java
public void updateCustomer(CustomerDto dto) {
    Customer myCustomer = repo.findById(dto.id);
    mapper.updateCustomerFromDto(dto, myCustomer);
    repo.save(myCustomer);
}
```

这种方法的缺点是我们不能在更新期间将`null`值传递给数据库。

### 4.2.更简单的实体

最后，请记住，我们可以从应用程序的设计阶段着手解决这个问题。

将我们的实体定义得越小越好。

让我们来看看我们的`Customer`实体。

我们将对其进行一点结构化，将所有的`phone`字段提取到`ContactPhone` 实体，并使其处于[一对多](/web/20220719012440/https://www.baeldung.com/hibernate-one-to-many)关系下:

```java
@Entity public class CustomerStructured {
    @Id 
    @GeneratedValue(strategy = GenerationType.AUTO)
    public Long id;
    public String name;
    @OneToMany(fetch = FetchType.EAGER, targetEntity=ContactPhone.class, mappedBy="customerId")    
    private List<ContactPhone> contactPhones;
}
```

代码是干净的，更重要的是，我们取得了一些成就。现在我们可以更新我们的实体，而不必检索和填充所有的`phone`数据。

处理小的有界实体允许我们只更新必要的字段。

这种方法的唯一不便之处是，我们应该有意识地设计我们的实体，而不陷入过度工程的陷阱。

## 5.自定义查询

我们可以实现的另一种方法是为部分更新定义一个自定义查询。

事实上，JPA 定义了两个注释，`[@Modifying](/web/20220719012440/https://www.baeldung.com/spring-data-jpa-modifying-annotation)`和 [`@Query`](/web/20220719012440/https://www.baeldung.com/spring-data-jpa-query) ，它们允许我们显式地编写更新语句。

我们现在可以告诉我们的应用程序在更新过程中如何操作，而不会给 ORM 带来负担。

让我们在存储库中添加我们的自定义更新方法:

```java
@Modifying
@Query("update Customer u set u.phone = :phone where u.id = :id")
void updatePhone(@Param(value = "id") long id, @Param(value = "phone") String phone); 
```

现在我们可以重写我们的更新方法:

```java
public void updateCustomerContacts(long id, String phone) {
    repo.updatePhone(id, phone);
} 
```

我们现在能够执行部分更新。只用了几行代码，并且没有改变我们的实体，我们已经实现了我们的目标。

这种技术的缺点是，我们必须为对象的每个可能的部分更新定义一个方法。

## 6.结论

部分数据更新是非常基本的操作；虽然我们可以用 ORM 来处理它，但是完全控制它有时是有利可图的。

正如我们已经看到的，我们可以预加载我们的数据，然后更新它或定义我们的自定义语句，但记住要意识到这些方法隐含的缺点以及如何克服它们。

和往常一样，这篇文章的源代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220719012440/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-data-jpa-enterprise)