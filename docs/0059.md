# 使用 Spring Boot 加载初始数据快速指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-data-sql-and-schema-sql>

## 1。概述

Spring Boot 使管理我们的数据库变更变得非常容易。如果我们保留默认配置，它将在我们的包中搜索实体并自动创建各自的表。

但是我们有时需要对数据库变更进行更细粒度的控制。这时候我们就可以在 Spring 中使用`data.sql`和`schema.sql`文件了。

## 延伸阅读:

## [Spring Boot 与 H2 数据库](/web/20220707143831/https://www.baeldung.com/spring-boot-h2-database)

Learn how to configure and how to use the H2 database with Spring Boot.[Read more](/web/20220707143831/https://www.baeldung.com/spring-boot-h2-database) →

## [使用 Flyway 进行数据库迁移](/web/20220707143831/https://www.baeldung.com/database-migrations-with-flyway)

This article describes key concepts of Flyway and how we can use this framework to continuously remodel our application's database schema reliably and easily.[Read more](/web/20220707143831/https://www.baeldung.com/database-migrations-with-flyway) →

## [用 Spring 数据 JPA 生成数据库模式](/web/20220707143831/https://www.baeldung.com/spring-data-jpa-generate-db-schema)

JPA provides a standard for generating DDL from our entity model. Here we explore how to do this in Spring Data and compare that with native Hibernate.[Read more](/web/20220707143831/https://www.baeldung.com/spring-data-jpa-generate-db-schema) →

## 2。`data.sql`文件

让我们假设我们正在使用 JPA，并在我们的项目中定义一个简单的`Country`实体:

```
@Entity
public class Country {

    @Id
    @GeneratedValue(strategy = IDENTITY)
    private Integer id;

    @Column(nullable = false)
    private String name;

    //...
}
```

如果我们运行我们的应用程序， **Spring Boot 将为我们创建一个空表，但不会在其中填充任何东西。**

一个简单的方法是创建一个名为`data.sql`的文件:

```
INSERT INTO country (name) VALUES ('India');
INSERT INTO country (name) VALUES ('Brazil');
INSERT INTO country (name) VALUES ('USA');
INSERT INTO country (name) VALUES ('Italy');
```

当我们用类路径上的这个文件运行项目时，Spring 将获取它并使用它来填充数据库。

## 3。`schema.sql`文件

有时，我们不想依赖默认的模式创建机制。

在这种情况下，我们可以创建一个自定义的`schema.sql`文件:

```
CREATE TABLE country (
    id   INTEGER      NOT NULL AUTO_INCREMENT,
    name VARCHAR(128) NOT NULL,
    PRIMARY KEY (id)
);
```

Spring 将获取这个文件，并使用它来创建一个模式。

**请注意，基于脚本的初始化，即通过`schema.sql`和`data.sql`和 Hibernate 一起初始化会导致一些问题。**

我们要么禁用 Hibernate 自动模式创建:

`spring.jpa.hibernate.ddl-auto=none`

这将确保直接使用`schema.sql`和 `data.sql`执行基于脚本的初始化。

如果我们仍然希望 Hibernate 自动模式生成与基于脚本的模式创建和数据填充相结合，我们必须使用:

```
spring.jpa.defer-datasource-initialization=true
```

**这将确保在执行 Hibernate 模式创建之后，还会读取`schema.sql`以获取任何额外的模式更改，并执行`data.sql`以填充数据库。**

此外，默认情况下，只有嵌入式数据库才会执行基于脚本的初始化，要始终使用脚本初始化数据库，我们必须使用:

```
spring.sql.init.mode=always
```

请参考关于使用 SQL 脚本初始化数据库的官方 Spring 文档。

## 4。使用 Hibernate 控制数据库创建

Spring 提供了一个 JPA 特有的**属性，Hibernate 用它来生成 DDL:**`**spring.jpa.hibernate.ddl-auto**`**。**

标准休眠属性值为`create`、`update`、`create-drop`、`validate`和`none`:

*   `create`–Hibernate 首先删除现有的表，然后创建新的表。
*   `update`–将基于映射(注释或 XML)创建的对象模型与现有模式进行比较，然后 Hibernate 根据差异更新模式。它从不删除现有的表或列，即使应用程序不再需要它们。
*   `create-drop`——类似于`create`，只是 Hibernate 会在所有操作完成后丢弃数据库；通常用于单元测试
*   `validate`–Hibernate 只验证表和列是否存在；否则，它将引发异常。
*   `none`–该值有效关闭 DDL 生成。

如果没有检测到模式管理器，Spring Boot 内部将该参数值默认为`create-drop`，否则对于所有其他情况为`none`。

我们必须小心地设置该值，或者使用其他机制之一来初始化数据库。

## 5.定制数据库模式创建

默认情况下，Spring Boot 会自动创建一个嵌入式`DataSource`的模式。

如果我们需要控制或定制这种行为，我们可以使用属性 [`spring.sql.init.mode`](https://web.archive.org/web/20220707143831/https://docs.spring.io/spring-boot/docs/current-SNAPSHOT/api/org/springframework/boot/sql/init/DatabaseInitializationMode.html) 。该属性采用三个值之一:

*   `always`–始终初始化数据库
*   `embedded`–如果正在使用嵌入式数据库，请始终初始化。如果未指定属性值，这是默认值。
*   `never`–从不初始化数据库

值得注意的是，如果我们使用非嵌入式数据库，比如 MySQL 或 PostGreSQL，并且想要初始化它的模式，我们必须将这个属性设置为`always`。

该属性是在 Spring Boot 2.5.0 中引入的；如果我们使用 Spring Boot 的早期版本，我们需要使用`spring.datasource.initialization-mode` 。

## 6。`@Sql`

Spring 还提供了`@Sql`注释——一种初始化和填充测试模式的声明性方法。

让我们看看如何使用`@Sql`注释来创建一个新表，并为我们的集成测试加载初始数据:

```
@Sql({"/employees_schema.sql", "/import_employees.sql"})
public class SpringBootInitialLoadIntegrationTest {

    @Autowired
    private EmployeeRepository employeeRepository;

    @Test
    public void testLoadDataForTestClass() {
        assertEquals(3, employeeRepository.findAll().size());
    }
}
```

以下是`@Sql`注释的属性:

*   `config – `SQL 脚本的本地配置。我们将在下一节详细描述这一点。
*   `executionPhase`–我们还可以指定何时执行脚本，或者是`BEFORE_TEST_METHOD`或者是`AFTER_TEST_METHOD`。
*   `statements` –我们可以声明要执行的内联 SQL 语句。
*   `scripts` –我们可以声明要执行的 SQL 脚本文件的路径。这是`value `属性的别名。

`@Sql`注释**可以在类级别或方法级别使用。**

我们将通过注释该方法来加载特定测试用例所需的额外数据:

```
@Test
@Sql({"/import_senior_employees.sql"})
public void testLoadDataForTestCase() {
    assertEquals(5, employeeRepository.findAll().size());
}
```

## 7。`@SqlConfig`

我们可以通过使用`@SqlConfig`注释来**配置解析和运行 SQL 脚本**的方式。

`@SqlConfig`可以在类级别声明，在这里它作为一个全局配置。或者我们可以用它来配置一个特定的`@Sql`注释。

让我们看一个例子，其中我们指定了 SQL 脚本的编码以及执行脚本的事务模式:

```
@Test
@Sql(scripts = {"/import_senior_employees.sql"}, 
  config = @SqlConfig(encoding = "utf-8", transactionMode = TransactionMode.ISOLATED))
public void testLoadDataForTestCase() {
    assertEquals(5, employeeRepository.findAll().size());
}
```

而且我们来看看`@SqlConfig`的各种属性:

*   `blockCommentStartDelimiter`–分隔符，用于标识 SQL 脚本文件中块注释的开始
*   `blockCommentEndDelimiter`–表示 SQL 脚本文件中块注释结束的分隔符
*   `commentPrefix`–前缀，用于标识 SQL 脚本文件中的单行注释
*   `dataSource`–将运行脚本和语句的`javax.sql.DataSource` bean 的名称
*   `encoding`–SQL 脚本文件的编码；默认为平台编码
*   `errorMode`–运行脚本时遇到错误时使用的模式
*   `separator`–用于分隔单个语句的字符串；默认为“–”
*   `transactionManager`–将用于交易的`PlatformTransactionManager `的 bean 名称
*   `transactionMode`–在事务中执行脚本时将使用的模式

## 8。`@SqlGroup`

Java 8 和更高版本允许使用重复的注释。我们也可以将这个特性用于`@Sql`注释。对于 Java 7 及以下版本，有一个容器注释— `@SqlGroup`。

**使用`@SqlGroup`注释，我们将声明多个`@Sql`注释**:

```
@SqlGroup({
  @Sql(scripts = "/employees_schema.sql", 
    config = @SqlConfig(transactionMode = TransactionMode.ISOLATED)),
  @Sql("/import_employees.sql")})
public class SpringBootSqlGroupAnnotationIntegrationTest {

    @Autowired
    private EmployeeRepository employeeRepository;

    @Test
    public void testLoadDataForTestCase() {
        assertEquals(3, employeeRepository.findAll().size());
    }
}
```

## 9。结论

在这篇简短的文章中，我们看到了如何利用`schema.sql`和`data.sql`文件来建立一个初始模式并用数据填充它。

我们还看了如何使用`@Sql`、 `@SqlConfig`和` @SqlGroup `注释来加载测试数据。

请记住，这种方法更适合基本和简单的场景，任何高级数据库处理都需要更高级和更精细的工具，如 [Liquibase](/web/20220707143831/https://www.baeldung.com/liquibase-refactor-schema-of-java-app) 或 [Flyway](/web/20220707143831/https://www.baeldung.com/database-migrations-with-flyway) 。

代码片段一如既往地可以在 GitHub 上找到[。](https://web.archive.org/web/20220707143831/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-boot-persistence)