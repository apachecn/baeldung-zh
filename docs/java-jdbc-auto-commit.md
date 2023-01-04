# JDBC 自动提交指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-jdbc-auto-commit>

## 1。简介

用 [JDBC](/web/20220926201541/https://www.baeldung.com/java-jdbc) API 创建的数据库[连接](https://web.archive.org/web/20220926201541/https://docs.oracle.com/en/java/javase/11/docs/api/java.sql/java/sql/Connection.html)有一个特性叫做[自动提交模式](https://web.archive.org/web/20220926201541/https://www.ibm.com/docs/en/i/7.4?topic=transactions-jdbc-auto-commit-mode)。

打开这种模式有助于消除管理交易所需的样板代码。然而，尽管如此，当执行 SQL [语句](https://web.archive.org/web/20220926201541/https://docs.oracle.com/en/java/javase/11/docs/api/java.sql/java/sql/Statement.html)时，它的目的以及它如何影响事务处理有时可能并不清楚。

在本文中，我们将讨论什么是自动提交模式，以及如何正确地将其用于自动和显式事务管理。我们还将讨论自动提交打开或关闭时要避免的各种问题。

## 2。什么是 JDBC 自动提交模式？

在使用 JDBC 时，开发人员不一定了解如何有效地管理数据库事务。因此，如果手动处理事务，开发人员可能不会在适当的时候启动它们，或者根本不启动它们。同样的问题也适用于在必要时发布[提交](https://web.archive.org/web/20220926201541/https://docs.oracle.com/en/java/javase/11/docs/api/java.sql/java/sql/Connection.html#commit())或[回滚](https://web.archive.org/web/20220926201541/https://docs.oracle.com/en/java/javase/11/docs/api/java.sql/java/sql/Connection.html#rollback())。

为了解决这个问题，JDBC 的**自动提交模式提供了一种执行 SQL 语句的方法，事务管理由 [JDBC 驱动程序](/web/20220926201541/https://www.baeldung.com/java-jdbc-loading-drivers)自动处理。**

因此，自动提交模式的目的是减轻开发人员自己管理事务的负担。这样，打开它可以更容易地用 JDBC API 开发应用程序。当然，这只在每个 SQL 语句完成后立即持久化数据更新是可以接受的情况下才有帮助。

## 3。自动提交为真时的自动事务管理

默认情况下，JDBC 驱动程序为新的数据库连接打开自动提交模式。当它打开时，它们会在自己的事务中自动运行每个 SQL 语句。

除了使用这个默认设置，我们还可以通过将`true`传递给连接的 [`setAutoCommit`](https://web.archive.org/web/20220926201541/https://docs.oracle.com/en/java/javase/11/docs/api/java.sql/java/sql/Connection.html#setAutoCommit(boolean)) 方法来手动打开自动提交:

```java
connection.setAutoCommit(true);
```

这种自己打开它的方式在我们以前关闭它，但后来我们需要恢复自动事务管理时很有用。

既然我们已经介绍了如何确保自动提交打开，我们将演示当这样设置时，JDBC 驱动程序在它们自己的事务中运行 SQL 语句。也就是说，它们会立即自动将每个语句中的任何数据更新提交给数据库。

### 3.1。设置示例代码

在这个例子中，我们将使用 [H2 内存数据库](https://web.archive.org/web/20220926201541/https://www.h2database.com/)来存储我们的数据。为了使用它，我们首先需要定义 Maven [依赖关系](https://web.archive.org/web/20220926201541/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22com.h2database%22%20AND%20a%3A%22h2%22):

```java
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <version>1.4.200</version>
</dependency>
```

首先，让我们创建一个数据库表来保存人员的详细信息:

```java
CREATE TABLE Person (
    id INTEGER not null,
    name VARCHAR(50),
    lastName VARCHAR(50),
    age INTEGER,PRIMARY KEY (id)
)
```

接下来，我们将创建两个到数据库的连接。我们将使用第一个来运行 SQL 查询并更新表。我们将使用第二个连接来测试是否对该表进行了更新:

```java
Connection connection1 = DriverManager.getConnection("jdbc:h2:mem:testdb", "sa", "");
Connection connection2 = DriverManager.getConnection("jdbc:h2:mem:testdb", "sa", "");
```

注意，我们需要使用一个单独的连接来测试提交的数据。这是因为如果我们在第一个连接上运行任何选择查询，那么他们将看到尚未提交的更新。

现在，我们将创建一个 POJO 来表示保存个人信息的数据库记录:

```java
public class Person {

    private Integer id;
    private String name;
    private String lastName;
    private Integer age;

    // standard constructor, getters, and setters
}
```

为了向我们的表中插入一条记录，让我们创建一个名为`insertPerson`的方法:

```java
private static int insertPerson(Connection connection, Person person) throws SQLException {    
    try (PreparedStatement preparedStatement = connection.prepareStatement(
      "INSERT INTO Person VALUES (?,?,?,?)")) {

        preparedStatement.setInt(1, person.getId());
        preparedStatement.setString(2, person.getName());
        preparedStatement.setString(3, person.getLastName());
        preparedStatement.setInt(4, person.getAge());

        return preparedStatement.executeUpdate();
    }        
} 
```

然后我们将添加一个`updatePersonAgeById`方法来更新表中的特定记录:

```java
private static void updatePersonAgeById(Connection connection, int id, int newAge) throws SQLException {
    try (PreparedStatement preparedStatement = connection.prepareStatement(
      "UPDATE Person SET age = ? WHERE id = ?")) {
        preparedStatement.setInt(1, newAge);
        preparedStatement.setInt(2, id);

        preparedStatement.executeUpdate();
    }
} 
```

最后，让我们添加一个`selectAllPeople`方法来从表中选择所有记录。我们将用它来检查 SQL `insert`和`update`语句的结果:

```java
private static List selectAllPeople(Connection connection) throws SQLException {

    List people = null;

    try (Statement statement = connection.createStatement()) {
        people = new ArrayList();
        ResultSet resultSet = statement.executeQuery("SELECT * FROM Person");

        while (resultSet.next()) {
            Person person = new Person();
            person.setId(resultSet.getInt("id"));
            person.setName(resultSet.getString("name"));
            person.setLastName(resultSet.getString("lastName"));
            person.setAge(resultSet.getInt("age"));

            people.add(person);
        }
    }

    return people;
} 
```

有了这些实用方法，我们将测试打开自动提交的效果。

### 3.2。运行测试

为了测试我们的示例代码，让我们首先在表中插入一个人。然后，从不同的连接，我们将检查数据库是否已经更新，而不需要我们发出`commit`:

```java
Person person = new Person(1, "John", "Doe", 45);
insertPerson(connection1, person);

List people = selectAllPeople(connection2);
assertThat("person record inserted OK into empty table", people.size(), is(equalTo(1)));
Person personInserted = people.iterator().next();
assertThat("id correct", personInserted.getId(), is(equalTo(1))); 
```

然后，将这个新记录插入到表中，让我们更新这个人的年龄。此后，我们将从第二个连接开始检查更改是否已经保存到数据库中，而不需要调用`commit`:

```java
updatePersonAgeById(connection1, 1, 65);

people = selectAllPeople(connection2);
Person personUpdated = people.iterator().next();
assertThat("updated age correct", personUpdated.getAge(), is(equalTo(65))); 
```

因此，我们已经通过测试验证了当自动提交模式打开时，JDBC 驱动程序会在自己的事务中隐式运行每个 SQL 语句。因此，我们不需要自己调用`commit`来保存对数据库的更新。

## 4。自动提交为假时的显式事务管理

当我们想要自己处理事务并将多个 SQL 语句组合成一个事务时，我们需要禁用自动提交模式。

我们通过传递`false to` 连接的`setAutoCommit`方法来做到这一点:

```java
connection.setAutoCommit(false);
```

**当自动提交模式关闭时，我们需要通过在连接上调用`commit`或`rollback` 来手动标记每个事务的结束。**

但是，我们需要注意，即使关闭了自动提交，JDBC 驱动程序仍然会在需要时自动为我们启动一个事务。例如，这发生在我们运行第一个 SQL 语句之前，也发生在每次提交或回滚之后。

让我们演示一下，当我们在 auto-commit 关闭的情况下执行多个 SQL 语句时，产生的更新将只在我们调用`commit`时保存到数据库中。

### 4.1。运行测试

首先，让我们使用第一个连接插入人员记录。然后，在不调用`commit`的情况下，我们将断言我们无法从另一个连接看到数据库中插入的记录:

```java
Person person = new Person(1, "John", "Doe", 45);
insertPerson(connection1, person);

List<Person> people = selectAllPeople(connection2);
assertThat("No people have been inserted into database yet", people.size(), is(equalTo(0))); 
```

接下来，我们将在记录中更新这个人的年龄。和以前一样，我们将在不调用 commit 的情况下断言，我们仍然不能使用第二个连接从数据库中选择记录:

```java
updatePersonAgeById(connection1, 1, 65);

people = selectAllPeople(connection2);
assertThat("No people have been inserted into database yet", people.size(), is(equalTo(0))); 
```

为了完成我们的测试，我们将调用`commit`，然后断言我们现在可以使用第二个连接看到数据库中的所有更新:

```java
connection1.commit();

people = selectAllPeople(connection2);
Person personUpdated = people.iterator().next();
assertThat("person's age updated to 65", personUpdated.getAge(), is(equalTo(65))); 
```

正如我们在上面的测试中所验证的，当自动提交模式关闭时，我们需要手动调用`commit`来保存我们对数据库的更改。这样做将保存自当前事务开始以来我们执行的所有 SQL 语句的任何更新。如果这是我们的第一笔交易，那就是从我们打开连接开始，否则就是在我们最后的`commit`或`rollback`之后。

## 5。注意事项和潜在问题

对于相对简单的应用程序，打开自动提交来运行 SQL 语句会很方便。也就是说，不需要人工交易控制。然而，在更复杂的情况下，我们应该考虑当 JDBC 驱动程序自动处理事务时，这有时可能会导致不必要的副作用或问题。

我们需要考虑的一件事是，当**自动提交开启时，它可能会浪费大量的处理时间和资源**。这是因为它会导致驱动程序在自己的事务中运行每个 SQL 语句，无论是否需要。

例如，如果我们用不同的值多次执行相同的语句，那么每个调用都被包装在自己的事务中。因此，这会导致不必要的执行和资源管理开销。

因此，在这种情况下，我们最好关闭自动提交，并将对同一 SQL 语句的多个调用显式批处理到一个事务中。这样，我们很可能会看到应用程序性能的显著提高。

另外需要注意的是，我们不建议在打开事务期间重新打开自动提交。这是因为**在事务中间打开自动提交模式将提交所有待定的数据库更新，无论当前事务是否完成**。因此，我们可能应该避免这样做，因为这可能会导致数据不一致。

## 6。结论

在本文中，我们讨论了 JDBC API 中自动提交模式的用途。我们还讨论了如何通过分别打开或关闭事务管理来启用隐式和显式事务管理。最后，我们谈到了使用它时需要考虑的各种问题。

和往常一样，例子的完整源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220926201541/https://github.com/eugenp/tutorials/tree/master/persistence-modules/core-java-persistence-2)