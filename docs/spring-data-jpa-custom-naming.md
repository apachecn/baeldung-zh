# Spring 数据 JPA 的自定义命名约定

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-data-jpa-custom-naming>

## 1.介绍

Spring Data JPA 提供了许多在应用程序中使用 JPA 的特性。在这些特性中，有 DDL 和 DML 查询中表和列名的标准化。

在这个简短的教程中，我们将了解如何配置这个默认的命名约定。

## 2.默认命名约定

首先，我们来看看 Spring 关于表名和列名的默认命名约定。

假设我们有一个`Person `实体:

```java
@Entity
public class Person {
    @Id
    private Long id;
    private String firstName;
    private String lastName;
}
```

我们这里有几个名字必须映射到数据库。嗯， **Spring 默认使用小写字母**，这意味着它只使用小写字母并用下划线分隔单词。因此，`Person` 实体的表创建查询应该是:

```java
create table person (id bigint not null, first_name varchar(255), last_name varchar(255), primary key (id));
```

返回所有名字的选择查询将是:

```java
select first_name from person;
```

为此， **Spring 实现了其版本的 [Hibernate 的`PhysicalNamingStrategy`T4:](/web/20220625230628/https://www.baeldung.com/hibernate-naming-strategy)`[SpringPhysicalNamingStrategy](https://web.archive.org/web/20220625230628/https://github.com/spring-projects/spring-boot/blob/main/spring-boot-project/spring-boot/src/main/java/org/springframework/boot/orm/jpa/hibernate/SpringImplicitNamingStrategy.java)`。**

## 3.RDMS 区分大小写

在详细介绍如何创建我们的自定义命名约定之前，让我们先谈一谈 RDMS 是如何管理标识符大小写的。

有两种情况需要考虑:RDMS 区分大小写，或者不区分大小写。

在第一种情况下，**RDMS 将严格匹配具有相同大小写**的标识符。因此，在我们的示例中，以下查询将有效:

```java
select first_name from person;
```

而这个会抛出一个错误，甚至不返回结果:

```java
select FIRST_NAME from PERSON;
```

另一方面，**对于不区分大小写的 RDMS，这两个查询都可以。**

我们会做些什么来迫使 RDMS 匹配他们案件的标识符呢？我们可以使用带引号的标识符(例如，“person”)。

通过在我们的标识符周围使用引号，我们告诉数据库在将这些标识符与表和列名进行比较时，它也应该匹配大小写。因此，仍然使用我们的示例，下面的查询可以工作:

```java
select "first_name" from "person";
```

而这个不会:

```java
select "first_name" from "PERSON";
```

不过这只是理论上的，因为每个 RDMS 都以自己的方式管理引用的标识符，所以[里程数会有所不同](https://web.archive.org/web/20220625230628/https://www.alberton.info/dbms_identifiers_and_case_sensitivity.html)。

## 4.自定义命名约定

现在，让我们实现我们自己的命名约定。

想象一下我们不能使用 Spring lower snake case 策略，但是我们需要使用 upper snake case。然后，我们需要**提供一个`PhysicalNamingStrategy`** 的实现。

由于我们将继续使用 snake case，最快的选择是从`SpringPhysicalNamingStrategy`继承并将标识符转换为大写:

```java
public class UpperCaseNamingStrategy extends SpringPhysicalNamingStrategy {
    @Override
    protected Identifier getIdentifier(String name, boolean quoted, JdbcEnvironment jdbcEnvironment) {
        return new Identifier(name.toUpperCase(), quoted);
    }
}
```

我们只是覆盖了`getIdentifier()`方法，它负责将超类中的标识符转换成小写。在这里，我们用它来转换成大写字母。

一旦我们编写了实现，我们必须注册它，以便 Hibernate 知道如何使用它。使用 Spring，这是通过**在我们的`application.properties`中设置`spring.jpa.hibernate.naming.physical-strategy`属性**来完成的:

```java
spring.jpa.hibernate.naming.physical-strategy=com.baeldung.namingstrategy.UpperCaseNamingStrategy
```

现在，我们的查询使用大写标识符:

```java
create table PERSON (ID bigint not null, FIRST_NAME varchar(255), LAST_NAME varchar(255), primary key (ID));

select FIRST_NAME from PERSON;
```

假设我们想使用带引号的标识符，这样 RDMS 就必须匹配大小写。然后我们不得不**使用`true`作为`Identifier` `()`构造函数**的`quoted` 参数:

```java
@Override
protected Identifier getIdentifier(String name, boolean quoted, JdbcEnvironment jdbcEnvironment) {
    return new Identifier(name.toUpperCase(), true);
}
```

在我们的查询将呈现带引号的标识符之后:

```java
create table "PERSON" ("ID" bigint not null, "FIRST_NAME" varchar(255), "LAST_NAME" varchar(255), primary key ("ID"));

select "FIRST_NAME" from "PERSON";
```

## 5.结论

在这篇短文中，我们讨论了使用 Spring Data JPA 实现定制命名策略的可能性，以及 RDMS 将如何处理关于其内部配置的 DDL 和 DML 语句。

像往常一样，这篇文章的完整代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220625230628/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-data-jpa-enterprise)