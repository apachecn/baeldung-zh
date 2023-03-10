# REST 查询语言–实现 OR 操作

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/rest-api-query-search-or-operation>

[This article is part of a series:](javascript:void(0);)[• REST Query Language with Spring and JPA Criteria](/web/20220628101111/https://www.baeldung.com/rest-search-language-spring-jpa-criteria)
[• REST Query Language with Spring Data JPA Specifications](/web/20220628101111/https://www.baeldung.com/rest-api-search-language-spring-data-specifications)
[• REST Query Language with Spring Data JPA and Querydsl](/web/20220628101111/https://www.baeldung.com/rest-api-search-language-spring-data-querydsl)
[• REST Query Language – Advanced Search Operations](/web/20220628101111/https://www.baeldung.com/rest-api-query-search-language-more-operations)
• REST Query Language – Implementing OR Operation (current article)[• REST Query Language with RSQL](/web/20220628101111/https://www.baeldung.com/rest-api-search-language-rsql-fiql)
[• REST Query Language with Querydsl Web Support](/web/20220628101111/https://www.baeldung.com/rest-api-search-querydsl-web-in-spring-data-jpa)

## 1。概述

在这篇简短的文章中，我们将扩展我们在[上一篇文章](/web/20220628101111/https://www.baeldung.com/rest-api-query-search-language-more-operations)中实现的高级搜索操作，并将**或基于 OR 的搜索标准包含到我们的 REST API 查询语言**中。

## 2。实施方法

以前，`search`查询参数中的所有条件都是由 AND 操作符组成的谓词。让我们改变这一点。

我们应该能够实现这个特性，或者作为对现有方法的一个简单、快速的改变，或者从头开始一个新的方法。

对于简单的方法，我们将标记标准，以表明它必须使用 OR 运算符进行组合。

例如，下面是测试“**名或姓”的 API 的 URL:**

```java
http://localhost:8080/users?search=firstName:john,'lastName:doe
```

请注意，我们用单引号标记了标准`lastName`,以示区别。我们将在我们的标准值对象–`SpecSearchCriteria:`中捕获 or 操作符的谓词

```java
public SpecSearchCriteria(
  String orPredicate, String key, SearchOperation operation, Object value) {
    super();

    this.orPredicate 
      = orPredicate != null
      && orPredicate.equals(SearchOperation.OR_PREDICATE_FLAG);

    this.key = key;
    this.operation = operation;
    this.value = value;
}
```

## 3。`UserSpecificationBuilder`改进

现在，让我们修改我们的规范构建器`UserSpecificationBuilder,` ，以便在构建`Specification<User>`时考虑 OR 限定标准:

```java
public Specification<User> build() {
    if (params.size() == 0) {
        return null;
    }
    Specification<User> result = new UserSpecification(params.get(0));

    for (int i = 1; i < params.size(); i++) {
        result = params.get(i).isOrPredicate()
          ? Specification.where(result).or(new UserSpecification(params.get(i))) 
          : Specification.where(result).and(new UserSpecification(params.get(i)));
    }
    return result;
 }
```

## 4。`UserController`改进

最后，让我们在控制器中设置一个新的 REST 端点，通过 OR 操作符使用这个搜索功能。改进的解析逻辑提取特殊标志，该标志有助于用 OR 运算符标识标准:

```java
@GetMapping("/users/espec")
@ResponseBody
public List<User> findAllByOrPredicate(@RequestParam String search) {
    Specification<User> spec = resolveSpecification(search);
    return dao.findAll(spec);
}

protected Specification<User> resolveSpecification(String searchParameters) {
    UserSpecificationsBuilder builder = new UserSpecificationsBuilder();
    String operationSetExper = Joiner.on("|")
      .join(SearchOperation.SIMPLE_OPERATION_SET);
    Pattern pattern = Pattern.compile(
      "(\\p{Punct}?)(\\w+?)("
      + operationSetExper 
      + ")(\\p{Punct}?)(\\w+?)(\\p{Punct}?),");
    Matcher matcher = pattern.matcher(searchParameters + ",");
    while (matcher.find()) {
        builder.with(matcher.group(1), matcher.group(2), matcher.group(3), 
        matcher.group(5), matcher.group(4), matcher.group(6));
    }

    return builder.build();
}
```

## 5。带`OR`条件的带电测试

在这个现场测试示例中，使用新的 API 端点，我们将通过名字“john”或姓氏“doe”来搜索用户。注意，参数`lastName`有一个单引号，将其限定为“或”谓词:

```java
private String EURL_PREFIX
  = "http://localhost:8082/spring-rest-full/auth/users/espec?search=";

@Test
public void givenFirstOrLastName_whenGettingListOfUsers_thenCorrect() {
    Response response = givenAuth().get(EURL_PREFIX + "firstName:john,'lastName:doe");
    String result = response.body().asString();

    assertTrue(result.contains(userJohn.getEmail()));
    assertTrue(result.contains(userTom.getEmail()));
}
```

## 6。使用`OR`条件的持久性测试

现在，让我们对名为“john”或姓为“doe”的用户**在持久性级别执行与上面相同的测试:**

```java
@Test
public void givenFirstOrLastName_whenGettingListOfUsers_thenCorrect() {
    UserSpecificationsBuilder builder = new UserSpecificationsBuilder();

    SpecSearchCriteria spec 
      = new SpecSearchCriteria("firstName", SearchOperation.EQUALITY, "john");
    SpecSearchCriteria spec1 
      = new SpecSearchCriteria("'","lastName", SearchOperation.EQUALITY, "doe");

    List<User> results = repository
      .findAll(builder.with(spec).with(spec1).build());

    assertThat(results, hasSize(2));
    assertThat(userJohn, isIn(results));
    assertThat(userTom, isIn(results));
}
```

## 7 .**。替代方法**

在另一种方法中，我们可以提供更像 SQL 查询的完整`WHERE`子句的搜索查询。

例如，下面是通过`firstName`和`age:`进行更复杂搜索的 URL

```java
http://localhost:8080/users?search=( firstName:john OR firstName:tom ) AND age>22
```

注意，我们用空格分隔了单个标准、操作符和分组括号，以形成有效的中缀表达式。

让我们用一个`CriteriaParser`来解析中缀表达式。我们的`CriteriaParser` 将给定的中缀表达式分割成记号(标准、括号和&或操作符),并为其创建一个后缀表达式:

```java
public Deque<?> parse(String searchParam) {

    Deque<Object> output = new LinkedList<>();
    Deque<String> stack = new LinkedList<>();

    Arrays.stream(searchParam.split("\\s+")).forEach(token -> {
        if (ops.containsKey(token)) {
            while (!stack.isEmpty() && isHigerPrecedenceOperator(token, stack.peek())) {
                output.push(stack.pop().equalsIgnoreCase(SearchOperation.OR_OPERATOR)
                  ? SearchOperation.OR_OPERATOR : SearchOperation.AND_OPERATOR);
            }
            stack.push(token.equalsIgnoreCase(SearchOperation.OR_OPERATOR) 
              ? SearchOperation.OR_OPERATOR : SearchOperation.AND_OPERATOR);

        } else if (token.equals(SearchOperation.LEFT_PARANTHESIS)) {
            stack.push(SearchOperation.LEFT_PARANTHESIS);
        } else if (token.equals(SearchOperation.RIGHT_PARANTHESIS)) {
            while (!stack.peek().equals(SearchOperation.LEFT_PARANTHESIS)) { 
                output.push(stack.pop());
            }
            stack.pop();
        } else {
            Matcher matcher = SpecCriteraRegex.matcher(token);
            while (matcher.find()) {
                output.push(new SpecSearchCriteria(
                  matcher.group(1), 
                  matcher.group(2), 
                  matcher.group(3), 
                  matcher.group(4), 
                  matcher.group(5)));
            }
        }
    });

    while (!stack.isEmpty()) {
        output.push(stack.pop());
    }

    return output;
}
```

让我们在我们的规范构建器中添加一个新方法，`GenericSpecificationBuilder,`来从后缀表达式中构造搜索`Specification` :

```java
 public Specification<U> build(Deque<?> postFixedExprStack, 
        Function<SpecSearchCriteria, Specification<U>> converter) {

        Deque<Specification<U>> specStack = new LinkedList<>();

        while (!postFixedExprStack.isEmpty()) {
            Object mayBeOperand = postFixedExprStack.pollLast();

            if (!(mayBeOperand instanceof String)) {
                specStack.push(converter.apply((SpecSearchCriteria) mayBeOperand));
            } else {
                Specification<U> operand1 = specStack.pop();
                Specification<U> operand2 = specStack.pop();
                if (mayBeOperand.equals(SearchOperation.AND_OPERATOR)) {
                    specStack.push(Specification.where(operand1)
                      .and(operand2));
                }
                else if (mayBeOperand.equals(SearchOperation.OR_OPERATOR)) {
                    specStack.push(Specification.where(operand1)
                      .or(operand2));
                }
            }
        }
        return specStack.pop();
```

最后，让我们在我们的`UserController`中添加另一个 REST 端点，用新的`CriteriaParser`解析复杂表达式:

```java
@GetMapping("/users/spec/adv")
@ResponseBody
public List<User> findAllByAdvPredicate(@RequestParam String search) {
    Specification<User> spec = resolveSpecificationFromInfixExpr(search);
    return dao.findAll(spec);
}

protected Specification<User> resolveSpecificationFromInfixExpr(String searchParameters) {
    CriteriaParser parser = new CriteriaParser();
    GenericSpecificationsBuilder<User> specBuilder = new GenericSpecificationsBuilder<>();
    return specBuilder.build(parser.parse(searchParameters), UserSpecification::new);
}
```

## 8。结论

在本教程中，我们改进了 REST 查询语言，增加了使用 OR 操作符进行搜索的能力。

本文的完整实现可以在 GitHub 项目中找到。这是一个基于 Maven 的项目，因此应该很容易导入和运行。

Next **»**[REST Query Language with RSQL](/web/20220628101111/https://www.baeldung.com/rest-api-search-language-rsql-fiql)**«** Previous[REST Query Language – Advanced Search Operations](/web/20220628101111/https://www.baeldung.com/rest-api-query-search-language-more-operations)