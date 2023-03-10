# rxjava-jdbc 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/rxjava-jdbc>

## 1。概述

简单地说，rxjava-jdbc 是一个用于与关系数据库交互的 API，它允许流畅的方法调用。在这个快速教程中，我们将看看这个库，以及如何利用它的一些通用特性。

如果您想了解 RxJava 的基础知识，请查看本文。

## 延伸阅读:

## [rx Java 简介](/web/20221129015757/https://www.baeldung.com/rx-java)

Discover RxJava - a library for composing asynchronous and event-based programs.[Read more](/web/20221129015757/https://www.baeldung.com/rx-java) →

## [用 RxJava 处理背压](/web/20221129015757/https://www.baeldung.com/rxjava-backpressure)

A guide demonstrating several strategies of handling backpressure in RxJava[Read more](/web/20221129015757/https://www.baeldung.com/rxjava-backpressure) →

## RxJava 中的可观察效用运算符

Learn how to use various RxJava utility operators.[Read more](/web/20221129015757/https://www.baeldung.com/rxjava-observable-operators) →

## 2。Maven 依赖关系

让我们从需要添加到我们的`pom.xml`中的 Maven 依赖项开始:

```java
<dependency>
    <groupId>com.github.davidmoten</groupId>
    <artifactId>rxjava-jdbc</artifactId>
    <version>0.7.11</version>
</dependency>
```

我们可以在 [Maven Central](https://web.archive.org/web/20221129015757/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22rxjava-jdbc%22) 上找到最新版本的 API。

## 3。主要部件

**`Database`类是运行所有常见类型的数据库交互的主要入口点。**为了创建一个`Database`对象，我们可以将一个`ConnectionProvider`接口实现的实例传递给`from()`静态方法:

```java
public static ConnectionProvider connectionProvider
  = new ConnectionProviderFromUrl(
  DB_CONNECTION, DB_USER, DB_PASSWORD);
Database db = Database.from(connectionProvider);
```

`ConnectionProvider`有几个值得一看的实现，比如`ConnectionProviderFromContext`、`ConnectionProviderFromDataSource`、`ConnectionProviderFromUrl`和`ConnectionProviderPooled`。

为了做基本的操作，我们可以使用`Database`的以下 API:

*   `select()`–用于 SQL 选择查询
*   `update()`–用于 DDL 语句，如创建和删除，以及插入、更新和删除

## 4。启动

在下一个快速示例中，我们将展示如何完成所有基本的数据库操作:

```java
public class BasicQueryTypesTest {

    Observable<Integer> create,
      insert1, 
      insert2, 
      insert3, 
      update, 
      delete = null;

    @Test
    public void whenCreateTableAndInsertRecords_thenCorrect() {
        create = db.update(
          "CREATE TABLE IF NOT EXISTS EMPLOYEE("
          + "id int primary key, name varchar(255))")
          .count();
        insert1 = db.update(
          "INSERT INTO EMPLOYEE(id, name) VALUES(1, 'John')")
          .dependsOn(create)
          .count();
        update = db.update(
          "UPDATE EMPLOYEE SET name = 'Alan' WHERE id = 1")
          .dependsOn(create)
          .count();
        insert2 = db.update(
          "INSERT INTO EMPLOYEE(id, name) VALUES(2, 'Sarah')")
          .dependsOn(create)
          .count();
        insert3 = db.update(
          "INSERT INTO EMPLOYEE(id, name) VALUES(3, 'Mike')")
          .dependsOn(create)
          .count();
        delete = db.update(
          "DELETE FROM EMPLOYEE WHERE id = 2")
          .dependsOn(create)
          .count();
        List<String> names = db.select(
          "select name from EMPLOYEE where id < ?")
          .parameter(3)
          .dependsOn(create)
          .dependsOn(insert1)
          .dependsOn(insert2)
          .dependsOn(insert3)
          .dependsOn(update)
          .dependsOn(delete)
          .getAs(String.class)
          .toList()
          .toBlocking()
          .single();

        assertEquals(Arrays.asList("Alan"), names);
    }
}
```

这里有一个简短的说明——我们调用`dependsOn()`来确定查询的运行顺序。

否则，代码将失败或产生不可预知的结果，除非我们指定希望查询以什么顺序执行。

## 5。自动地图

automap 特性允许我们将选定的数据库记录映射到对象。

让我们看一下自动映射数据库记录的两种方法。

### 5.1。使用接口自动映射

我们可以使用带注释的接口将数据库记录转换成对象。为此，我们可以创建一个带注释的接口:

```java
public interface Employee {

    @Column("id")
    int id();

    @Column("name")
    String name();
}
```

然后，我们可以运行我们的测试:

```java
@Test
public void whenSelectFromTableAndAutomap_thenCorrect() {
    List<Employee> employees = db.select("select id, name from EMPLOYEE")
      .dependsOn(create)
      .dependsOn(insert1)
      .dependsOn(insert2)
      .autoMap(Employee.class)
      .toList()
      .toBlocking()
      .single();

    assertThat(
      employees.get(0).id()).isEqualTo(1);
    assertThat(
      employees.get(0).name()).isEqualTo("Alan");
    assertThat(
      employees.get(1).id()).isEqualTo(2);
    assertThat(
      employees.get(1).name()).isEqualTo("Sarah");
}
```

### 5.2。使用类自动映射

我们还可以使用具体的类将数据库记录自动映射到对象。让我们看看这个类是什么样子的:

```java
public class Manager {

    private int id;
    private String name;

    // standard constructors, getters, and setters
}
```

现在，我们可以运行我们的测试:

```java
@Test
public void whenSelectManagersAndAutomap_thenCorrect() {
    List<Manager> managers = db.select("select id, name from MANAGER")
      .dependsOn(create)
      .dependsOn(insert1)
      .dependsOn(insert2)
      .autoMap(Manager.class)
      .toList()
      .toBlocking()
      .single();

    assertThat(
      managers.get(0).getId()).isEqualTo(1);
    assertThat(
     managers.get(0).getName()).isEqualTo("Alan");
    assertThat(
      managers.get(1).getId()).isEqualTo(2);
    assertThat(
      managers.get(1).getName()).isEqualTo("Sarah");
}
```

这里有一些注意事项:

*   `create`、`insert1`、`insert2`是对`Observables`的引用，通过创建`Manager`表并将记录插入其中返回
*   查询中所选列的数量必须与`Manager`类构造函数中的参数数量相匹配
*   列的类型必须能够自动映射到构造函数中的类型

有关自动映射的更多信息，请访问 GitHub 上的 [rxjava-jdbc 存储库](https://web.archive.org/web/20221129015757/https://github.com/davidmoten/rxjava-jdbc)

## 6。处理大型物体

该 API 支持处理像 CLOBs 和 BLOBS 这样的大型对象。在接下来的小节中，我们将了解如何利用这一功能。

### 6.1。土块

让我们看看如何插入和选择 CLOB:

```java
@Before
public void setup() throws IOException {
    create = db.update(
      "CREATE TABLE IF NOT EXISTS " + 
      "SERVERLOG (id int primary key, document CLOB)")
        .count();

    InputStream actualInputStream
      = new FileInputStream("src/test/resources/actual_clob");
    actualDocument = getStringFromInputStream(actualInputStream);

    InputStream expectedInputStream = new FileInputStream(
      "src/test/resources/expected_clob");

    expectedDocument = getStringFromInputStream(expectedInputStream);
    insert = db.update(
      "insert into SERVERLOG(id,document) values(?,?)")
        .parameter(1)
        .parameter(Database.toSentinelIfNull(actualDocument))
      .dependsOn(create)
      .count();
}

@Test
public void whenSelectCLOB_thenCorrect() throws IOException {
    db.select("select document from SERVERLOG where id = 1")
      .dependsOn(create)
      .dependsOn(insert)
      .getAs(String.class)
      .toList()
      .toBlocking()
      .single();

    assertEquals(expectedDocument, actualDocument);
}
```

注意，`getStringFromInputStream()`是一个转换`InputStream to a String.`内容的方法

### 6.2。斑点

我们可以使用 API 以非常相似的方式处理 BLOBs。唯一的区别是，我们必须传递一个字节数组，而不是将一个`String`传递给`toSentinelIfNull()`方法。

我们可以这样做:

```java
@Before
public void setup() throws IOException {
    create = db.update(
      "CREATE TABLE IF NOT EXISTS " 
      + "SERVERLOG (id int primary key, document BLOB)")
        .count();

    InputStream actualInputStream
      = new FileInputStream("src/test/resources/actual_clob");
    actualDocument = getStringFromInputStream(actualInputStream);
    byte[] bytes = this.actualDocument.getBytes(StandardCharsets.UTF_8);

    InputStream expectedInputStream = new FileInputStream(
      "src/test/resources/expected_clob");
    expectedDocument = getStringFromInputStream(expectedInputStream);
    insert = db.update(
      "insert into SERVERLOG(id,document) values(?,?)")
      .parameter(1)
      .parameter(Database.toSentinelIfNull(bytes))
      .dependsOn(create)
      .count();
}
```

然后，我们可以重用前面例子中的相同测试。

## 7 .**。交易**

接下来，我们来看看对事务的支持。

事务管理允许我们处理用于在单个事务中分组多个数据库操作的事务，以便它们都可以被提交——永久保存到数据库，或者一起回滚。

让我们看一个简单的例子:

```java
@Test
public void whenCommitTransaction_thenRecordUpdated() {
    Observable<Boolean> begin = db.beginTransaction();
    Observable<Integer> createStatement = db.update(
      "CREATE TABLE IF NOT EXISTS EMPLOYEE(id int primary key, name varchar(255))")
      .dependsOn(begin)
      .count();
    Observable<Integer> insertStatement = db.update(
      "INSERT INTO EMPLOYEE(id, name) VALUES(1, 'John')")
      .dependsOn(createStatement)
      .count();
    Observable<Integer> updateStatement = db.update(
      "UPDATE EMPLOYEE SET name = 'Tom' WHERE id = 1")
      .dependsOn(insertStatement)
      .count();
    Observable<Boolean> commit = db.commit(updateStatement);
    String name = db.select("select name from EMPLOYEE WHERE id = 1")
      .dependsOn(commit)
      .getAs(String.class)
      .toBlocking()
      .single();

    assertEquals("Tom", name);
}
```

为了启动一个事务，我们调用方法`beginTransaction()`。在这个方法被调用后，每个数据库操作都在同一个事务中运行，直到方法`commit()`或`rollback()`被调用。

我们可以使用`rollback()`方法，同时捕捉一个`Exception`来回滚整个事务，以防代码由于任何原因而失败。我们可以为所有的`Exceptions`或特定的预期`Exceptions`这样做。

## 8。返回生成的密钥

如果我们在正在处理的表中设置了`auto_increment`字段，我们可能需要检索生成的值。我们可以通过调用`returnGeneratedKeys()`方法来做到这一点。

让我们看一个简单的例子:

```java
@Test
public void whenInsertAndReturnGeneratedKey_thenCorrect() {
    Integer key = db.update("INSERT INTO EMPLOYEE(name) VALUES('John')")
      .dependsOn(createStatement)
      .returnGeneratedKeys()
      .getAs(Integer.class)
      .count()
      .toBlocking()
      .single();

    assertThat(key).isEqualTo(1);
}
```

## 9。结论

在本教程中，我们已经看到了如何利用 rxjava `–` jdbc 的流畅风格的方法。我们还浏览了它提供的一些特性，比如自动映射、处理大型对象和事务。

和往常一样，GitHub 上有完整版本的代码[。](https://web.archive.org/web/20221129015757/https://github.com/eugenp/tutorials/tree/master/rxjava-modules/rxjava-libraries)