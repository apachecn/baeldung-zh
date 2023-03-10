# 使用 Querydsl Web 支持在多个表上使用 REST 查询语言

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/rest-querydsl-multiple-tables>

## 1。概述

在本教程中，我们将继续第二部分的 [**Spring Data Querydsl Web 支持。**](/web/20220524020913/https://www.baeldung.com/rest-api-search-querydsl-web-in-spring-data-jpa) **在这里，我们将重点介绍关联实体以及如何通过 HTTP 创建查询。**

遵循第一部分中使用的相同配置，我们将创建一个基于 Maven 的项目。请参考原文查看如何设置基础。

## 2。实体

**首先，让我们添加一个新实体(`Address) `，在用户和她的地址之间创建一个关系。**我们使用一对一的关系来保持简单。

因此，我们将有以下类:

```java
@Entity 
public class User {

    @Id 
    @GeneratedValue
    private Long id;

    private String name;

    @OneToOne(fetch = FetchType.LAZY, mappedBy = "user") 
    private Address addresses;

    // getters & setters 
} 
```

```java
@Entity 
public class Address {

    @Id 
    @GeneratedValue
    private Long id;

    private String address;

    private String country;

    @OneToOne(fetch = FetchType.LAZY) 
    @JoinColumn(name = "user_id") 
    private User user;

    // getters & setters
} 
```

## 3。Spring 数据仓库

此时，我们必须像往常一样创建 Spring 数据存储库，每个实体一个。请注意，这些存储库将具有 Querydsl 配置。

**让我们看看`AddressRepository` 存储库并解释框架配置是如何工作的:**

```java
public interface AddressRepository extends JpaRepository<Address, Long>, 
  QuerydslPredicateExecutor<Address>, QuerydslBinderCustomizer<QAddress> {

    @Override 
    default void customize(QuerydslBindings bindings, QAddress root) {
        bindings.bind(String.class)
          .first((SingleValueBinding<StringPath, String>) StringExpression::eq);
    }
} 
```

我们覆盖了`customize()`方法来配置默认绑定。在这种情况下，对于所有的`String `属性，我们将定制默认的方法绑定为 equals。

一旦存储库设置完毕，我们只需添加一个`@RestController`来管理 HTTP 查询。

## 4。查询休息控制器

在第一部分中，我们解释了对`user `存储库的查询`@RestController`，在这里，我们将重用它。

**同样，我们可能想要查询`address`表；所以对于这一点，我们只要添加一个类似的方法:**

```java
@GetMapping(value = "/addresses", produces = MediaType.APPLICATION_JSON_VALUE)
public Iterable<Address> queryOverAddress(
  @QuerydslPredicate(root = Address.class) Predicate predicate) {
    BooleanBuilder builder = new BooleanBuilder();
    return addressRepository.findAll(builder.and(predicate));
} 
```

让我们创建一些测试来看看这是如何工作的。

## 5。集成测试

我们包含了一个测试来证明`Querydsl works.` 如何做到这一点，我们使用 MockMvc 框架来模拟通过`user `的 HTTP 查询，将这个实体与新的实体`address.` 连接起来，因此，我们现在能够进行查询过滤`address` 属性。

**让我们检索居住在西班牙的所有用户:**

`/users?addresses.country=Spain `

```java
@Test
public void givenRequest_whenQueryUserFilteringByCountrySpain_thenGetJohn() throws Exception {
    mockMvc.perform(get("/users?address.country=Spain")).andExpect(status().isOk()).andExpect(content()
      .contentType(contentType))
      .andExpect(jsonPath("$", hasSize(1)))
      .andExpect(jsonPath("$[0].name", is("John")))
      .andExpect(jsonPath("$[0].address.address", is("Fake Street 1")))
      .andExpect(jsonPath("$[0].address.country", is("Spain")));
} 
```

因此，Querydsl 将映射通过 HTTP 发送的谓词，并生成以下 SQL 脚本:

```java
select user0_.id as id1_1_, 
       user0_.name as name2_1_ 
from user user0_ 
      cross join address address1_ 
where user0_.id=address1_.user_id 
      and address1_.country='Spain'
```

## 6。结论

总之，我们已经看到，Querydsl 为 web 客户端提供了一种非常简单的创建动态查询的方法；这个框架的另一个强大用途。

在[第一部分](/web/20220524020913/https://www.baeldung.com/rest-api-search-querydsl-web-in-spring-data-jpa)中，我们看到了如何从一个表中检索数据；因此，现在我们可以添加连接几个表的查询，为 web 客户端提供更好的直接过滤 HTTP 请求的体验。

这个例子的实现可以在 GitHub 项目中检查[——这是一个基于 Maven 的项目，所以它应该很容易导入和运行。](https://web.archive.org/web/20220524020913/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-data-rest-querydsl)