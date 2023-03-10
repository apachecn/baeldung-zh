# 使用 SQLite 的 Spring Boot

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-sqlite>

## 1.概观

在这个快速教程中，我们将介绍在支持 JPA 的 Spring Boot 应用程序中使用 [SQLite](https://web.archive.org/web/20220625233741/https://sqlite.org/index.html) 数据库的步骤。

Spring Boot [支持一些众所周知的现成内存数据库](/web/20220625233741/https://www.baeldung.com/java-in-memory-databases)，但是 SQLite 对我们的要求更多。

让我们看看需要什么。

## 2.项目设置

对于我们的示例，**我们将从我们在过去的教程中使用的一个** **[Spring 数据休息应用程序](https://web.archive.org/web/20220625233741/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-data-rest)开始。**

在 pom 中，我们需要添加 [`sqllite-jdbc`](https://web.archive.org/web/20220625233741/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22sqlite-jdbc%22%20AND%20g%3A%22org.xerial%22) 依赖项:

```java
<dependency>
    <groupId>org.xerial</groupId>
    <artifactId>sqlite-jdbc</artifactId>
    <version>3.25.2</version>
</dependency>
```

这种依赖性给了我们使用 [JDBC](https://web.archive.org/web/20220625233741/https://static.javadoc.io/org.xerial/sqlite-jdbc/3.25.2/org/sqlite/JDBC.html) 与 SQLite 通信所需要的东西。但是，**如果我们要使用 ORM，这还不够。**

## 3.SQLite 方言

参见， **Hibernate 不附带一个 [`Dialect`](https://web.archive.org/web/20220625233741/https://docs.jboss.org/hibernate/orm/4.3/javadocs/org/hibernate/dialect/Dialect.html) 用于 SQLite** 。我们需要自己创造一个。

### 3.1.延伸`Dialect`

我们的第一步是扩展`org.hibernate.dialect.Dialect` 类来注册 SQLite 提供的[数据类型](https://web.archive.org/web/20220625233741/https://www.sqlite.org/datatype3.html):

```java
public class SQLiteDialect extends Dialect {

    public SQLiteDialect() {
        registerColumnType(Types.BIT, "integer");
        registerColumnType(Types.TINYINT, "tinyint");
        registerColumnType(Types.SMALLINT, "smallint");
        registerColumnType(Types.INTEGER, "integer");
        // other data types
    }
}
```

有几个，所以一定要查看其余的示例代码。

接下来，我们需要覆盖一些默认的`Dialect`行为。

### 3.2.标识列支持

例如，**我们需要告诉 Hibernate SQLite 如何处理`@Id`列**，这可以通过一个定制的`IdentityColumnSupport`实现来实现:

```java
public class SQLiteIdentityColumnSupport extends IdentityColumnSupportImpl {

    @Override
    public boolean supportsIdentityColumns() {
        return true;
    }

    @Override
    public String getIdentitySelectString(String table, String column, int type) 
      throws MappingException {
        return "select last_insert_rowid()";
    }

    @Override
    public String getIdentityColumnString(int type) throws MappingException {
        return "integer";
    }
}
```

为了简单起见，让我们只将 identity 列的类型保持为`Integer` 。为了获得下一个可用的标识值，我们将指定适当的机制。

然后，我们只需在不断增长的`SQLiteDialect`类中覆盖相应的方法:

```java
@Override
public IdentityColumnSupport getIdentityColumnSupport() {
    return new SQLiteIdentityColumnSupport();
}
```

### 3.3.禁用约束处理

并且， **SQLite 不支持数据库约束，所以我们需要通过再次覆盖主键和外键的适当方法来禁用这些**:

```java
@Override
public boolean hasAlterTable() {
    return false;
}

@Override
public boolean dropConstraints() {
    return false;
}

@Override
public String getDropForeignKeyString() {
    return "";
}

@Override
public String getAddForeignKeyConstraintString(String cn, 
  String[] fk, String t, String[] pk, boolean rpk) {
    return "";
}

@Override
public String getAddPrimaryKeyConstraintString(String constraintName) {
    return "";
}
```

稍后，我们将能够在我们的 Spring Boot 配置中引用这种新的方言。

## 4.`DataSource`配置

此外，由于 **`Spring Boot`没有为 SQLite 数据库提供现成的配置支持**，我们还需要公开我们自己的`DataSource` bean:

```java
@Autowired Environment env;

@Bean
public DataSource dataSource() {
    final DriverManagerDataSource dataSource = new DriverManagerDataSource();
    dataSource.setDriverClassName(env.getProperty("driverClassName"));
    dataSource.setUrl(env.getProperty("url"));
    dataSource.setUsername(env.getProperty("user"));
    dataSource.setPassword(env.getProperty("password"));
    return dataSource;
}
```

最后，我们将在我们的`persistence.properties`文件中配置以下属性:

```java
driverClassName=org.sqlite.JDBC
url=jdbc:sqlite:memory:myDb?cache=shared
username=sa
password=sa
hibernate.dialect=com.baeldung.dialect.SQLiteDialect
hibernate.hbm2ddl.auto=create-drop
hibernate.show_sql=true
```

注意，我们需要将缓存保持为`shared`，以便在多个数据库连接之间保持数据库更新可见。

**因此，使用上述配置，应用程序将启动，并将启动一个名为** **`myDb`** 的内存数据库，剩余的 [Spring Data Rest](/web/20220625233741/https://www.baeldung.com/spring-data-rest-intro) 配置可以占用该数据库。

## 5.结论

在本文中，我们使用了一个示例 Spring Data Rest 应用程序，并将其指向一个 SQLite 数据库。然而，要做到这一点，我们必须创建一个定制的 Hibernate 方言。

请务必在 Github 上查看应用程序[。用`mvn -Dspring.profiles.active=sqlite spring-boot:run `运行，浏览到](https://web.archive.org/web/20220625233741/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-data-rest)`[http://localhost:8080](https://web.archive.org/web/20220625233741/http://localhost:8080/)`即可。