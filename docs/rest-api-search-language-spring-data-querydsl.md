# REST 查询语言与 Spring 数据 JPA 和 Querydsl

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/rest-api-search-language-spring-data-querydsl>

 ![announcement-icon.png](img/3c0fee71b30ba6e8e6b150991641420f.png)

JPA can behave very differently depending on the exact circumstances under which it is used. Code that works in our local environment or in staging performs very poorly (or even flat out fails) when thrown against real-scale databases in production environments.

Debugging these JPA issues in production is pretty difficult - existing APMs don’t provide enough granular insights at the code level, and tracking every single place someone queried entities one by one instead of in bulk can be a grueling, time-consuming task.

**Lightrun** is a new approach to debugging in production. Using Lightrun’s Logs and Snapshots, you can now get debugger-level granularity in production without opening inbound ports, redeploying, restarting, or even stropping the running application.

In addition, instrumenting Lightrun Metrics at runtime allows you to track down persistence issues securely and in real-time. Want to see it in action? **Check out our 2-minute tutorial** for debugging JPA performance issues in production using Lightrun:

[>> Debugging Spring Persistence and JPA Issues Using Lightrun](/web/20220525000110/https://www.baeldung.com/lightrun-n-jpa)

[This article is part of a series:](javascript:void(0);)[• REST Query Language with Spring and JPA Criteria](/web/20220525000110/https://www.baeldung.com/rest-search-language-spring-jpa-criteria)
[• REST Query Language with Spring Data JPA Specifications](/web/20220525000110/https://www.baeldung.com/rest-api-search-language-spring-data-specifications)
• REST Query Language with Spring Data JPA and Querydsl (current article)[• REST Query Language – Advanced Search Operations](/web/20220525000110/https://www.baeldung.com/rest-api-query-search-language-more-operations)
[• REST Query Language – Implementing OR Operation](/web/20220525000110/https://www.baeldung.com/rest-api-query-search-or-operation)
[• REST Query Language with RSQL](/web/20220525000110/https://www.baeldung.com/rest-api-search-language-rsql-fiql)
[• REST Query Language with Querydsl Web Support](/web/20220525000110/https://www.baeldung.com/rest-api-search-querydsl-web-in-spring-data-jpa)

## 1。概述

在本教程中，我们将使用 Spring Data JPA 和[query DSL](/web/20220525000110/https://www.baeldung.com/intro-to-querydsl)为 **REST API 构建一个查询语言。**

在本系列的[的前两篇文章中，我们使用 JPA 标准和 Spring Data JPA 规范构建了相同的搜索/过滤功能。](/web/20220525000110/https://www.baeldung.com/spring-rest-api-query-search-language-tutorial)

那么，为什么要使用查询语言呢？因为——对于任何足够复杂的 API——通过非常简单的字段搜索/过滤您的资源是远远不够的。**一种更灵活的查询语言**，允许你过滤出你所需要的资源。

## 2。Querydsl 配置

首先——让我们看看如何配置我们的项目来使用 Querydsl。

我们需要将以下依赖项添加到`pom.xml`:

```
<dependency> 
    <groupId>com.querydsl</groupId> 
    <artifactId>querydsl-apt</artifactId> 
    <version>4.2.2</version>
    </dependency>
<dependency> 
    <groupId>com.querydsl</groupId> 
    <artifactId>querydsl-jpa</artifactId> 
    <version>4.2.2</version> 
</dependency>
```

我们还需要配置 APT–注释处理工具–插件，如下所示:

```
<plugin>
    <groupId>com.mysema.maven</groupId>
    <artifactId>apt-maven-plugin</artifactId>
    <version>1.1.3</version>
    <executions>
        <execution>
            <goals>
                <goal>process</goal>
            </goals>
            <configuration>
                <outputDirectory>target/generated-sources/java</outputDirectory>
                <processor>com.mysema.query.apt.jpa.JPAAnnotationProcessor</processor>
            </configuration>
        </execution>
    </executions>
</plugin>
```

**这将为我们的实体生成 [Q 类型](/web/20220525000110/https://www.baeldung.com/intro-to-querydsl#2-exploring-generated-classes)。**

## 3。`MyUser` 实体

接下来，让我们看看我们将在搜索 API 中使用的“`MyUser`”实体:

```
@Entity
public class MyUser {
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;

    private String firstName;
    private String lastName;
    private String email;

    private int age;
}
```

## 4。习俗`Predicate W`与 `PathBuilder`同

现在——让我们基于一些任意的约束创建一个自定义的`Predicate`。

我们在这里使用`PathBuilder`而不是自动生成的 Q 类型，因为我们需要为更抽象的用法动态地创建路径:

```
public class MyUserPredicate {

    private SearchCriteria criteria;

    public BooleanExpression getPredicate() {
        PathBuilder<MyUser> entityPath = new PathBuilder<>(MyUser.class, "user");

        if (isNumeric(criteria.getValue().toString())) {
            NumberPath<Integer> path = entityPath.getNumber(criteria.getKey(), Integer.class);
            int value = Integer.parseInt(criteria.getValue().toString());
            switch (criteria.getOperation()) {
                case ":":
                    return path.eq(value);
                case ">":
                    return path.goe(value);
                case "<":
                    return path.loe(value);
            }
        } 
        else {
            StringPath path = entityPath.getString(criteria.getKey());
            if (criteria.getOperation().equalsIgnoreCase(":")) {
                return path.containsIgnoreCase(criteria.getValue().toString());
            }
        }
        return null;
    }
}
```

注意谓词的实现是如何处理多种类型的操作的。这是因为根据定义，查询语言是一种开放的语言，您可以使用任何支持的操作通过任何字段进行过滤。

为了表示这种开放的过滤标准，我们使用了一个简单但非常灵活的实现—`SearchCriteria`:

```
public class SearchCriteria {
    private String key;
    private String operation;
    private Object value;
}
```

`SearchCriteria`包含了我们表示约束所需的细节:

*   `key`:字段名称，例如:`firstName`、`age`、…等
*   `operation`:运算——例如:相等、小于、…等
*   `value`:字段值-例如:约翰，25，…等

## 5。`MyUserRepository`

现在，让我们来看看我们的`MyUserRepository`。

我们需要我们的`MyUserRepository` 来扩展`QuerydslPredicateExecutor`,这样我们就可以稍后使用`Predicates`来过滤搜索结果:

```
public interface MyUserRepository extends JpaRepository<MyUser, Long>, 
  QuerydslPredicateExecutor<MyUser>, QuerydslBinderCustomizer<QMyUser> {
    @Override
    default public void customize(
      QuerydslBindings bindings, QMyUser root) {
        bindings.bind(String.class)
          .first((SingleValueBinding<StringPath, String>) StringExpression::containsIgnoreCase);
        bindings.excluding(root.email);
      }
}
```

注意，我们在这里使用为`MyUser`实体生成的 Q 类型，它将被命名为`QMyUser.`

## 6。结合`Predicates`

接下来——让我们看看在结果过滤中组合谓词来使用多个约束。

在下面的例子中，我们与一个构建者`MyUserPredicatesBuilder` 一起工作，来组合`Predicates`:

```
public class MyUserPredicatesBuilder {
    private List<SearchCriteria> params;

    public MyUserPredicatesBuilder() {
        params = new ArrayList<>();
    }

    public MyUserPredicatesBuilder with(
      String key, String operation, Object value) {

        params.add(new SearchCriteria(key, operation, value));
        return this;
    }

    public BooleanExpression build() {
        if (params.size() == 0) {
            return null;
        }

        List predicates = params.stream().map(param -> {
            MyUserPredicate predicate = new MyUserPredicate(param);
            return predicate.getPredicate();
        }).filter(Objects::nonNull).collect(Collectors.toList());

        BooleanExpression result = Expressions.asBoolean(true).isTrue();
        for (BooleanExpression predicate : predicates) {
            result = result.and(predicate);
        }        
        return result;
    }
}
```

## 7 .**。测试搜索查询**

接下来，让我们测试我们的搜索 API。

我们将从用几个用户初始化数据库开始——让这些用户准备好并可用于测试:

```
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = { PersistenceConfig.class })
@Transactional
@Rollback
public class JPAQuerydslIntegrationTest {

    @Autowired
    private MyUserRepository repo;

    private MyUser userJohn;
    private MyUser userTom;

    @Before
    public void init() {
        userJohn = new MyUser();
        userJohn.setFirstName("John");
        userJohn.setLastName("Doe");
        userJohn.setEmail("[[email protected]](/web/20220525000110/https://www.baeldung.com/cdn-cgi/l/email-protection)");
        userJohn.setAge(22);
        repo.save(userJohn);

        userTom = new MyUser();
        userTom.setFirstName("Tom");
        userTom.setLastName("Doe");
        userTom.setEmail("[[email protected]](/web/20220525000110/https://www.baeldung.com/cdn-cgi/l/email-protection)");
        userTom.setAge(26);
        repo.save(userTom);
    }
}
```

接下来，让我们看看如何查找姓氏为的用户:

```
@Test
public void givenLast_whenGettingListOfUsers_thenCorrect() {
    MyUserPredicatesBuilder builder = new MyUserPredicatesBuilder().with("lastName", ":", "Doe");

    Iterable<MyUser> results = repo.findAll(builder.build());
    assertThat(results, containsInAnyOrder(userJohn, userTom));
}
```

现在，让我们看看如何找到一个名为**，姓为**的用户:

```
@Test
public void givenFirstAndLastName_whenGettingListOfUsers_thenCorrect() {
    MyUserPredicatesBuilder builder = new MyUserPredicatesBuilder()
      .with("firstName", ":", "John").with("lastName", ":", "Doe");

    Iterable<MyUser> results = repo.findAll(builder.build());

    assertThat(results, contains(userJohn));
    assertThat(results, not(contains(userTom)));
}
```

接下来，让我们看看如何找到给定了姓氏和最小年龄的用户

```
@Test
public void givenLastAndAge_whenGettingListOfUsers_thenCorrect() {
    MyUserPredicatesBuilder builder = new MyUserPredicatesBuilder()
      .with("lastName", ":", "Doe").with("age", ">", "25");

    Iterable<MyUser> results = repo.findAll(builder.build());

    assertThat(results, contains(userTom));
    assertThat(results, not(contains(userJohn)));
}
```

现在，让我们看看如何搜索实际上不存在的**`MyUser` :**

```
@Test
public void givenWrongFirstAndLast_whenGettingListOfUsers_thenCorrect() {
    MyUserPredicatesBuilder builder = new MyUserPredicatesBuilder()
      .with("firstName", ":", "Adam").with("lastName", ":", "Fox");

    Iterable<MyUser> results = repo.findAll(builder.build());
    assertThat(results, emptyIterable());
}
```

最后，让我们看看如何在只给出名字的一部分的情况下找到一个`MyUser` **，如下例所示:**

```
@Test
public void givenPartialFirst_whenGettingListOfUsers_thenCorrect() {
    MyUserPredicatesBuilder builder = new MyUserPredicatesBuilder().with("firstName", ":", "jo");

    Iterable<MyUser> results = repo.findAll(builder.build());

    assertThat(results, contains(userJohn));
    assertThat(results, not(contains(userTom)));
}
```

## 8。`UserController`

最后，让我们把所有东西放在一起，构建 REST API。

我们正在定义一个`UserController`,它定义了一个简单的方法`findAll()`,带有一个“`search`”参数来传递查询字符串:

```
@Controller
public class UserController {

    @Autowired
    private MyUserRepository myUserRepository;

    @RequestMapping(method = RequestMethod.GET, value = "/myusers")
    @ResponseBody
    public Iterable<MyUser> search(@RequestParam(value = "search") String search) {
        MyUserPredicatesBuilder builder = new MyUserPredicatesBuilder();

        if (search != null) {
            Pattern pattern = Pattern.compile("(\w+?)(:|<|>)(\w+?),");
            Matcher matcher = pattern.matcher(search + ",");
            while (matcher.find()) {
                builder.with(matcher.group(1), matcher.group(2), matcher.group(3));
            }
        }
        BooleanExpression exp = builder.build();
        return myUserRepository.findAll(exp);
    }
}
```

下面是一个快速测试 URL 的例子:

```
http://localhost:8080/myusers?search=lastName:doe,age>25
```

回应是:

```
[{
    "id":2,
    "firstName":"tom",
    "lastName":"doe",
    "email":"[[email protected]](/web/20220525000110/https://www.baeldung.com/cdn-cgi/l/email-protection)",
    "age":26
}]
```

## 9。结论

第三篇文章讲述了为 REST API 构建查询语言的第一步，充分利用了 Querydsl 库。

当然，实现还处于早期，但是可以很容易地发展到支持更多的操作。

本文的**完整实现**可以在[的 GitHub 项目](https://web.archive.org/web/20220525000110/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-rest-query-language "The Full Example Project on Github")中找到——这是一个基于 Maven 的项目，因此应该很容易导入和运行。

Next **»**[REST Query Language – Advanced Search Operations](/web/20220525000110/https://www.baeldung.com/rest-api-query-search-language-more-operations)**«** Previous[REST Query Language with Spring Data JPA Specifications](/web/20220525000110/https://www.baeldung.com/rest-api-search-language-spring-data-specifications)**