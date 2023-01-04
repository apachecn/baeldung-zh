# 具有 Spring 数据 JPA 规范的 REST 查询语言

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/rest-api-search-language-spring-data-specifications>

[This article is part of a series:](javascript:void(0);)[• REST Query Language with Spring and JPA Criteria](/web/20220926194244/https://www.baeldung.com/rest-search-language-spring-jpa-criteria)
• REST Query Language with Spring Data JPA Specifications (current article)[• REST Query Language with Spring Data JPA and Querydsl](/web/20220926194244/https://www.baeldung.com/rest-api-search-language-spring-data-querydsl)
[• REST Query Language – Advanced Search Operations](/web/20220926194244/https://www.baeldung.com/rest-api-query-search-language-more-operations)
[• REST Query Language – Implementing OR Operation](/web/20220926194244/https://www.baeldung.com/rest-api-query-search-or-operation)
[• REST Query Language with RSQL](/web/20220926194244/https://www.baeldung.com/rest-api-search-language-rsql-fiql)
[• REST Query Language with Querydsl Web Support](/web/20220926194244/https://www.baeldung.com/rest-api-search-querydsl-web-in-spring-data-jpa)

## 1。概述

在本教程中，我们将使用 Spring Data JPA 和规范构建一个**搜索/过滤 REST API** 。

我们在[这个系列](/web/20220926194244/https://www.baeldung.com/spring-rest-api-query-search-language-tutorial)的[第一篇文章](/web/20220926194244/https://www.baeldung.com/rest-search-language-spring-jpa-criteria "REST Query Language with Spring and JPA Criteria")中开始研究一种带有基于 JPA 标准的解决方案的查询语言。

那么，**为什么是查询语言呢？因为通过非常简单的字段搜索/过滤我们的资源对于太复杂的 API 来说是不够的。**查询语言更加灵活**，允许我们筛选出我们需要的资源。**

## 2。`User`实体

首先，让我们从搜索 API 的一个简单的`User`实体开始:

```
@Entity
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;

    private String firstName;
    private String lastName;
    private String email;

    private int age;

    // standard getters and setters
}
```

## 3。使用`Specification`过滤

现在让我们直接进入问题中最有趣的部分:使用定制的 Spring 数据 JPA `Specifications`进行查询。

我们将创建一个实现`Specification`接口的`UserSpecification`，并且我们将**传入我们自己的约束来构造实际的查询**:

```
public class UserSpecification implements Specification<User> {

    private SearchCriteria criteria;

    @Override
    public Predicate toPredicate
      (Root<User> root, CriteriaQuery<?> query, CriteriaBuilder builder) {

        if (criteria.getOperation().equalsIgnoreCase(">")) {
            return builder.greaterThanOrEqualTo(
              root.<String> get(criteria.getKey()), criteria.getValue().toString());
        } 
        else if (criteria.getOperation().equalsIgnoreCase("<")) {
            return builder.lessThanOrEqualTo(
              root.<String> get(criteria.getKey()), criteria.getValue().toString());
        } 
        else if (criteria.getOperation().equalsIgnoreCase(":")) {
            if (root.get(criteria.getKey()).getJavaType() == String.class) {
                return builder.like(
                  root.<String>get(criteria.getKey()), "%" + criteria.getValue() + "%");
            } else {
                return builder.equal(root.get(criteria.getKey()), criteria.getValue());
            }
        }
        return null;
    }
}
```

正如我们所看到的，**我们基于一些简单的约束**创建了一个`Specification`，我们在下面的`SearchCriteria`类中表示它:

```
public class SearchCriteria {
    private String key;
    private String operation;
    private Object value;
}
```

`SearchCriteria`实现持有一个约束的基本表示，我们将基于这个约束来构造查询:

*   `key`:字段名称，例如`firstName`、`age`等。
*   `operation`:运算，例如，相等，小于等。
*   `value`:字段值，例如约翰，25 等。

当然，实现很简单，还可以改进。然而，它是我们需要的强大而灵活的操作的坚实基础。

## 4。`UserRepository`

接下来，我们来看看`UserRepository`。

我们只是扩展了`JpaSpecificationExecutor`来获得新的规范 API:

```
public interface UserRepository 
  extends JpaRepository<User, Long>, JpaSpecificationExecutor<User> {}
```

## 5。测试搜索查询

现在让我们测试一下新的搜索 API。

首先，让我们创建一些用户，让他们在测试运行时做好准备:

```
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = { PersistenceJPAConfig.class })
@Transactional
@TransactionConfiguration
public class JPASpecificationIntegrationTest {

    @Autowired
    private UserRepository repository;

    private User userJohn;
    private User userTom;

    @Before
    public void init() {
        userJohn = new User();
        userJohn.setFirstName("John");
        userJohn.setLastName("Doe");
        userJohn.setEmail("[[email protected]](/web/20220926194244/https://www.baeldung.com/cdn-cgi/l/email-protection)");
        userJohn.setAge(22);
        repository.save(userJohn);

        userTom = new User();
        userTom.setFirstName("Tom");
        userTom.setLastName("Doe");
        userTom.setEmail("[[email protected]](/web/20220926194244/https://www.baeldung.com/cdn-cgi/l/email-protection)");
        userTom.setAge(26);
        repository.save(userTom);
    }
}
```

接下来，让我们看看如何查找姓氏为的用户:

```
@Test
public void givenLast_whenGettingListOfUsers_thenCorrect() {
    UserSpecification spec = 
      new UserSpecification(new SearchCriteria("lastName", ":", "doe"));

    List<User> results = repository.findAll(spec);

    assertThat(userJohn, isIn(results));
    assertThat(userTom, isIn(results));
}
```

现在，我们将找到一个名为**，姓为**的用户:

```
@Test
public void givenFirstAndLastName_whenGettingListOfUsers_thenCorrect() {
    UserSpecification spec1 = 
      new UserSpecification(new SearchCriteria("firstName", ":", "john"));
    UserSpecification spec2 = 
      new UserSpecification(new SearchCriteria("lastName", ":", "doe"));

    List<User> results = repository.findAll(Specification.where(spec1).and(spec2));

    assertThat(userJohn, isIn(results));
    assertThat(userTom, not(isIn(results)));
}
```

注:我们用`where`和`and`到**组合规格。**

接下来，让我们找到一个具有给定的**姓氏和最小年龄**的用户:

```
@Test
public void givenLastAndAge_whenGettingListOfUsers_thenCorrect() {
    UserSpecification spec1 = 
      new UserSpecification(new SearchCriteria("age", ">", "25"));
    UserSpecification spec2 = 
      new UserSpecification(new SearchCriteria("lastName", ":", "doe"));

    List<User> results = 
      repository.findAll(Specification.where(spec1).and(spec2));

    assertThat(userTom, isIn(results));
    assertThat(userJohn, not(isIn(results)));
}
```

现在我们来看看如何搜索一个**实际上并不存在的`User`**:

```
@Test
public void givenWrongFirstAndLast_whenGettingListOfUsers_thenCorrect() {
    UserSpecification spec1 = 
      new UserSpecification(new SearchCriteria("firstName", ":", "Adam"));
    UserSpecification spec2 = 
      new UserSpecification(new SearchCriteria("lastName", ":", "Fox"));

    List<User> results = 
      repository.findAll(Specification.where(spec1).and(spec2));

    assertThat(userJohn, not(isIn(results)));
    assertThat(userTom, not(isIn(results)));  
}
```

最后，我们将找到一个仅给出名字一部分的`User` **:**

```
@Test
public void givenPartialFirst_whenGettingListOfUsers_thenCorrect() {
    UserSpecification spec = 
      new UserSpecification(new SearchCriteria("firstName", ":", "jo"));

    List<User> results = repository.findAll(spec);

    assertThat(userJohn, isIn(results));
    assertThat(userTom, not(isIn(results)));
}
```

## 6。结合`Specifications`

接下来，让我们看看如何结合我们的自定义`Specifications` 来使用多个约束，并根据多个标准进行过滤。

我们将实现一个生成器— `UserSpecificationsBuilder` —来轻松流畅地组合`Specifications`:

```
public class UserSpecificationsBuilder {

    private final List<SearchCriteria> params;

    public UserSpecificationsBuilder() {
        params = new ArrayList<SearchCriteria>();
    }

    public UserSpecificationsBuilder with(String key, String operation, Object value) {
        params.add(new SearchCriteria(key, operation, value));
        return this;
    }

    public Specification<User> build() {
        if (params.size() == 0) {
            return null;
        }

        List<Specification> specs = params.stream()
          .map(UserSpecification::new)
          .collect(Collectors.toList());

        Specification result = specs.get(0);

        for (int i = 1; i < params.size(); i++) {
            result = params.get(i)
              .isOrPredicate()
                ? Specification.where(result)
                  .or(specs.get(i))
                : Specification.where(result)
                  .and(specs.get(i));
        }       
        return result;
    }
}
```

## 7。`UserController`

最后，让我们使用这个新的持久性搜索/过滤功能，**通过一个简单的`search`操作创建一个`UserController`来设置 REST API** :

```
@Controller
public class UserController {

    @Autowired
    private UserRepository repo;

    @RequestMapping(method = RequestMethod.GET, value = "/users")
    @ResponseBody
    public List<User> search(@RequestParam(value = "search") String search) {
        UserSpecificationsBuilder builder = new UserSpecificationsBuilder();
        Pattern pattern = Pattern.compile("(\\w+?)(:|<|>)(\\w+?),");
        Matcher matcher = pattern.matcher(search + ",");
        while (matcher.find()) {
            builder.with(matcher.group(1), matcher.group(2), matcher.group(3));
        }

        Specification<User> spec = builder.build();
        return repo.findAll(spec);
    }
}
```

注意，为了支持其他非英语系统，`Pattern`对象可以更改为:

```
Pattern pattern = Pattern.compile("(\\w+?)(:|<|>)(\\w+?),", Pattern.UNICODE_CHARACTER_CLASS);
```

下面是一个测试 API 的测试 URL:

```
http://localhost:8080/users?search=lastName:doe,age>25
```

下面是我的回答:

```
[{
    "id":2,
    "firstName":"tom",
    "lastName":"doe",
    "email":"[[email protected]](/web/20220926194244/https://www.baeldung.com/cdn-cgi/l/email-protection)",
    "age":26
}]
```

**在我们的`Pattern`示例中，由于搜索被一个“，”分开，所以搜索词不能包含这个字符。**该模式也不匹配空白。

如果我们想搜索包含逗号的值，我们可以考虑使用不同的分隔符，如“；”。

另一种选择是改变模式，搜索引号之间的值，然后从搜索词中去掉这些值:

```
Pattern pattern = Pattern.compile("(\\w+?)(:|<|>)(\"([^\"]+)\")");
```

## 8。结论

本文介绍了一个简单的实现，它可以作为强大的 REST 查询语言的基础。

我们很好地利用了 Spring 数据规范，以确保 API 远离领域，并且**可以选择处理许多其他类型的操作。**

该条的**完整实现**可以在[的 GitHub 项目](https://web.archive.org/web/20220926194244/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-rest-query-language "The Full Example Project on Github")中找到。这是一个基于 Maven 的项目，因此应该很容易导入和运行。

Next **»**[REST Query Language with Spring Data JPA and Querydsl](/web/20220926194244/https://www.baeldung.com/rest-api-search-language-spring-data-querydsl)**«** Previous[REST Query Language with Spring and JPA Criteria](/web/20220926194244/https://www.baeldung.com/rest-search-language-spring-jpa-criteria)