# ORMLite 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/ormlite>

## 1。概述

ORMLite 是一个用于 Java 应用程序的轻量级 ORM 库。它为最常见的用例**提供了 ORM 工具的标准特性，而没有其他 ORM 框架所增加的复杂性和开销。**

它的主要特点是:

*   使用 Java 注释定义实体类
*   可扩展的`DAO`类
*   用于创建复杂查询的`QueryBuilder`类
*   为创建和删除数据库表生成的类
*   对交易的支持
*   支持实体关系

在接下来的小节中，我们将看看如何设置库，定义实体类以及使用库在数据库上执行操作。

## 2。Maven 依赖关系

要开始使用 ORMLite，我们需要将`[ormlite-jdbc](https://web.archive.org/web/20220526045555/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22ormlite-jdbc%22)`依赖项添加到我们的`pom.xml`中:

```java
<dependency>
    <groupId>com.j256.ormlite</groupId>
    <artifactId>ormlite-jdbc</artifactId>
    <version>5.0</version>
</dependency>
```

默认情况下，这也带来了 [`h2`](https://web.archive.org/web/20220526045555/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22h2%22%20AND%20g%3A%22com.h2database%22) 的依赖性。在我们的例子中，我们将使用一个`H2`内存数据库，所以我们不需要另一个 JDBC 驱动程序。

如果您想使用不同的数据库，您还需要相应的依赖项。

## 3。定义实体类

要用 ORMLite 为持久性设置我们的模型类，我们可以使用两个主要的注释:

*   `@DatabaseTable`为实体类
*   `@DatabaseField`为属性

让我们首先定义一个带有一个`name`字段和一个`libraryId`字段的`Library`实体，这也是一个主键:

```java
@DatabaseTable(tableName = "libraries")
public class Library {	

    @DatabaseField(generatedId = true)
    private long libraryId;

    @DatabaseField(canBeNull = false)
    private String name;

    public Library() {
    }

    // standard getters, setters
}
```

如果我们不想依赖默认的类名，`@DatabaseTable`注释有一个可选的`tableName`属性来指定表的名称。

**对于我们希望在数据库表中作为列保存的每个字段，我们必须添加 `@DatabaseField`注释。**

将作为表的主键的属性可以用`id`、`generatedId`或`generatedSequence`属性来标记。在我们的例子中，我们选择了`generatedId=true`属性，这样主键将自动递增。

**另外，请注意，该类需要一个至少具有`package-scope`可见性的无参数构造函数。**

我们可以用来配置字段的其他一些常见属性是`columnName`、`dataType`、`defaultValue`、`canBeNull`、`unique`。

### 3.1。使用`JPA`标注

除了特定于 ORMLite 的注释，**我们还可以使用`JPA-style`注释来定义我们的实体**。

在使用`JPA`标准注释之前，我们定义的`Library`实体的等效实体是:

```java
@Entity
public class LibraryJPA {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private long libraryId;

    @Column
    private String name;

    // standard getters, setters
}
```

尽管 ORMLite 可以识别这些注释，但我们仍然需要添加`[javax.persistence-api](https://web.archive.org/web/20220526045555/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22javax.persistence-api%22)`依赖项来使用它们。

支持的`JPA`注释的完整列表是@ `Entity`、`@Id,`、`@Column,`、@ `OneToOne`、`@ManyToOne,`、@ `JoinColumn`、@ `Version`。

## 4。`ConnectionSource`

为了使用定义的对象，**我们需要设置一个`ConnectionSource`** 。

为此，我们可以使用创建单个连接的`JdbcConnectionSource`类，或者使用代表简单池连接源的`JdbcPooledConnectionSource`:

```java
JdbcPooledConnectionSource connectionSource 
  = new JdbcPooledConnectionSource("jdbc:h2:mem:myDb");

// work with the connectionSource

connectionSource.close();
```

通过将其他具有更好性能的外部数据源包装在一个`DataSourceConnectionSource`对象中，也可以使用它们。

## 5。`TableUtils`阶级

基于`ConnectionSource`、**，我们可以使用`TableUtils`类中的静态方法对数据库模式**执行操作:

*   `createTable()`–基于实体类定义或`DatabaseTableConfig`对象创建表格
*   `createTableIfNotExists()`–与前一种方法类似，只是它只创建不存在的表；这只适用于支持它的数据库
*   `dropTable()`–删除表格
*   `clearTable`()–从表格中删除数据

让我们看看如何使用`TableUtils`为我们的`Library`类创建表格:

```java
TableUtils.createTableIfNotExists(connectionSource, Library.class);
```

## 6。`DAO`对象

ORMLite 包含了**一个`DaoManager`类，它可以用 CRUD 功能**为我们创建`DAO`对象:

```java
Dao<Library, Long> libraryDao 
  = DaoManager.createDao(connectionSource, Library.class);
```

`DaoManager`不会为`createDao(),`的每个后续调用重新生成类，而是重用它以获得更好的性能。

接下来，我们可以对`Library`对象执行 CRUD 操作:

```java
Library library = new Library();
library.setName("My Library");
libraryDao.create(library);

Library result = libraryDao.queryForId(1L);

library.setName("My Other Library");
libraryDao.update(library);

libraryDao.delete(library);
```

`DAO`也是一个迭代器，可以遍历所有记录:

```java
libraryDao.forEach(lib -> {
    System.out.println(lib.getName());
});
```

但是，ORMLite 只会在循环一直进行到最后时关闭底层 SQL 语句。异常或 return 语句可能会导致代码中的资源泄漏。

因此，ORMLite 文档建议我们直接使用迭代器:

```java
try (CloseableWrappedIterable<Library> wrappedIterable 
  = libraryDao.getWrappedIterable()) {
    wrappedIterable.forEach(lib -> {
        System.out.println(lib.getName());
    });
 }
```

这样，我们可以使用`try-with-resources`或`finally` 块关闭迭代器，避免任何资源泄漏。

### 6.1。自定义刀类

如果我们想要扩展所提供的`DAO` 对象的行为，我们可以创建一个扩展`Dao`类型的新接口:

```java
public interface LibraryDao extends Dao<Library, Long> {
    public List<Library> findByName(String name) throws SQLException;
}
```

然后，让我们添加一个实现这个接口并扩展了`BaseDaoImpl`类的类:

```java
public class LibraryDaoImpl extends BaseDaoImpl<Library, Long> 
  implements LibraryDao {
    public LibraryDaoImpl(ConnectionSource connectionSource) throws SQLException {
        super(connectionSource, Library.class);
    }

    @Override
    public List<Library> findByName(String name) throws SQLException {
        return super.queryForEq("name", name);
    }
}
```

注意，我们需要一个这种形式的构造函数。

最后，为了使用我们的定制`DAO,`，我们需要将类名添加到`Library`类定义中:

```java
@DatabaseTable(tableName = "libraries", daoClass = LibraryDaoImpl.class)
public class Library { 
    // ...
}
```

这使我们能够使用`DaoManager`来创建自定义类的实例:

```java
LibraryDao customLibraryDao 
  = DaoManager.createDao(connectionSource, Library.class);
```

然后我们可以使用标准`DAO`类中的所有方法，以及我们的自定义方法:

```java
Library library = new Library();
library.setName("My Library");

customLibraryDao.create(library);
assertEquals(
  1, customLibraryDao.findByName("My Library").size());
```

## 7。定义实体关系

ORMLite 使用“外来”对象或集合的概念来定义实体之间的持久关系。

让我们看看如何定义每种类型的字段。

### 7.1。外来物体字段

我们可以通过在用`@DatabaseField`注释的字段上使用`foreign=true`属性来创建两个实体类之间的单向一对一关系。该字段的类型必须也保存在数据库中。

首先，让我们定义一个名为`Address`的新实体类:

```java
@DatabaseTable(tableName="addresses")
public class Address {
    @DatabaseField(generatedId = true)
    private long addressId;

    @DatabaseField(canBeNull = false)
    private String addressLine;

    // standard getters, setters 
}
```

接下来，我们可以将类型为`Address`的字段添加到标记为`foreign`的`Library`类中:

```java
@DatabaseTable(tableName = "libraries")
public class Library {      
    //...

    @DatabaseField(foreign=true, foreignAutoCreate = true, 
      foreignAutoRefresh = true)
    private Address address;

    // standard getters, setters
}
```

请注意，我们还向`@DatabaseField`注释添加了两个属性:`foreignAutoCreate`和`foreignAutoRefresh`，它们都被设置为`true.`

`foreignAutoCreate=true`属性意味着当我们保存一个带有`address`字段的`Library`对象时，如果它的`id`不为空并且有一个`generatedId=true`属性，那么外来对象也将被保存。

如果我们将`foreignAutoCreate`设置为`false`，这是默认值，那么我们需要在保存引用它的`Library`对象之前显式地持久化这个外来对象。

类似地，`foreignAutoRefresh` = `true`属性指定当检索一个`Library`对象时，关联的外来对象也将被检索。否则，我们需要手动刷新它。

让我们添加一个带有`Address`字段的新的`Library`对象，并调用`libraryDao`来持久化这两个对象:

```java
Library library = new Library();
library.setName("My Library");
library.setAddress(new Address("Main Street nr 20"));

Dao<Library, Long> libraryDao 
  = DaoManager.createDao(connectionSource, Library.class);
libraryDao.create(library);
```

然后，我们可以调用`addressDao`来验证`Address`也被保存了:

```java
Dao<Address, Long> addressDao 
  = DaoManager.createDao(connectionSource, Address.class);
assertEquals(1, 
  addressDao.queryForEq("addressLine", "Main Street nr 20")
  .size());
```

### 7.2。国外收款

对于关系的`many`方，我们可以使用带有`@ForeignCollectionField`注释的类型`ForeignCollection<T>`或`Collection<T>`。

让我们像上面一样创建一个新的`Book`实体，然后在`Library`类中添加一个一对多关系:

```java
@DatabaseTable(tableName = "libraries")
public class Library {  
    // ...

    @ForeignCollectionField(eager=false)
    private ForeignCollection<Book> books;

    // standard getters, setters
}
```

除此之外，我们还需要在`Book`类中添加一个类型为`Library`的字段:

```java
@DatabaseTable
public class Book {
    // ...
    @DatabaseField(foreign = true, foreignAutoRefresh = true) 
    private Library library;

    // standard getters, setters
}
```

**`ForeignCollection`有`add()`和`remove()`方法**，它们操作类型`Book:`的记录

```java
Library library = new Library();
library.setName("My Library");
libraryDao.create(library);

libraryDao.refresh(library);

library.getBooks().add(new Book("1984"));
```

这里，我们已经创建了一个`library`对象，然后向`books`字段添加了一个新的`Book`对象，这也将它持久化到数据库中。

**注意，由于我们的集合被标记为延迟加载`(eager=false),`，我们需要调用`refresh()`方法**才能使用 book 字段。

我们还可以通过设置`Book`类中的`library` 字段来创建关系:

```java
Book book = new Book("It");
book.setLibrary(library);
bookDao.create(book);
```

为了验证两个`Book`对象都被添加到了`library`中，我们可以使用`queryForEq()`方法找到所有带有给定`library_id`的`Book`记录

```java
assertEquals(2, bookDao.queryForEq("library_id", library).size());
```

这里，`library_id`是外键列的默认名称，主键是从`library`对象中推断出来的。

## 8。`QueryBuilder`

每个`DAO`都可以用来获得一个`QueryBuilder`对象，然后我们可以利用它来构建更强大的查询。

这个类包含对应于 SQL 查询中常用操作的方法，例如:`selectColumns(), where(), groupBy(),` `having(), countOf(), distinct(), orderBy(), join().`

让我们来看一个例子，看看如何找到所有与多个`Book`相关联的`Library`记录:

```java
List<Library> libraries = libraryDao.queryBuilder()
  .where()
  .in("libraryId", bookDao.queryBuilder()
    .selectColumns("library_id")
    .groupBy("library_id")
    .having("count(*) > 1"))
  .query();
```

## 9。结论

在本文中，我们已经了解了如何使用 ORMLite 定义实体，以及可以用来操作对象及其相关关系数据库的库的主要特性。

这个例子的完整源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220526045555/https://github.com/eugenp/tutorials/tree/master/libraries-data-db)