# JDBC 春天数据简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-data-jdbc-intro>

## 1.概观

Spring Data JDBC 是一个持久性框架，不像 Spring Data JPA 那样复杂。它不提供缓存、延迟加载、后写或 JPA 的许多其他特性。尽管如此，它有自己的 ORM，并提供了我们在 Spring Data JPA **中使用的大多数功能，如映射实体、存储库、查询注释和`JdbcTemplate`** 。

需要记住的一件重要事情是 **Spring Data JDBC 不提供模式生成**。因此，我们负责显式地创建模式。

## 2.将 Spring 数据 JDBC 添加到项目中

通过 JDBC 依赖启动器，Spring Boot 应用程序可以使用 Spring Data JDBC。**这个依赖启动器没有带数据库驱动，虽然**。这个决定必须由开发商做出。让我们添加 Spring 数据 JPA 的依赖启动器:

```
<dependency> 
    <groupId>org.springframework.boot</groupId> 
    <artifactId>spring-boot-starter-data-jdbc</artifactId>
</dependency> 
```

在这个例子中，我们使用的是 H2 数据库。正如我们前面提到的，Spring Data JDBC 不提供模式生成。在这种情况下，我们可以创建一个定制的`schema.sql`文件，该文件将包含用于创建模式对象的 SQL DDL 命令。自动地，Spring Boot 将选择这个文件并使用它来创建数据库对象。

## 3.添加实体

和其他 Spring 数据项目一样，我们使用注释来映射 POJOs 和数据库表。在 **Spring 数据 JDBC 中，实体需要有一个`@Id`。** Spring 数据 JDBC 使用`@Id`注释来标识实体。

与 Spring Data JPA 类似，Spring Data JDBC 默认使用一种命名策略，将 Java 实体映射到关系数据库表，将属性映射到列名。默认情况下，实体和属性的大小写名称分别映射到表和列的大小写名称。例如，名为`AddressBook`的 Java 实体被映射到名为`address_book`的数据库表。

此外，我们可以通过使用`@Table`和`@Column`注释将实体和属性显式地映射到表和列。例如，下面我们定义了将在本例中使用的实体:

```
public class Person {
    @Id
    private long id;
    private String firstName;
    private String lastName;
    // constructors, getters, setters
}
```

我们不需要在`Person`类中使用注释`@Table` 或`@Column`。Spring 数据 JDBC 的默认命名策略隐式地执行实体和表之间的所有映射。

## 4.申报 JDBC 储存库

Spring Data JDBC 使用类似于 Spring Data JPA 的语法。我们可以通过扩展`Repository`、`CrudRepository, or PagingAndSortingRepository`接口来创建一个 Spring 数据 JDBC 存储库。通过实现`CrudRepository`，我们获得了最常用方法的实现，比如`save`、`delete`和`findById`等等。

让我们创建一个我们将在示例中使用的 JDBC 存储库:

```
@Repository 
public interface PersonRepository extends CrudRepository<Person, Long> {
}
```

如果我们需要分页和排序功能，最好的选择是扩展`PagingAndSortingRepository` 接口。

## 5.定制 JDBC 存储库

尽管`CrudRepository`有内置的方法，我们需要为特定的情况创建我们的方法。

现在，让我们用一个非修改查询和一个修改查询来定制我们的`PersonRepository`:

```
@Repository
public interface PersonRepository extends CrudRepository<Person, Long> {

    List<Person> findByFirstName(String firstName);

    @Modifying
    @Query("UPDATE person SET first_name = :name WHERE id = :id")
    boolean updateByFirstName(@Param("id") Long id, @Param("name") String name);
}
```

从 2.0 版本开始，Spring 数据 JDBC 支持[查询方式](https://web.archive.org/web/20220926200248/https://docs.spring.io/spring-data/jdbc/docs/current/reference/html/#jdbc.query-methods)。也就是说，如果我们将我们的查询方法命名为包含关键字，例如，`findByFirstName,` Spring Data JDBC 将自动生成查询对象。

然而，对于修改查询，我们使用`@Modifying`注释来注释修改实体的查询方法。同样，我们用`@Query`注释来修饰它。

在`@Query`注释中，我们添加了 SQL 命令。在 Spring 数据 JDBC 中，我们用普通的 SQL 编写查询。我们不使用任何像 JPQL 这样的高级查询语言。结果，应用程序变得与数据库供应商紧密耦合。

因此，更改到不同的数据库也变得更加困难。

我们需要记住的一件事是， **Spring Data JDBC 不支持引用索引号为**的参数。**我们只能通过名字**来引用参数。

## 6.填充数据库

最后，我们需要用数据填充数据库，这些数据将用于测试我们上面创建的 Spring 数据 JDBC 存储库。因此，我们将创建一个将插入虚拟数据的数据库种子。让我们为这个示例添加数据库播种器的实现:

```
@Component
public class DatabaseSeeder {

    @Autowired
    private JdbcTemplate jdbcTemplate;
    public void insertData() {
        jdbcTemplate.execute("INSERT INTO Person(first_name,last_name) VALUES('Victor', 'Hugo')");
        jdbcTemplate.execute("INSERT INTO Person(first_name,last_name) VALUES('Dante', 'Alighieri')");
        jdbcTemplate.execute("INSERT INTO Person(first_name,last_name) VALUES('Stefan', 'Zweig')");
        jdbcTemplate.execute("INSERT INTO Person(first_name,last_name) VALUES('Oscar', 'Wilde')");
    }
}
```

如上所示，我们使用 Spring JDBC 来执行`INSERT`语句。特别是，Spring JDBC 处理与数据库的连接，并让我们使用`JdbcTemplate` s 执行 SQL 命令。这个解决方案非常灵活，因为我们可以完全控制执行的查询。

## 7.结论

总而言之，Spring Data JDBC 提供了一个像使用 Spring JDBC 一样简单的解决方案——它背后没有魔法。尽管如此，它也提供了我们习惯使用 Spring Data JPA 的大部分特性。

与 Spring Data JPA 相比，Spring Data JDBC 的最大优势之一是在访问数据库时提高了性能。这是因为 Spring 数据 JDBC **直接与数据库**通信。 **Spring Data JDBC 在查询数据库时并没有包含大部分 Spring Data magic】。**

使用 Spring 数据 JDBC 的最大缺点之一是对数据库供应商的依赖。如果我们决定将数据库从 MySQL 改为 Oracle，**我们可能不得不处理来自不同方言的数据库的问题**。

这个 Spring 数据 JDBC 教程的实现可以在 GitHub 上找到[。](https://web.archive.org/web/20220926200248/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-data-jdbc)