# ActiveJDBC 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/active-jdbc>

## 1.介绍

ActiveJDBC 是一个轻量级 ORM，遵循了 Ruby on Rails 的主要 ORM`ActiveRecord`的核心思想。

它关注于通过移除典型的持久性管理器的额外层来简化与数据库的交互，并且关注于 SQL 的使用，而不是创建一种新的查询语言。

此外，它通过`DBSpec` 类为数据库交互提供了自己的编写单元测试的方法。

让我们看看这个库与其他流行的 Java ORMs 有何不同，以及如何使用它。

## 2.ActiveJDBC 与其他 ORM

与大多数其他 Java ORMs 相比，ActiveJDBC 有着明显的不同。它从数据库中推断出 DB 模式参数，从而消除了将实体映射到底层表的需要。

没有会话，没有持久管理器，不需要学习新的查询语言，没有 getter/setter。就大小和依赖项的数量而言，库本身是轻量级的。

这种实现鼓励使用测试数据库，这些数据库在执行测试后被框架清理，从而降低了维护测试数据库的成本。

然而，每当我们创建或更新模型时，都需要一点额外的[插装](https://web.archive.org/web/20220628051749/http://javalite.io/instrumentation)步骤。我们将在接下来的章节中讨论这一点。

## 3.设计原则

*   从数据库推断元数据
*   基于约定的配置
*   没有会话，没有“连接、重新连接”
*   轻量级模型，简单的 POJOs
*   禁止代理
*   贫血域模型的避免
*   不需要 Dao 和 dto

## 4.建立图书馆

使用 MySQL 数据库的典型 Maven 设置包括:

```java
<dependency>
    <groupId>org.javalite</groupId>
    <artifactId>activejdbc</artifactId>
    <version>1.4.13</version>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>5.1.34</version>
</dependency>
```

最新版本的 [activejdbc](https://web.archive.org/web/20220628051749/https://search.maven.org/classic/#search|gav|1|g%3A%22org.javalite%22%20AND%20a%3A%22activejdbc%22) 和 [mysql connector](https://web.archive.org/web/20220628051749/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22mysql%22%20AND%20a%3A%22mysql-connector-java%22) 工件可以在 Maven Central repository 上找到。

**[插装](https://web.archive.org/web/20220628051749/http://javalite.io/instrumentation)是简化的代价，也是使用 ActiveJDBC 项目时所需要的。**

项目中需要配置一个插装插件:

```java
<plugin>
    <groupId>org.javalite</groupId>
    <artifactId>activejdbc-instrumentation</artifactId>
    <version>1.4.13</version>
    <executions>
        <execution>
            <phase>process-classes</phase>
            <goals>
                <goal>instrument</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

最新的[active JDBC-instrumentation](https://web.archive.org/web/20220628051749/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.javalite%22%20AND%20a%3A%22activejdbc-instrumentation%22)插件也可以在 Maven Central 中找到。

现在，我们可以通过执行以下两个命令之一来处理仪器:

```java
mvn process-classes
mvn activejdbc-instrumentation:instrument
```

## 5.使用 ActiveJDBC

### 5.1.模型

我们可以用一行代码创建一个简单的模型——它涉及到扩展`Model`类。

该库使用英语的[屈折](https://web.archive.org/web/20220628051749/http://javalite.io/english_inflections)来实现名词的复数和单数形式的转换。这可以使用`@Table`注释来覆盖。

让我们看看一个简单的模型是什么样子的:

```java
import org.javalite.activejdbc.Model;

public class Employee extends Model {}
```

### 5.2.连接到数据库

提供了两个类—`Base`和`DB –` 来连接数据库`.`

连接到数据库的最简单方法是:

```java
Base.open("com.mysql.jdbc.Driver", "jdbc:mysql://host/organization", "user", "xxxxx");
```

当模型运行时，它们利用在当前线程中找到的连接。在任何 DB 操作之前，这个连接由`Base`或`DB`类放在本地线程上。

上面的方法允许使用更简洁的 API，不需要像其他 Java ORMs 那样的 DB 会话或持久性管理器。

让我们看看如何使用`DB` 类连接到数据库`:`

```java
new DB("default").open(
  "com.mysql.jdbc.Driver", 
  "jdbc:mysql://localhost/dbname", 
  "root", 
  "XXXXXX");
```

如果我们看看`Base`和`DB`在连接数据库时有什么不同，这有助于我们得出结论:如果在单个数据库上操作，应该使用`Base`,而对于多个数据库应该使用`DB`。

### 5.3.插入记录

向数据库添加记录非常简单。如前所述，不需要 setters 和 getters:

```java
Employee e = new Employee();
e.set("first_name", "Hugo");
e.set("last_name", "Choi");
e.saveIt();
```

或者，我们可以这样添加相同的记录:

```java
Employee employee = new Employee("Hugo","Choi");
employee.saveIt();
```

甚至流利地说:

```java
new Employee()
 .set("first_name", "Hugo", "last_name", "Choi")
 .saveIt();
```

### 5.4.更新记录

下面的代码片段显示了如何更新记录:

```java
Employee employee = Employee.findFirst("first_name = ?", "Hugo");
employee
  .set("last_name","Choi")
  .saveIt();
```

### 5.5.删除记录

```java
Employee e = Employee.findFirst("first_name = ?", "Hugo");
e.delete();
```

如果需要删除所有记录:

```java
Employee.deleteAll();
```

如果我们想从级联到子表的主表中删除一条记录，使用`deleteCascade`:

```java
Employee employee = Employee.findFirst("first_name = ?","Hugo");
employee.deleteCascade();
```

### 5.6.获取记录

让我们从数据库中取出一条记录:

```java
Employee e = Employee.findFirst("first_name = ?", "Hugo");
```

如果我们想获取多条记录，我们可以使用`where`方法:

```java
List<Employee> employees = Employee.where("first_name = ?", "Hugo");
```

## 6.交易支持

在 Java ORMs 中，有一个显式的连接或管理器对象(JPA 中的 EntityManager，Hibernate 中的 SessionManager 等。).ActiveJDBC 中没有这种东西。

调用`Base.open()`打开一个连接，将它附加到当前线程，因此所有模型的所有后续方法都重用这个连接。调用`Base.close()`关闭连接并将其从当前线程中移除。

为了管理事务，有几个方便的调用:

开始交易:

```java
Base.openTransaction();
```

提交交易:

```java
Base.commitTransaction();
```

回滚事务:

```java
Base.rollbackTransaction();
```

## 7.支持的数据库

最新版本支持 SQLServer、MySQL、Oracle、PostgreSQL、H2、SQLite3、DB2 等数据库。

## 8.结论

在这个快速教程中，我们关注并探索了 ActiveJDBC 的基础知识。

和往常一样，与本文相关的源代码可以在 Github 上找到[。](https://web.archive.org/web/20220628051749/https://github.com/eugenp/tutorials/tree/master/persistence-modules/activejdbc)