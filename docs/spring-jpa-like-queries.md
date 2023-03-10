# 比如 Spring JPA 存储库中的查询

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-jpa-like-queries>

## 1.概观

在这个快速教程中，我们将介绍在 [Spring JPA 库](/web/20220707143830/https://www.baeldung.com/the-persistence-layer-with-spring-data-jpa)中创建 LIKE 查询的不同方法。

我们将从创建查询方法时可以使用的各种关键字开始。然后，我们将用命名的和有序的参数覆盖`@Query`注释。

## 2.设置

对于我们的例子，我们将查询一个`movie`表。

让我们定义我们的`Movie`实体:

```java
@Entity
public class Movie {
    @Id
    @GeneratedValue(strategy = GenerationType.SEQUENCE)
    private Long id;
    private String title;
    private String director;
    private String rating;
    private int duration;

    // standard getters and setters
}
```

定义了我们的`Movie`实体后，让我们创建一些样本 insert 语句:

```java
INSERT INTO movie(id, title, director, rating, duration) 
    VALUES(1, 'Godzilla: King of the Monsters', ' Michael Dougherty', 'PG-13', 132);
INSERT INTO movie(id, title, director, rating, duration) 
    VALUES(2, 'Avengers: Endgame', 'Anthony Russo', 'PG-13', 181);
INSERT INTO movie(id, title, director, rating, duration) 
    VALUES(3, 'Captain Marvel', 'Anna Boden', 'PG-13', 123);
INSERT INTO movie(id, title, director, rating, duration) 
    VALUES(4, 'Dumbo', 'Tim Burton', 'PG', 112);
INSERT INTO movie(id, title, director, rating, duration) 
    VALUES(5, 'Booksmart', 'Olivia Wilde', 'R', 102);
INSERT INTO movie(id, title, director, rating, duration) 
    VALUES(6, 'Aladdin', 'Guy Ritchie', 'PG', 128);
INSERT INTO movie(id, title, director, rating, duration) 
    VALUES(7, 'The Sun Is Also a Star', 'Ry Russo-Young', 'PG-13', 100);
```

## 3.比如查询方法

对于许多简单的查询场景，我们可以利用各种关键字在我们的存储库中创建查询方法。

现在就让我们来探索一下。

### 3.1.`Containing`、`Contains`、`IsContaining`和`Like`

让我们看看如何使用查询方法执行以下 LIKE 查询:

```java
SELECT * FROM movie WHERE title LIKE '%in%';
```

首先，让我们使用`Containing`、`Contains` 和`IsContaining`来定义查询方法:

```java
List<Movie> findByTitleContaining(String title);
List<Movie> findByTitleContains(String title);
List<Movie> findByTitleIsContaining(String title);
```

让我们用部分标题`in`来调用我们的查询方法:

```java
List<Movie> results = movieRepository.findByTitleContaining("in");
assertEquals(3, results.size());

results = movieRepository.findByTitleIsContaining("in");
assertEquals(3, results.size());

results = movieRepository.findByTitleContains("in");
assertEquals(3, results.size());
```

我们可以预期这三种方法都会返回相同的结果。

**Spring 也为我们提供了一个`Like`关键字，但是它的行为略有不同，因为我们需要为搜索参数提供通配符。**

让我们定义一个 LIKE 查询方法:

```java
List<Movie> findByTitleLike(String title);
```

现在我们将使用之前使用的相同值调用我们的`findByTitleLike`方法，但是包括通配符:

```java
results = movieRepository.findByTitleLike("%in%");
assertEquals(3, results.size());
```

### 3.2.`StartsWith`

让我们看看下面的查询:

```java
SELECT * FROM Movie WHERE Rating LIKE 'PG%';
```

我们将使用`StartsWith`关键字来创建一个查询方法:

```java
List<Movie> findByRatingStartsWith(String rating);
```

定义了我们的方法后，让我们用值`PG`来调用它:

```java
List<Movie> results = movieRepository.findByRatingStartsWith("PG");
assertEquals(6, results.size());
```

### 3.3.`EndsWith`

**Spring 为我们提供了与`EndsWith`关键字相反的功能。**

让我们考虑这个查询:

```java
SELECT * FROM Movie WHERE director LIKE '%Burton';
```

现在我们将定义一个`EndsWith`查询方法:

```java
List<Movie> findByDirectorEndsWith(String director);
```

一旦我们定义了我们的方法，让我们用`Burton` 参数调用它:

```java
List<Movie> results = movieRepository.findByDirectorEndsWith("Burton");
assertEquals(1, results.size());
```

### 3.4.不区分大小写

我们经常希望找到包含某个字符串的所有记录，而不考虑大小写。在 SQL 中，我们可以通过强制该列全部为大写或小写字母并为我们查询的值提供相同的值来实现这一点。

**对于 Spring JPA，我们可以使用 [`IgnoreCase`](/web/20220707143830/https://www.baeldung.com/spring-data-case-insensitive-queries) 关键字与我们的另一个关键字**:

```java
List<Movie> findByTitleContainingIgnoreCase(String title);
```

现在我们可以用`the`调用该方法，并期望得到包含小写和大写结果的结果:

```java
List<Movie> results = movieRepository.findByTitleContainingIgnoreCase("the");
assertEquals(2, results.size());
```

### 3.5.`Not`

有时我们希望找到所有不包含特定字符串的记录。**我们可以使用`NotContains`、`NotContaining` 和`NotLike` 关键字来实现。**

让我们定义一个使用`NotContaining`的查询来查找评级不包含`PG`的电影:

```java
List<Movie> findByRatingNotContaining(String rating);
```

现在让我们调用我们新定义的方法:

```java
List<Movie> results = movieRepository.findByRatingNotContaining("PG");
assertEquals(1, results.size());
```

为了实现查找导演姓名不以特定字符串开头的记录的功能，我们将使用`NotLike`关键字来保持对通配符位置的控制:

```java
List<Movie> findByDirectorNotLike(String director);
```

最后，让我们调用方法来查找导演姓名以`An`之外的其他字母开头的所有电影:

```java
List<Movie> results = movieRepository.findByDirectorNotLike("An%");
assertEquals(5, results.size());
```

我们可以以类似的方式使用`NotLike`来实现与`EndsWith`相结合的`Not` 功能。

## 4.使用`@Query`

有时，我们需要创建对于查询方法来说过于复杂的查询，或者会产生长得离谱的方法名。在那些情况下，**我们可以使用 [`@Query`注释](/web/20220707143830/https://www.baeldung.com/spring-data-jpa-query)来查询我们的数据库。**

### 4.1.命名参数

为了进行比较，我们将创建一个与我们之前定义的`findByTitleContaining`方法等效的查询:

```java
@Query("SELECT m FROM Movie m WHERE m.title LIKE %:title%")
List<Movie> searchByTitleLike(@Param("title") String title);
```

我们在提供的查询中包含通配符。`@Param`注释在这里很重要，因为我们使用了一个命名参数。

### 4.2.有序参数

除了命名参数，我们还可以在查询中使用有序参数:

```java
@Query("SELECT m FROM Movie m WHERE m.rating LIKE ?1%")
List<Movie> searchByRatingStartsWith(String rating);
```

我们可以控制我们的通配符，所以这个查询相当于`findByRatingStartsWith`查询方法。

让我们找到所有评分以`PG`开头的电影:

```java
List<Movie> results = movieRepository.searchByRatingStartsWith("PG");
assertEquals(6, results.size());
```

当我们在带有不可信数据的 LIKE 查询中使用有序参数时，我们应该对传入的搜索值进行转义。

如果我们使用 Spring Boot 2.4.1 或更高版本，我们可以使用[SpEL](/web/20220707143830/https://www.baeldung.com/spring-expression-language)方法:

```java
@Query("SELECT m FROM Movie m WHERE m.director LIKE %?#{escape([0])} escape ?#{escapeCharacter()}")
List<Movie> searchByDirectorEndsWith(String director);
```

现在让我们用值`Burton`调用我们的方法:

```java
List<Movie> results = movieRepository.searchByDirectorEndsWith("Burton");
assertEquals(1, results.size());
```

## 5.结论

在这篇短文中，我们学习了如何在 Spring JPA 存储库中创建 LIKE 查询。

首先，我们学习了如何使用提供的关键字来创建查询方法。

然后我们学习了如何使用带有命名参数和有序参数的`@Query`参数来完成相同的任务。

完整的示例代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220707143830/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-data-jpa-repo-2)