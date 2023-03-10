# 用 JPA 将实体类名映射到 SQL 表名

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jpa-entity-table-names>

## 1.介绍

在这个简短的教程中，我们将学习如何使用 JPA 设置 SQL 表名。

我们将介绍 JPA 如何生成默认名称，以及如何提供自定义名称。

## 2.默认表名

JPA 默认表名的生成是特定于它的实现的。

例如，在 Hibernate 中，默认的表名是第一个字母大写的类名。这是通过`ImplicitNamingStrategy`合同确定的。

但是我们可以通过[实现一个`PhysicalNamingStrategy`接口](/web/20221108213701/https://www.baeldung.com/hibernate-naming-strategy)来改变这种行为。

## 3.使用`@Table`

设置自定义 SQL 表名最简单的方法是用`@` `javax.persistence.Table`标注实体，并定义其名称参数:

```java
@Entity
@Table(name = "ARTICLES")
public class Article {
    // ...
}
```

我们还可以将表名存储在静态最终变量中:

```java
@Entity
@Table(name = Article.TABLE_NAME)
public class Article {
    public static final String TABLE_NAME= "ARTICLES";
    // ...
}
```

## 4.在 JPQL 查询中覆盖表名

默认情况下，在 JPQL 查询中，我们使用实体类名:

```java
select * from Article
```

但是我们可以通过在`@javax.persistence.Entity`注释中定义 name 参数来改变它:

```java
@Entity(name = "MyArticle")
```

然后我们将 JPQL 查询改为:

```java
select * from MyArticle
```

## 5.结论

在本文中，我们已经了解了 JPA 如何生成默认表名，以及如何使用 JPA 设置 SQL 表名。

像往常一样，所有的源代码都可以在 GitHub 上获得[。](https://web.archive.org/web/20221108213701/https://github.com/eugenp/tutorials/tree/master/persistence-modules/java-jpa-2)