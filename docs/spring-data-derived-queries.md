# Spring 数据 JPA 存储库中的派生查询方法

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-data-derived-queries>

## 1.概观

对于简单的查询，只需查看代码中相应的方法名，就可以很容易地推导出查询应该是什么。

在本教程中，我们将探索 [Spring Data JPA](/web/20221109154644/https://www.baeldung.com/the-persistence-layer-with-spring-data-jpa#customquery) 如何以方法命名约定的形式利用这一思想。

## 延伸阅读:

## [Spring Data JPA 简介](/web/20221109154644/https://www.baeldung.com/the-persistence-layer-with-spring-data-jpa)

Introduction to Spring Data JPA with Spring 4 - the Spring config, the DAO, manual and generated queries and transaction management.[Read more](/web/20221109154644/https://www.baeldung.com/the-persistence-layer-with-spring-data-jpa) →

## [Spring 数据中的 CrudRepository、JpaRepository 和 paging and sorting repository](/web/20221109154644/https://www.baeldung.com/spring-data-repositories)

Learn about the different flavours of repositories offered by Spring Data.[Read more](/web/20221109154644/https://www.baeldung.com/spring-data-repositories) →

## [用 Spring 数据对查询结果进行排序](/web/20221109154644/https://www.baeldung.com/spring-data-sorting)

Learn different ways to sort results in Spring Data queries.[Read more](/web/20221109154644/https://www.baeldung.com/spring-data-sorting) →

## 2.Spring 中派生查询方法的结构

**派生方法名有两个主要部分，由第一个`By` 关键字**分隔:

```java
List<User> findByName(String name)
```

第一部分——如`find`——是`introducer`，其余部分——如`ByName`——是`criteria`。

**Spring Data JPA 支持`find`、`read`、`query`、`count`、`get`。**所以，我们可以做`queryByName`，春天的数据会有同样的表现。

我们还可以使用`Distinct`、`First`或`Top`来删除重复项或[来限制我们的结果集](/web/20221109154644/https://www.baeldung.com/jpa-limit-query-results#spring-data-jpa):

```java
List<User> findTop3ByAge()
```

标准部分包含查询的实体特定条件表达式。我们可以使用条件关键字和实体的属性名。

我们还可以用`And`和`Or`连接表达式，我们马上就会看到。

## 3.示例应用程序

首先，我们当然需要使用 Spring 数据 JPA 的应用程序。

在该应用程序中，让我们定义一个实体类:

```java
@Table(name = "users")
@Entity
class User {
    @Id
    @GeneratedValue
    private Integer id;

    private String name;
    private Integer age;
    private ZonedDateTime birthDate;
    private Boolean active;

    // standard getters and setters
}
```

让我们也定义一个存储库。

它将扩展`JpaRepository`，Spring 数据仓库类型的[之一:](/web/20221109154644/https://www.baeldung.com/spring-data-repositories)

```java
interface UserRepository extends JpaRepository<User, Integer> {}
```

这是我们放置所有派生查询方法的地方。

## 4.相等条件关键字

完全相等是查询中最常用的条件之一。我们有几个选项来表示查询中的`=`或 IS 操作符。

对于精确匹配条件，我们可以只添加属性名，而不添加任何关键字:

```java
List<User> findByName(String name);
```

为了可读性，我们可以添加`Is`或`Equals`:

```java
List<User> findByNameIs(String name);
List<User> findByNameEquals(String name);
```

当我们需要表达不平等时，这种额外的可读性就派上了用场:

```java
List<User> findByNameIsNot(String name);
```

这比`findByNameNot(String)`可读性强多了！

由于等式是一个特例，我们不应该使用=运算符。默认情况下，Spring Data JPA 处理 [`null`参数](/web/20221109154644/https://www.baeldung.com/spring-data-jpa-null-parameters)。因此，当我们为等式条件传递一个`null`值时，Spring 在生成的 SQL 中将查询解释为 NULL。

我们还可以使用`IsNull`关键字将 IS NULL 标准添加到查询中:

```java
List<User> findByNameIsNull();
List<User> findByNameIsNotNull();
```

注意`IsNull` 和`IsNotNull` 都不需要方法参数。

还有两个不需要任何参数的关键字。

我们可以使用`True`和`False` 关键字为`boolean`类型添加等式条件:

```java
List<User> findByActiveTrue();
List<User> findByActiveFalse();
```

当然，我们有时想要比完全平等更宽松的东西，所以让我们看看我们还能做些什么。

## 5.相似条件关键字

当我们需要用属性的模式查询结果时，我们有几个选项。

我们可以使用`StartingWith`找到以某个值开头的名称:

```java
List<User> findByNameStartingWith(String prefix);
```

粗略地说，这可以翻译成“WHERE `name` LIKE `‘value%'`”。

如果我们想要以一个值结尾的名字，`EndingWith`就是我们想要的:

```java
List<User> findByNameEndingWith(String suffix);
```

或者我们可以找到哪些名称包含带有`Containing`的值:

```java
List<User> findByNameContaining(String infix);
```

请注意，上述所有条件都称为预定义模式表达式。所以，**我们在调用这些方法的时候，不需要在参数**里面添加`% `运算符。

但是让我们假设我们正在做一些更复杂的事情。假设我们需要获取名称以`a`开头、包含`b`并以`c`结尾的用户。

为此，我们可以用关键字`Like` 添加我们自己的 LIKE:

```java
List<User> findByNameLike(String likePattern);
```

当我们调用这个方法时，我们可以传递我们喜欢的模式:

```java
String likePattern = "a%b%c";
userRepository.findByNameLike(likePattern);
```

名字到此为止。让我们试试`User`中的其他值。

## 6.比较条件关键字

此外，我们可以使用`LessThan`和`LessThanEqual` 关键字，通过`<` 和`<=` 操作符将记录与给定值进行比较:

```java
List<User> findByAgeLessThan(Integer age);
List<User> findByAgeLessThanEqual(Integer age);
```

在相反的情况下，我们可以使用`GreaterThan`和`GreaterThanEqual` 关键字:

```java
List<User> findByAgeGreaterThan(Integer age);
List<User> findByAgeGreaterThanEqual(Integer age);
```

或者我们可以用`Between`找到两个年龄之间的用户:

```java
List<User> findByAgeBetween(Integer startAge, Integer endAge);
```

我们还可以提供一个年龄集合来匹配使用`In`:

```java
List<User> findByAgeIn(Collection<Integer> ages);
```

因为我们知道用户的出生日期，所以我们可能希望查询在给定日期之前或之后出生的用户。

我们用`Before`和`After`来表示:

```java
List<User> findByBirthDateAfter(ZonedDateTime birthDate);
List<User> findByBirthDateBefore(ZonedDateTime birthDate);
```

## 7.多重条件表达式

我们可以使用`And`和`Or` 关键字组合任意多的表达式:

```java
List<User> findByNameOrBirthDate(String name, ZonedDateTime birthDate);
List<User> findByNameOrBirthDateAndActive(String name, ZonedDateTime birthDate, Boolean active);
```

优先顺序是`And` 然后`Or`，就像 Java 一样。

虽然 Spring Data JPA 没有限制我们可以添加多少个表达式，但我们不应该在这里发疯。长名字不可读且难以维护。对于复杂的查询，看一下**的`[@Query](/web/20221109154644/https://www.baeldung.com/spring-data-jpa-query)`注释。**

## 8.对结果进行排序

接下来，我们来看看排序。

我们可以使用`OrderBy`让用户按照名字的字母顺序排序:

```java
List<User> findByNameOrderByName(String name);
List<User> findByNameOrderByNameAsc(String name);
```

升序是默认的排序选项，但是我们可以使用`Desc` 来反向排序:

```java
List<User> findByNameOrderByNameDesc(String name);
```

## 9.`findOne` vs `findById`在一场`CrudRepository`

春队用 Spring Boot 2.x 对 [`CrudRepository`](/web/20221109154644/https://www.baeldung.com/spring-data-repositories#crudrepository) 做了一些重大改动，其中一个就是把`findOne`改名为`findById`。

以前在 Spring Boot 1.x 中，当我们想通过主键检索一个实体时，我们会调用`findOne`:

```java
User user = userRepository.findOne(1);
```

从 Spring Boot 2.x 开始，我们可以用`findById`做同样的事情:

```java
User user = userRepository.findById(1);
```

注意，`findById() `方法已经在`CrudRepository` 中为我们定义了。所以，我们不需要在扩展`CrudRepository`的定制库中明确定义它。

## 10.结论

在本文中，我们解释了 Spring Data JPA 中的查询派生机制。我们使用属性条件关键字在 Spring Data JPA 存储库中编写派生的查询方法。

这篇文章的源代码可以在[GitHub 项目](https://web.archive.org/web/20221109154644/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-data-jpa-repo)中找到。