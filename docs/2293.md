# 使用 JDBC 将 Null 插入整数列

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jdbc-insert-null-into-integer-column>

## 1.介绍

在本文中，我们将研究如何使用普通的 JDBC**在数据库中存储`null`值。我们将首先描述使用`null`值的原因，然后是几个代码示例。**

## 2.使用`null`值

`null`是超越所有编程语言的关键词。它代表一个特殊的值。**人们普遍认为`null`没有价值，或者说它代表不了什么**。在数据库列中存储一个`null`意味着在硬盘上预留了空间。如果有合适的值可用，我们可以将它存储在那个空间中。

**另一种感知是`null`等于零或者`a` 空白串**。特定上下文中的零或空字符串可能有含义，例如，仓库中的零个项目。此外，我们可以对这两个值执行类似于`sum`或`concat` 的操作。但是那些操作在对付`null`的时候没有任何意义。

使用`null`值来表示我们数据中的特殊情况有很多优点。其中一个优势是，大多数数据库引擎从内部函数(如`sum`或`avg`)中排除了`null`值。另一方面，当`null`在我们的代码中时，我们可以编写特殊的动作来减少缺失的值。

将`null`带上谈判桌也带来了一些不利因素。当编写处理包含`null`值的数据的代码时，我们必须以不同的方式处理这些数据。这可能导致难看的代码、混乱和错误。同样，`null`值在数据库中可以有一个可变长度。存储在`Integer`和`Byte`列中的 n 个`ull`将具有不同的长度。

## 3.履行

对于我们的例子，我们将使用一个简单的 Maven 模块和一个 [H2 内存数据库](/web/20220628144612/https://www.baeldung.com/spring-boot-h2-database)。不需要其他依赖项。

首先，让我们创建名为`Person`的 POJO 类。这个类将有四个字段。`Id`作为我们数据库的主键，`name,`和`lastName,` 是字符串，`age`表示为`Integer`。 **`Age`不是必填字段，可以是`null` :**

```
public class Person {
    private Integer id;
    private String name;
    private String lastName;
    private Integer age;
    //getters and setters
} 
```

为了创建反映这个 Java 类的数据库表，我们将使用下面的 SQL 查询:

```
CREATE TABLE Person (id INTEGER not null, name VARCHAR(50), lastName VARCHAR(50), age INTEGER, PRIMARY KEY (id));
```

所有这些都过去了，现在我们可以专注于我们的主要目标。要在`Integer`列中设置一个`null`值，在`PreparedStatement`接口中有两种定义方式。

### 3.1.使用`setNull`方法

使用`setNull` 、**方法，我们总是在执行 SQL 查询**之前确定我们的字段值是`null`。这允许我们在代码中有更多的灵活性。

有了列索引，我们**还必须向`PreparedStatement`实例提供关于底层列类型**的信息。在我们的例子中，这是`java.sql.Types.INTEGER`。

该方法仅适用于`null`值。对于任何其他的，我们必须使用适当的方法对`PreparedStatement`实例进行说明:

```
@Test
public void givenNewPerson_whenSetNullIsUsed_thenNewRecordIsCreated() throws SQLException {
    Person person = new Person(1, "John", "Doe", null);

    try (PreparedStatement preparedStatement = DBConfig.getConnection().prepareStatement(SQL)) {
        preparedStatement.setInt(1, person.getId());
        preparedStatement.setString(2, person.getName());
        preparedStatement.setString(3, person.getLastName());
        if (person.getAge() == null) {
            preparedStatement.setNull(4, Types.INTEGER);
        }
        else {
            preparedStatement.setInt(4, person.getAge());
        }
        int noOfRows = preparedStatement.executeUpdate();

        assertThat(noOfRows, equalTo(1));
    }
} 
```

在我们不检查`getAge`方法是否返回`null`并用`a null`值调用`setInt`方法的情况下，我们将得到一个`NullPointerException`。

### 3.2.使用`setObject`方法

`setObject` 方法让我们在处理代码中丢失的数据时缺乏灵活性。我们可以传递我们拥有的数据，**底层结构会将 Java `Object`类型映射到 SQL 类型**。

请注意，并非所有数据库都允许在不指定 SQL 类型的情况下传递`null`。例如，JDBC 驱动程序不能从`null.`推断 SQL 类型

为了安全起见，最好将一个 SQL 类型传递给`setObject`方法:

```
@Test
public void givenNewPerson_whenSetObjectIsUsed_thenNewRecordIsCreated() throws SQLException {
    Person person = new Person(2, "John", "Doe", null);

    try (PreparedStatement preparedStatement = DBConfig.getConnection().prepareStatement(SQL)) {
        preparedStatement.setInt(1, person.getId());
        preparedStatement.setString(2, person.getName());
        preparedStatement.setString(3, person.getLastName());
        preparedStatement.setObject(4, person.getAge(), Types.INTEGER);
        int noOfRows = preparedStatement.executeUpdate();

        assertThat(noOfRows, equalTo(1));
    }
}
```

## 4.结论

在本教程中，我们解释了`null`值在数据库中的一些基本用法。然后我们提供了如何用普通 JDBC 存储`Integer`列中的`null`值的例子。

和往常一样，所有代码都可以在 GitHub 上找到[。](https://web.archive.org/web/20220628144612/https://github.com/eugenp/tutorials/tree/master/persistence-modules/core-java-persistence-2)