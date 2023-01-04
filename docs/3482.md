# 带有 Spring 和 JPA 标准的 REST 查询语言

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/rest-search-language-spring-jpa-criteria>

[This article is part of a series:](javascript:void(0);)• REST Query Language with Spring and JPA Criteria (current article)[• REST Query Language with Spring Data JPA Specifications](/web/20220529020840/https://www.baeldung.com/rest-api-search-language-spring-data-specifications)
[• REST Query Language with Spring Data JPA and Querydsl](/web/20220529020840/https://www.baeldung.com/rest-api-search-language-spring-data-querydsl)
[• REST Query Language – Advanced Search Operations](/web/20220529020840/https://www.baeldung.com/rest-api-query-search-language-more-operations)
[• REST Query Language – Implementing OR Operation](/web/20220529020840/https://www.baeldung.com/rest-api-query-search-or-operation)
[• REST Query Language with RSQL](/web/20220529020840/https://www.baeldung.com/rest-api-search-language-rsql-fiql)
[• REST Query Language with Querydsl Web Support](/web/20220529020840/https://www.baeldung.com/rest-api-search-querydsl-web-in-spring-data-jpa)

## 1。概述

在这个新系列的第一篇文章[中，我们将探索一种简单的 REST API 查询语言**。我们将充分利用 Spring 的 REST API 和 JPA 2 的持久性标准。**](/web/20220529020840/https://www.baeldung.com/spring-rest-api-query-search-language-tutorial)

**为什么是查询语言？**因为——对于任何足够复杂的 API——通过非常简单的字段搜索/过滤您的资源是远远不够的。查询语言更加灵活，允许您筛选出您需要的资源。

## 2。`User`实体

首先，让我们提出我们将用于过滤/搜索 API 的简单实体——一个基本的`User`:

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
}
```

## 3。使用`CriteriaBuilder`过滤

现在，让我们进入问题的核心——持久层中的查询。

构建查询抽象是一个平衡的问题。一方面，我们需要大量的灵活性，另一方面，我们需要保持复杂性可控。从高层次来看，功能很简单——**传入一些约束，然后返回一些结果**。

让我们看看它是如何工作的:

```
@Repository
public class UserDAO implements IUserDAO {

    @PersistenceContext
    private EntityManager entityManager;

    @Override
    public List<User> searchUser(List<SearchCriteria> params) {
        CriteriaBuilder builder = entityManager.getCriteriaBuilder();
        CriteriaQuery<User> query = builder.createQuery(User.class);
        Root r = query.from(User.class);

        Predicate predicate = builder.conjunction();

        UserSearchQueryCriteriaConsumer searchConsumer = 
          new UserSearchQueryCriteriaConsumer(predicate, builder, r);
        params.stream().forEach(searchConsumer);
        predicate = searchConsumer.getPredicate();
        query.where(predicate);

        List<User> result = entityManager.createQuery(query).getResultList();
        return result;
    }

    @Override
    public void save(User entity) {
        entityManager.persist(entity);
    }
}
```

让我们来看看`UserSearchQueryCriteriaConsumer`类:

```
public class UserSearchQueryCriteriaConsumer implements Consumer<SearchCriteria>{

    private Predicate predicate;
    private CriteriaBuilder builder;
    private Root r;

    @Override
    public void accept(SearchCriteria param) {
        if (param.getOperation().equalsIgnoreCase(">")) {
            predicate = builder.and(predicate, builder
              .greaterThanOrEqualTo(r.get(param.getKey()), param.getValue().toString()));
        } else if (param.getOperation().equalsIgnoreCase("<")) {
            predicate = builder.and(predicate, builder.lessThanOrEqualTo(
              r.get(param.getKey()), param.getValue().toString()));
        } else if (param.getOperation().equalsIgnoreCase(":")) {
            if (r.get(param.getKey()).getJavaType() == String.class) {
                predicate = builder.and(predicate, builder.like(
                  r.get(param.getKey()), "%" + param.getValue() + "%"));
            } else {
                predicate = builder.and(predicate, builder.equal(
                  r.get(param.getKey()), param.getValue()));
            }
        }
    }

    // standard constructor, getter, setter
}
```

如您所见，`searchUser` API 接受一个非常简单的约束列表，基于这些约束编写一个查询，进行搜索并返回结果。

约束类也很简单:

```
public class SearchCriteria {
    private String key;
    private String operation;
    private Object value;
}
```

`SearchCriteria`实现保存了我们的`Query`参数:

*   `key`:用于保存字段名称，例如:`firstName`、`age`、…等。
*   `operation`:用于保存运算——例如:等于、小于、…等。
*   `value`:用于保存字段值，例如:约翰，25，…等。

## 4。测试搜索查询

现在，让我们测试我们的搜索机制，以确保它站得住脚。

首先，让我们通过添加两个用户来初始化数据库以进行测试，如下例所示:

```
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = { PersistenceConfig.class })
@Transactional
@TransactionConfiguration
public class JPACriteriaQueryTest {

    @Autowired
    private IUserDAO userApi;

    private User userJohn;

    private User userTom;

    @Before
    public void init() {
        userJohn = new User();
        userJohn.setFirstName("John");
        userJohn.setLastName("Doe");
        userJohn.setEmail("[[email protected]](/web/20220529020840/https://www.baeldung.com/cdn-cgi/l/email-protection)");
        userJohn.setAge(22);
        userApi.save(userJohn);

        userTom = new User();
        userTom.setFirstName("Tom");
        userTom.setLastName("Doe");
        userTom.setEmail("[[email protected]](/web/20220529020840/https://www.baeldung.com/cdn-cgi/l/email-protection)");
        userTom.setAge(26);
        userApi.save(userTom);
    }
}
```

现在，让我们用特定的`firstName`和`lastName`来得到一个`User`，如下例所示:

```
@Test
public void givenFirstAndLastName_whenGettingListOfUsers_thenCorrect() {
    List<SearchCriteria> params = new ArrayList<SearchCriteria>();
    params.add(new SearchCriteria("firstName", ":", "John"));
    params.add(new SearchCriteria("lastName", ":", "Doe"));

    List<User> results = userApi.searchUser(params);

    assertThat(userJohn, isIn(results));
    assertThat(userTom, not(isIn(results)));
}
```

接下来，让我们用相同的`lastName`得到`User`的`List`:

```
@Test
public void givenLast_whenGettingListOfUsers_thenCorrect() {
    List<SearchCriteria> params = new ArrayList<SearchCriteria>();
    params.add(new SearchCriteria("lastName", ":", "Doe"));

    List<User> results = userApi.searchUser(params);
    assertThat(userJohn, isIn(results));
    assertThat(userTom, isIn(results));
}
```

接下来，让我们获取`age` **大于或等于 25** 的用户:

```
@Test
public void givenLastAndAge_whenGettingListOfUsers_thenCorrect() {
    List<SearchCriteria> params = new ArrayList<SearchCriteria>();
    params.add(new SearchCriteria("lastName", ":", "Doe"));
    params.add(new SearchCriteria("age", ">", "25"));

    List<User> results = userApi.searchUser(params);

    assertThat(userTom, isIn(results));
    assertThat(userJohn, not(isIn(results)));
}
```

接下来，让我们搜索**实际上不存在的用户**:

```
@Test
public void givenWrongFirstAndLast_whenGettingListOfUsers_thenCorrect() {
    List<SearchCriteria> params = new ArrayList<SearchCriteria>();
    params.add(new SearchCriteria("firstName", ":", "Adam"));
    params.add(new SearchCriteria("lastName", ":", "Fox"));

    List<User> results = userApi.searchUser(params);
    assertThat(userJohn, not(isIn(results)));
    assertThat(userTom, not(isIn(results)));
}
```

最后，让我们搜索只给定了**部分**的用户`firstName`:

```
@Test
public void givenPartialFirst_whenGettingListOfUsers_thenCorrect() {
    List<SearchCriteria> params = new ArrayList<SearchCriteria>();
    params.add(new SearchCriteria("firstName", ":", "jo"));

    List<User> results = userApi.searchUser(params);

    assertThat(userJohn, isIn(results));
    assertThat(userTom, not(isIn(results)));
}
```

## 6。`UserController`

最后，现在让我们将对这种灵活搜索的持久性支持连接到 REST API 中。

我们将设置一个简单的`UserController`——带有一个`findAll()` **,使用“`search`”来传递整个搜索/过滤表达式**:

```
@Controller
public class UserController {

    @Autowired
    private IUserDao api;

    @RequestMapping(method = RequestMethod.GET, value = "/users")
    @ResponseBody
    public List<User> findAll(@RequestParam(value = "search", required = false) String search) {
        List<SearchCriteria> params = new ArrayList<SearchCriteria>();
        if (search != null) {
            Pattern pattern = Pattern.compile("(\w+?)(:|<|>)(\w+?),");
            Matcher matcher = pattern.matcher(search + ",");
            while (matcher.find()) {
                params.add(new SearchCriteria(matcher.group(1), 
                  matcher.group(2), matcher.group(3)));
            }
        }
        return api.searchUser(params);
    }
}
```

请注意我们是如何简单地从搜索表达式中创建搜索条件对象的。

我们现在可以开始使用 API，并确保一切正常工作:

```
http://localhost:8080/users?search=lastName:doe,age>25
```

以下是它的回应:

```
[{
    "id":2,
    "firstName":"tom",
    "lastName":"doe",
    "email":"[[email protected]](/web/20220529020840/https://www.baeldung.com/cdn-cgi/l/email-protection)",
    "age":26
}]
```

## 7。结论

这个简单而强大的实现在 REST API 上实现了相当多的智能过滤。是的——它仍然有些粗糙，可以改进(将在下一篇文章中改进)——但是它是在 API 上实现这种过滤功能的坚实起点。

该条的**完整实现**可以在[的 GitHub 项目](https://web.archive.org/web/20220529020840/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-rest-query-language "The Full Example Project on Github")中找到。

Next **»**[REST Query Language with Spring Data JPA Specifications](/web/20220529020840/https://www.baeldung.com/rest-api-search-language-spring-data-specifications)