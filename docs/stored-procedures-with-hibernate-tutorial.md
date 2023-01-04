# Hibernate 存储过程

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/stored-procedures-with-hibernate-tutorial>

## 1。概述

存储过程是驻留在数据库中的一组经过编译的 SQL 语句**。**它们用于封装逻辑并与其他程序共享逻辑，并受益于数据库特有的功能，如索引提示或特定关键字。

这篇文章演示了如何使用 **Hibernate** 调用 **MySQL 数据库**中的**存储过程**。

## 2。MySQL 中的存储过程

在讨论如何从 Hibernate 调用存储过程之前，我们需要创建它。

对于这个简单的 MySQL 示例，我们将[创建一个存储过程](https://web.archive.org/web/20220626115431/https://dev.mysql.com/doc/refman/5.7/en/create-procedure.html)来从`**foo**` 表中获取所有记录。

为了创建一个存储过程，我们使用了`CREATE PROCEDURE`语句:

```java
DELIMITER //
    CREATE PROCEDURE GetAllFoos()
        LANGUAGE SQL
        DETERMINISTIC
        SQL SECURITY DEFINER
        BEGIN
            SELECT * FROM foo;
        END //
DELIMITER;
```

在`BEGIN` 语句之前，我们可以定义可选语句。你可以通过官方的 [MySQL 文档](https://web.archive.org/web/20220626115431/https://dev.mysql.com/doc/refman/5.7/en/create-procedure.html)链接深入了解这些声明的细节。

我们可以使用`[CALL](https://web.archive.org/web/20220626115431/https://dev.mysql.com/doc/refman/5.7/en/call.html)`语句来确保我们的过程以期望的方式运行:

```java
CALL GetAllFoos();
```

既然我们已经启动并运行了存储过程，让我们直接跳到如何从 Hibernate 调用它。

## 3。用 Hibernate 调用存储过程

从 Hibernate 3 开始，我们可以使用包含存储过程的原始 SQL 语句来查询数据库。

在这一节中，我们将通过一个看似基本的例子来说明如何使用 Hibernate 调用 **`GetAllFoos()`** 过程。

### 3.1。配置

在开始编写可以运行的代码之前，我们需要在项目中配置 Hibernate。

当然，对于所有这些 Maven 依赖、MySQL 配置、Hibernate 配置和 **`SessionFactory`** 实例化——你可以查看 [Hibernate 文章](/web/20220626115431/https://www.baeldung.com/hibernate-4-spring)。

### 3.2。使用`CreateNativeSQL` 方法调用存储过程

Hibernate 允许直接用**原生 SQL** 格式表达查询。因此，我们可以直接创建一个原生 SQL 查询，并使用`CALL` 语句调用`getAllFoos()`存储过程:

```java
Query query = session.createSQLQuery("CALL GetAllFoos()").addEntity(Foo.class);
List<Foo> allFoos = query.list(); 
```

上面的查询返回一个列表，其中每个元素都是一个`Foo o`对象。

我们使用 **`addEntity()`** 方法从本机 **SQL** 查询中获取实体对象，否则，每当存储过程返回非原始值时，就会抛出 **`ClassCastException`** 。

### 3.3。使用`@NamedNativeQueries`调用存储过程

调用存储过程的另一种方法是使用`**@NamedNativeQueries**`注释。

**`@NamedNativeQueries`** 用于指定本地 **SQL** 命名查询**的数组，作用域为持久化单元:**

```java
@NamedNativeQueries({ 
  @NamedNativeQuery(
    name = "callGetAllFoos", 
    query = "CALL GetAllFoos()", 
    resultClass = Foo.class) 
})
@Entity
public class Foo implements Serializable {
    // Model definition
}
```

显然，每个命名查询都有一个**名称**属性，实际的 **SQL 查询**，以及引用 **Foo** 映射实体的`**resultClass**` 。

```java
Query query = session.getNamedQuery("callGetAllFoos");
List<Foo> allFoos = query.list();
```

在我们前面的例子中，`**resultClass**` 属性扮演着与 **`addEntity()`** 方法相同的角色。

这两种方法可以互换使用，因为两者在性能或生产率方面没有真正的区别。

### 3.4。使用`@NamedStoredProcedureQuery`调用存储过程

如果你使用的是 **JPA 2.1** 和 **Hibernate** 实现的`**EntityManagerFactory**` 和`**EntityManager**`。

**`@NamedStoredProcedureQuery`** 注释可以用来声明一个存储过程:

```java
@NamedStoredProcedureQuery(
  name="GetAllFoos",
  procedureName="GetAllFoos",
  resultClasses = { Foo.class }
)
@Entity
public class Foo implements Serializable {
    // Model Definition 
} 
```

为了调用我们的命名存储过程查询，我们需要实例化一个`**EntityManager,**` ，然后调用`**createNamedStoredProcedureQuery()**`方法来创建过程 ***:***

```java
StoredProcedureQuery spQuery = 
  entityManager.createNamedStoredProcedureQuery("getAllFoos"); 
```

我们可以通过调用`**StoredProcedureQuery**` 对象上的`**execute()**`方法直接得到`**Foo**` 实体的列表。

## 4。带参数的存储过程

几乎所有的存储过程都需要参数。在这一节中，我们将展示如何使用来自 **Hibernate** 的参数调用存储过程。

让我们在 **MySQL** 中创建一个`getFoosByName()`存储过程。

该过程返回一个列表，其中包含名称属性与`fooName` 参数匹配的`Foo` 对象:

```java
DELIMITER //
    CREATE PROCEDURE GetFoosByName(IN fooName VARCHAR(255))
        LANGUAGE SQL
        DETERMINISTIC
        SQL SECURITY DEFINER
        BEGIN
            SELECT * FROM foo WHERE name = fooName;
        END //
DELIMITER;
```

为了调用`**GetFoosByName()**` 过程，我们将使用命名参数:

```java
Query query = session.createSQLQuery("CALL GetFoosByName(:fooName)")
  .addEntity(Foo.class)
  .setParameter("fooName","New Foo");
```

同样，命名参数 **`:fooName`** 可以与`**@NamedNativeQuery**`注释一起使用:

```java
@NamedNativeQuery(
  name = "callGetFoosByName", 
  query = "CALL GetFoosByName(:fooName)", 
  resultClass = Foo.class
)
```

命名查询的调用如下:

```java
Query query = session.getNamedQuery("callGetFoosByName")
  .setParameter("fooName","New Foo");
```

当使用 **`@NamedStoredProcedureQuery`** 标注时，我们可以使用 **`@StoredProcedureParameter`标注**来指定参数:

```java
@NamedStoredProcedureQuery(
  name="GetFoosByName",
  procedureName="GetFoosByName",
  resultClasses = { Foo.class },
  parameters={
    @StoredProcedureParameter(name="fooName", type=String.class, mode=ParameterMode.IN)
  }
) 
```

我们可以利用 `**setParameter()**`方法调用带有`fooName` 参数的存储过程:

```java
StoredProcedureQuery spQuery = entityManager.createNamedStoredProcedureQuery("GetFoosByName")
  .setParameter("fooName", "NewFooName");
```

## 5。结论

本文展示了如何使用 Hibernate 通过不同的方法调用 MySQL 数据库中的存储过程。

值得一提的是**并不是所有的 RDBMS 都支持存储过程**。

您可以在链接的 [GitHub 项目](https://web.archive.org/web/20220626115431/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-data-jpa-query-2)中查看本文提供的示例。