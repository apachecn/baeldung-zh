# 具有 Querydsl Web 支持的 REST 查询语言

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/rest-api-search-querydsl-web-in-spring-data-jpa>

[This article is part of a series:](javascript:void(0);)[• REST Query Language with Spring and JPA Criteria](/web/20220701020622/https://www.baeldung.com/rest-search-language-spring-jpa-criteria)
[• REST Query Language with Spring Data JPA Specifications](/web/20220701020622/https://www.baeldung.com/rest-api-search-language-spring-data-specifications)
[• REST Query Language with Spring Data JPA and Querydsl](/web/20220701020622/https://www.baeldung.com/rest-api-search-language-spring-data-querydsl)
[• REST Query Language – Advanced Search Operations](/web/20220701020622/https://www.baeldung.com/rest-api-query-search-language-more-operations)
[• REST Query Language – Implementing OR Operation](/web/20220701020622/https://www.baeldung.com/rest-api-query-search-or-operation)
[• REST Query Language with RSQL](/web/20220701020622/https://www.baeldung.com/rest-api-search-language-rsql-fiql)
• REST Query Language with Querydsl Web Support (current article)

## 1。概述

在这个快速教程中，我们将讨论 Spring Data Querydsl Web 支持。

这绝对是我们在 REST 查询语言系列的[中关注的所有其他方法的一个有趣的替代。](/web/20220701020622/https://www.baeldung.com/spring-rest-api-query-search-language-tutorial)

## 2。Maven 配置

首先，让我们从我们的 maven 配置开始:

```
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.3.0.RELEASE</version>
</parent>

<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.data</groupId>
        <artifactId>spring-data-commons</artifactId>
    </dependency>
    <dependency>
        <groupId>com.mysema.querydsl</groupId>
        <artifactId>querydsl-apt</artifactId>
        <version>${querydsl.version}</version>
    </dependency>
    <dependency>
        <groupId>com.mysema.querydsl</groupId>
        <artifactId>querydsl-jpa</artifactId>
        <version>${querydsl.version}</version>
    </dependency>
...
```

请注意，从 **1.11** 开始，Querydsl web 支持在 **spring-data-commons** 中可用

## 3。用户资源库

接下来，让我们看看我们的存储库:

```
public interface UserRepository extends 
  JpaRepository<User, Long>, QueryDslPredicateExecutor<User>, QuerydslBinderCustomizer<QUser> {
    @Override
    default public void customize(QuerydslBindings bindings, QUser root) {
        bindings.bind(String.class).first(
          (StringPath path, String value) -> path.containsIgnoreCase(value));
        bindings.excluding(root.email);
    }
}
```

请注意:

*   我们正在覆盖`QuerydslBinderCustomizer` `customize()`来定制默认绑定
*   我们正在定制默认的`equals`绑定来忽略所有`String`属性的大小写
*   我们还从`Predicate`解析中排除了用户的电子邮件

点击查看完整文档[。](https://web.archive.org/web/20220701020622/https://docs.spring.io/spring-data/jpa/docs/1.9.0.RELEASE/reference/html/#core.web.type-safe)

## 4。用户控制器

现在，让我们来看看控制器:

```
@RequestMapping(method = RequestMethod.GET, value = "/users")
@ResponseBody
public Iterable<User> findAllByWebQuerydsl(
  @QuerydslPredicate(root = User.class) Predicate predicate) {
    return userRepository.findAll(predicate);
}
```

这是有趣的部分——注意**我们是如何使用`@QuerydslPredicate`注释直接从 HttpRequest** 中获得谓词的。

以下是包含这种类型查询的 URL 的外观:

```
http://localhost:8080/users?firstName=john
```

下面是一个潜在反应是如何构成的:

```
[
   {
      "id":1,
      "firstName":"john",
      "lastName":"doe",
      "email":"[[email protected]](/web/20220701020622/https://www.baeldung.com/cdn-cgi/l/email-protection)",
      "age":11
   }
]
```

## 5。现场测试

最后，让我们测试一下新的 Querydsl Web 支持:

```
@RunWith(SpringJUnit4ClassRunner.class)
@SpringApplicationConfiguration(classes = Application.class)
@WebAppConfiguration
public class UserLiveTest {

    private ObjectMapper mapper = new ObjectMapper();
    private User userJohn = new User("john", "doe", "[[email protected]](/web/20220701020622/https://www.baeldung.com/cdn-cgi/l/email-protection)");
    private User userTom = new User("tom", "doe", "[[email protected]](/web/20220701020622/https://www.baeldung.com/cdn-cgi/l/email-protection)");

    private static boolean setupDataCreated = false;

    @Before
    public void setupData() throws JsonProcessingException {
        if (!setupDataCreated) {
            givenAuth().contentType(MediaType.APPLICATION_JSON_VALUE)
                       .body(mapper.writeValueAsString(userJohn))
                       .post("http://localhost:8080/users");

            givenAuth().contentType(MediaType.APPLICATION_JSON_VALUE)
                       .body(mapper.writeValueAsString(userTom))
                       .post("http://localhost:8080/users");
            setupDataCreated = true;
        }
    }

    private RequestSpecification givenAuth() {
        return RestAssured.given().auth().preemptive().basic("user1", "user1Pass");
    }
}
```

首先，让我们获取系统中的所有用户:

```
@Test
public void whenGettingListOfUsers_thenCorrect() {
    Response response = givenAuth().get("http://localhost:8080/users");
    User[] result = response.as(User[].class);
    assertEquals(result.length, 2);
}
```

接下来，让我们通过**的名字**来查找用户:

```
@Test
public void givenFirstName_whenGettingListOfUsers_thenCorrect() {
    Response response = givenAuth().get("http://localhost:8080/users?firstName=john");
    User[] result = response.as(User[].class);
    assertEquals(result.length, 1);
    assertEquals(result[0].getEmail(), userJohn.getEmail());
}
```

接下来，以免通过**部分姓氏**找到用户:

```
@Test
public void givenPartialLastName_whenGettingListOfUsers_thenCorrect() {
    Response response = givenAuth().get("http://localhost:8080/users?lastName=do");
    User[] result = response.as(User[].class);
    assertEquals(result.length, 2);
}
```

现在，让我们试着通过**电子邮件**找到用户:

```
@Test
public void givenEmail_whenGettingListOfUsers_thenIgnored() {
    Response response = givenAuth().get("http://localhost:8080/users?email=john");
    User[] result = response.as(User[].class);
    assertEquals(result.length, 2);
}
```

注意:当我们试图通过电子邮件查找用户时，该查询被忽略，因为我们从`Predicate`解析中排除了用户的电子邮件。

## 6。结论

在本文中，我们快速介绍了 Spring Data Querydsl Web 支持，以及一种直接从 HTTP 请求中获取`Predicate`并使用它来检索数据的酷而简单的方法。

**«** Previous[REST Query Language with RSQL](/web/20220701020622/https://www.baeldung.com/rest-api-search-language-rsql-fiql)