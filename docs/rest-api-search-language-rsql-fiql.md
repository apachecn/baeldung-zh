# 使用 RSQL 的 REST 查询语言

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/rest-api-search-language-rsql-fiql>

[This article is part of a series:](javascript:void(0);)[• REST Query Language with Spring and JPA Criteria](/web/20220701014319/https://www.baeldung.com/rest-search-language-spring-jpa-criteria)
[• REST Query Language with Spring Data JPA Specifications](/web/20220701014319/https://www.baeldung.com/rest-api-search-language-spring-data-specifications)
[• REST Query Language with Spring Data JPA and Querydsl](/web/20220701014319/https://www.baeldung.com/rest-api-search-language-spring-data-querydsl)
[• REST Query Language – Advanced Search Operations](/web/20220701014319/https://www.baeldung.com/rest-api-query-search-language-more-operations)
[• REST Query Language – Implementing OR Operation](/web/20220701014319/https://www.baeldung.com/rest-api-query-search-or-operation)
• REST Query Language with RSQL (current article)[• REST Query Language with Querydsl Web Support](/web/20220701014319/https://www.baeldung.com/rest-api-search-querydsl-web-in-spring-data-jpa)

## 1。概述

在[系列的第五篇文章](/web/20220701014319/https://www.baeldung.com/spring-rest-api-query-search-language-tutorial)中，我们将展示如何在**的帮助下构建 REST API 查询语言，这是一个很酷的库——[rsql 解析器](https://web.archive.org/web/20220701014319/https://github.com/jirutka/rsql-parser)。**

RSQL 是提要条目查询语言( [FIQL](https://web.archive.org/web/20220701014319/https://tools.ietf.org/html/draft-nottingham-atompub-fiql-00) )的超集——提要的一个干净简单的过滤语法；所以它非常自然地适合 REST API。【T2

## 2。准备工作

首先，让我们向库中添加一个 Maven [依赖项](https://web.archive.org/web/20220701014319/https://search.maven.org/artifact/cz.jirutka.rsql/rsql-parser):

```java
<dependency>
    <groupId>cz.jirutka.rsql</groupId>
    <artifactId>rsql-parser</artifactId>
    <version>2.1.0</version>
</dependency>
```

并且**定义我们将在整个示例中使用的主要实体**—`User`:

```java
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

## 3。解析请求

RSQL 表达式在内部以节点的形式表示，visitor 模式用于解析输入。

记住这一点，我们将实现 [`RSQLVisitor`接口](https://web.archive.org/web/20220701014319/https://github.com/jirutka/rsql-parser/blob/master/src/main/java/cz/jirutka/rsql/parser/ast/RSQLVisitor.java)并创建我们自己的访问者实现—`CustomRsqlVisitor`:

```java
public class CustomRsqlVisitor<T> implements RSQLVisitor<Specification<T>, Void> {

    private GenericRsqlSpecBuilder<T> builder;

    public CustomRsqlVisitor() {
        builder = new GenericRsqlSpecBuilder<T>();
    }

    @Override
    public Specification<T> visit(AndNode node, Void param) {
        return builder.createSpecification(node);
    }

    @Override
    public Specification<T> visit(OrNode node, Void param) {
        return builder.createSpecification(node);
    }

    @Override
    public Specification<T> visit(ComparisonNode node, Void params) {
        return builder.createSecification(node);
    }
}
```

现在我们需要处理持久性，并从这些节点中构造查询。

我们将使用在之前使用的 Spring Data JPA 规范[——我们将实现一个`Specification`构建器来**构建我们访问**的每个节点的规范:](/web/20220701014319/https://www.baeldung.com/rest-api-search-language-spring-data-specifications)

```java
public class GenericRsqlSpecBuilder<T> {

    public Specification<T> createSpecification(Node node) {
        if (node instanceof LogicalNode) {
            return createSpecification((LogicalNode) node);
        }
        if (node instanceof ComparisonNode) {
            return createSpecification((ComparisonNode) node);
        }
        return null;
    }

    public Specification<T> createSpecification(LogicalNode logicalNode) {        
        List<Specification> specs = logicalNode.getChildren()
          .stream()
          .map(node -> createSpecification(node))
          .filter(Objects::nonNull)
          .collect(Collectors.toList());

        Specification<T> result = specs.get(0);
        if (logicalNode.getOperator() == LogicalOperator.AND) {
            for (int i = 1; i < specs.size(); i++) {
                result = Specification.where(result).and(specs.get(i));
            }
        } else if (logicalNode.getOperator() == LogicalOperator.OR) {
            for (int i = 1; i < specs.size(); i++) {
                result = Specification.where(result).or(specs.get(i));
            }
        }

        return result;
    }

    public Specification<T> createSpecification(ComparisonNode comparisonNode) {
        Specification<T> result = Specification.where(
          new GenericRsqlSpecification<T>(
            comparisonNode.getSelector(), 
            comparisonNode.getOperator(), 
            comparisonNode.getArguments()
          )
        );
        return result;
    }
}
```

注意如何:

*   `LogicalNode`是一个**和** `/` **或** `Node`并且有多个子女
*   `ComparisonNode`没有子节点，它保存了**选择器、操作符和参数**

例如，对于查询“`name==john`”–我们有:

1.  **选择器**:“名称”
2.  **操作员** : "== "
3.  **争论**:【约翰】

## 4。`Specification`创建自定义

当构建查询时，我们使用了一个`Specification:`

```java
public class GenericRsqlSpecification<T> implements Specification<T> {

    private String property;
    private ComparisonOperator operator;
    private List<String> arguments;

    @Override
    public Predicate toPredicate(Root<T> root, CriteriaQuery<?> query, CriteriaBuilder builder) {
        List<Object> args = castArguments(root);
        Object argument = args.get(0);
        switch (RsqlSearchOperation.getSimpleOperator(operator)) {

        case EQUAL: {
            if (argument instanceof String) {
                return builder.like(root.get(property), argument.toString().replace('*', '%'));
            } else if (argument == null) {
                return builder.isNull(root.get(property));
            } else {
                return builder.equal(root.get(property), argument);
            }
        }
        case NOT_EQUAL: {
            if (argument instanceof String) {
                return builder.notLike(root.<String> get(property), argument.toString().replace('*', '%'));
            } else if (argument == null) {
                return builder.isNotNull(root.get(property));
            } else {
                return builder.notEqual(root.get(property), argument);
            }
        }
        case GREATER_THAN: {
            return builder.greaterThan(root.<String> get(property), argument.toString());
        }
        case GREATER_THAN_OR_EQUAL: {
            return builder.greaterThanOrEqualTo(root.<String> get(property), argument.toString());
        }
        case LESS_THAN: {
            return builder.lessThan(root.<String> get(property), argument.toString());
        }
        case LESS_THAN_OR_EQUAL: {
            return builder.lessThanOrEqualTo(root.<String> get(property), argument.toString());
        }
        case IN:
            return root.get(property).in(args);
        case NOT_IN:
            return builder.not(root.get(property).in(args));
        }

        return null;
    }

    private List<Object> castArguments(final Root<T> root) {

        Class<? extends Object> type = root.get(property).getJavaType();

        List<Object> args = arguments.stream().map(arg -> {
            if (type.equals(Integer.class)) {
               return Integer.parseInt(arg);
            } else if (type.equals(Long.class)) {
               return Long.parseLong(arg);
            } else {
                return arg;
            }            
        }).collect(Collectors.toList());

        return args;
    }

    // standard constructor, getter, setter
}
```

请注意规范是如何使用泛型的，并且不依赖于任何特定的实体(比如用户)。

接下来——这是我们的**枚举“`RsqlSearchOperation`”**,它包含默认的 rsql 解析器操作符:

```java
public enum RsqlSearchOperation {
    EQUAL(RSQLOperators.EQUAL), 
    NOT_EQUAL(RSQLOperators.NOT_EQUAL), 
    GREATER_THAN(RSQLOperators.GREATER_THAN), 
    GREATER_THAN_OR_EQUAL(RSQLOperators.GREATER_THAN_OR_EQUAL), 
    LESS_THAN(RSQLOperators.LESS_THAN), 
    LESS_THAN_OR_EQUAL(RSQLOperators.LESS_THAN_OR_EQUAL), 
    IN(RSQLOperators.IN), 
    NOT_IN(RSQLOperators.NOT_IN);

    private ComparisonOperator operator;

    private RsqlSearchOperation(ComparisonOperator operator) {
        this.operator = operator;
    }

    public static RsqlSearchOperation getSimpleOperator(ComparisonOperator operator) {
        for (RsqlSearchOperation operation : values()) {
            if (operation.getOperator() == operator) {
                return operation;
            }
        }
        return null;
    }
}
```

## 5。测试搜索查询

现在，让我们通过一些真实场景开始测试我们新的灵活操作:

首先，让我们初始化数据:

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = { PersistenceConfig.class })
@Transactional
@TransactionConfiguration
public class RsqlTest {

    @Autowired
    private UserRepository repository;

    private User userJohn;

    private User userTom;

    @Before
    public void init() {
        userJohn = new User();
        userJohn.setFirstName("john");
        userJohn.setLastName("doe");
        userJohn.setEmail("[[email protected]](/web/20220701014319/https://www.baeldung.com/cdn-cgi/l/email-protection)");
        userJohn.setAge(22);
        repository.save(userJohn);

        userTom = new User();
        userTom.setFirstName("tom");
        userTom.setLastName("doe");
        userTom.setEmail("[[email protected]](/web/20220701014319/https://www.baeldung.com/cdn-cgi/l/email-protection)");
        userTom.setAge(26);
        repository.save(userTom);
    }
}
```

现在让我们测试不同的操作:

### 5.1。测试等式

在下面的例子中，我们将通过用户的`first`和`last name`来搜索用户:

```java
@Test
public void givenFirstAndLastName_whenGettingListOfUsers_thenCorrect() {
    Node rootNode = new RSQLParser().parse("firstName==john;lastName==doe");
    Specification<User> spec = rootNode.accept(new CustomRsqlVisitor<User>());
    List<User> results = repository.findAll(spec);

    assertThat(userJohn, isIn(results));
    assertThat(userTom, not(isIn(results)));
}
```

### 5.2。测试否定

接下来，让我们通过他们的`first name`搜索不是“john”的用户:

```java
@Test
public void givenFirstNameInverse_whenGettingListOfUsers_thenCorrect() {
    Node rootNode = new RSQLParser().parse("firstName!=john");
    Specification<User> spec = rootNode.accept(new CustomRsqlVisitor<User>());
    List<User> results = repository.findAll(spec);

    assertThat(userTom, isIn(results));
    assertThat(userJohn, not(isIn(results)));
}
```

### 5.3。测试大于

接下来，我们将搜索`age`大于`25`的用户:

```java
@Test
public void givenMinAge_whenGettingListOfUsers_thenCorrect() {
    Node rootNode = new RSQLParser().parse("age>25");
    Specification<User> spec = rootNode.accept(new CustomRsqlVisitor<User>());
    List<User> results = repository.findAll(spec);

    assertThat(userTom, isIn(results));
    assertThat(userJohn, not(isIn(results)));
}
```

### 5.4。测试如

接下来，我们将搜索其`first name`以`jo`开头的用户:

```java
@Test
public void givenFirstNamePrefix_whenGettingListOfUsers_thenCorrect() {
    Node rootNode = new RSQLParser().parse("firstName==jo*");
    Specification<User> spec = rootNode.accept(new CustomRsqlVisitor<User>());
    List<User> results = repository.findAll(spec);

    assertThat(userJohn, isIn(results));
    assertThat(userTom, not(isIn(results)));
}
```

### 5.5。测试`IN`

接下来，我们将搜索其`first name`为“`john`”或“`jack`”的用户:

```java
@Test
public void givenListOfFirstName_whenGettingListOfUsers_thenCorrect() {
    Node rootNode = new RSQLParser().parse("firstName=in=(john,jack)");
    Specification<User> spec = rootNode.accept(new CustomRsqlVisitor<User>());
    List<User> results = repository.findAll(spec);

    assertThat(userJohn, isIn(results));
    assertThat(userTom, not(isIn(results)));
}
```

## `**6\. UserController**`

最后，让我们将这一切与控制器联系起来:

```java
@RequestMapping(method = RequestMethod.GET, value = "/users")
@ResponseBody
public List<User> findAllByRsql(@RequestParam(value = "search") String search) {
    Node rootNode = new RSQLParser().parse(search);
    Specification<User> spec = rootNode.accept(new CustomRsqlVisitor<User>());
    return dao.findAll(spec);
}
```

以下是一个示例 URL:

```java
http://localhost:8080/users?search=firstName==jo*;age<25
```

回应是:

```java
[{
    "id":1,
    "firstName":"john",
    "lastName":"doe",
    "email":"[[email protected]](/web/20220701014319/https://www.baeldung.com/cdn-cgi/l/email-protection)",
    "age":24
}]
```

## 7 .**。结论**

本教程演示了如何为 REST API 构建查询/搜索语言，而不必重新发明语法，而是使用 FIQL / RSQL。

本文的**完整实现**可以在[的 GitHub 项目](https://web.archive.org/web/20220701014319/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-rest-query-language "The Full Example Project on Github")中找到——这是一个基于 Maven 的项目，因此应该很容易导入和运行。

Next **»**[REST Query Language with Querydsl Web Support](/web/20220701014319/https://www.baeldung.com/rest-api-search-querydsl-web-in-spring-data-jpa)**«** Previous[REST Query Language – Implementing OR Operation](/web/20220701014319/https://www.baeldung.com/rest-api-query-search-or-operation)