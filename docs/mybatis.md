# MyBatis 快速指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/mybatis>

## 1。简介

MyBatis 是一个开源的持久性框架，它简化了 Java 应用程序中数据库访问的实现。它支持定制 SQL、存储过程和不同类型的映射关系。

简单地说，它是 JDBC 和 Hibernate 的替代品。

## 2。Maven 依赖关系

为了使用 MyBatis，我们需要将依赖项添加到我们的`pom.xml:`

```java
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis</artifactId>
    <version>3.4.4</version>
</dependency>
```

依赖关系的最新版本可以在[这里](https://web.archive.org/web/20220628121808/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22mybatis%22)找到。

## 3。Java API

### 3.1。`SQLSessionFactory`

`SQLSessionFactory`是每个 MyBatis 应用程序的核心类。这个类通过使用`SQLSessionFactoryBuilder'`的`builder()` 方法来实例化，该方法加载一个配置 XML 文件:

```java
String resource = "mybatis-config.xml";
InputStream inputStream Resources.getResourceAsStream(resource);
SQLSessionFactory sqlSessionFactory
  = new SqlSessionFactoryBuilder().build(inputStream);
```

Java 配置文件包括数据源定义、事务管理器细节和定义实体间关系的映射器列表等设置，这些一起用于构建`SQLSessionFactory` 实例:

```java
public static SqlSessionFactory buildqlSessionFactory() {
    DataSource dataSource 
      = new PooledDataSource(DRIVER, URL, USERNAME, PASSWORD);

    Environment environment 
      = new Environment("Development", new JdbcTransactionFactory(), dataSource);

    Configuration configuration = new Configuration(environment);
    configuration.addMapper(PersonMapper.class);
    // ...

    SqlSessionFactoryBuilder builder = new SqlSessionFactoryBuilder();
    return builder.build(configuration);
}
```

### 3.2。`SQLSession`

包含执行数据库操作、获取映射器和管理事务的方法。可以从`SQLSessionFactory`类实例化。这个类的实例不是线程安全的。

执行数据库操作后，应该关闭会话。由于`SqlSession`实现了`AutoCloseable` 接口，我们可以使用`try-with-resources` 块:

```java
try(SqlSession session = sqlSessionFactory.openSession()) {
    // do work
}
```

## 4。地图绘制者

映射器是将方法映射到相应 SQL 语句的 Java 接口。MyBatis 为定义数据库操作提供了注释:

```java
public interface PersonMapper {

    @Insert("Insert into person(name) values (#{name})")
    public Integer save(Person person);

    // ...

    @Select(
      "Select personId, name from Person where personId=#{personId}")
    @Results(value = {
      @Result(property = "personId", column = "personId"),
      @Result(property="name", column = "name"),
      @Result(property = "addresses", javaType = List.class,
        column = "personId", [[email protected]](/web/20220628121808/https://www.baeldung.com/cdn-cgi/l/email-protection)(select = "getAddresses"))
    })
    public Person getPersonById(Integer personId);

    // ...
}
```

## 5.mybatis 注释

让我们来看看 MyBatis 提供的一些主要注释:

*   `**@Insert, @Select, @Update, @Delete** –` 这些注释表示通过调用带注释的方法执行的 SQL 语句:

    ```java
    @Insert("Insert into person(name) values (#{name})")
    public Integer save(Person person);

    @Update("Update Person set name= #{name} where personId=#{personId}")
    public void updatePerson(Person person);

    @Delete("Delete from Person where personId=#{personId}")
    public void deletePersonById(Integer personId);

    @Select("SELECT person.personId, person.name FROM person 
      WHERE person.personId = #{personId}")
    Person getPerson(Integer personId);
    ```

*   **`@Results`** –这是一个结果映射列表，包含数据库列如何映射到 Java 类属性的细节:

    ```java
    @Select("Select personId, name from Person where personId=#{personId}")
    @Results(value = {
      @Result(property = "personId", column = "personId")
        // ...   
    })
    public Person getPersonById(Integer personId);
    ```

*   **`@Result`**–表示从`@Results.` 中检索到的结果列表中的`Result`的单个实例，包括从数据库列到 Java bean 属性的映射、属性的 Java 类型以及与其他 Java 对象的关联:

    ```java
    @Results(value = {
      @Result(property = "personId", column = "personId"),
      @Result(property="name", column = "name"),
      @Result(property = "addresses", javaType =List.class) 
        // ... 
    })
    public Person getPersonById(Integer personId);
    ```

*   `**@Many** –` it specifies a mapping of one object to a collection of the other objects:

    ```java
    @Results(value ={
      @Result(property = "addresses", javaType = List.class, 
        column = "personId",
        [[email protected]](/web/20220628121808/https://www.baeldung.com/cdn-cgi/l/email-protection)(select = "getAddresses"))
    })
    ```

    这里的`getAddresses` 是通过查询地址表返回`Address` 集合的方法。

    ```java
    @Select("select addressId, streetAddress, personId from address 
      where personId=#{personId}")
    public Address getAddresses(Integer personId);
    ```

    类似于`@Many` 注释，我们有`@One` 注释，它指定了对象之间的一对一映射关系。

*   `**@MapKey** –` 用于将记录列表转换为`Map`条记录，记录的关键字由`value`属性定义:

    ```java
    @Select("select * from Person")
    @MapKey("personId")
    Map<Integer, Person> getAllPerson();
    ```

*   `**@Options** –` 该注释指定了要定义的各种开关和配置，因此我们可以`@Options`定义它们:

    ```java
    @Insert("Insert into address (streetAddress, personId) 
      values(#{streetAddress}, #{personId})")
    @Options(useGeneratedKeys = false, flushCache=true)
    public Integer saveAddress(Address address);
    ```

## 6。动态 SQL

动态 SQL 是 MyBatis 提供的一个非常强大的特性。这样，我们就可以精确地构建复杂的 SQL。

使用传统的 JDBC 代码，我们必须编写 SQL 语句，用准确的空格将它们连接起来，并将逗号放在正确的位置。在大型 SQL 语句的情况下，这很容易出错并且很难调试。

让我们探索一下如何在我们的应用程序中使用动态 SQL:

```java
@SelectProvider(type=MyBatisUtil.class, method="getPersonByName")
public Person getPersonByName(String name);
```

这里我们指定了一个类和一个方法名，它实际上构造并生成了最终的 SQL:

```java
public class MyBatisUtil {

    // ...

    public String getPersonByName(String name){
        return new SQL() {{
            SELECT("*");
            FROM("person");
            WHERE("name like #{name} || '%'");
        }}.toString();
    }
}
```

动态 SQL 将所有 SQL 结构作为一个类提供，例如`SELECT`、`WHERE`等。这样，我们就可以动态地改变`WHERE`子句的生成。

## 7。存储过程支持

我们还可以使用`@Select`注释来执行存储过程。这里我们需要传递存储过程的名称、参数列表，并对该过程使用一个显式的`Call`:

```java
@Select(value= "{CALL getPersonByProc(#{personId,
  mode=IN, jdbcType=INTEGER})}")
@Options(statementType = StatementType.CALLABLE)
public Person getPersonByProc(Integer personId);
```

## 8。结论

在这个快速教程中，我们看到了 MyBatis 提供的不同特性，以及它如何简化面向数据库的应用程序的开发。我们也看到了库提供的各种注释。

这篇文章的完整代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220628121808/https://github.com/eugenp/tutorials/tree/master/mybatis)