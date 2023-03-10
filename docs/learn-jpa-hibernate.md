# 学习 JPA 和 Hibernate

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/learn-jpa-hibernate>

对象关系映射(ORM)是将 Java 对象转换成数据库表的过程。换句话说，这允许我们在没有任何 SQL 的情况下与关系数据库进行交互。Java 持久化 API (JPA)是一种规范，它定义了如何在 Java 应用程序中持久化数据。JPA 的主要焦点是 ORM 层。

Hibernate 是当今最流行的 Java ORM 框架之一。它的第一次发布是在大约二十年前，现在仍然有很好的社区支持和定期发布。此外， **Hibernate 是 JPA 规范**的标准实现，具有一些 Hibernate 特有的附加特性。让我们来看看 JPA 和 Hibernate 的一些核心特性。

![series creational patterns - icon](img/2d9f6e42e1c46d100856c201a5dcbad7.png)

## 定义实体

*   [*定义 JPA 实体*](/web/20220628083227/https://www.baeldung.com/jpa-entities)
**   [*冬眠实体生命周期*](/web/20220628083227/https://www.baeldung.com/hibernate-entity-lifecycle)**   [*JPA 实体生命周期事件*](/web/20220628083227/https://www.baeldung.com/jpa-entity-lifecycle-events)**   [*默认 JPA 中的列值*](/web/20220628083227/https://www.baeldung.com/jpa-default-column-values)**   [*JPA @基本标注*](/web/20220628083227/https://www.baeldung.com/jpa-basic-annotation)**   [*用 JPA*](/web/20220628083227/https://www.baeldung.com/jpa-entity-table-names) *将实体类名映射到 SQL 表名***   [*@ Size、@Length、@Column(length=value)*](/web/20220628083227/https://www.baeldung.com/jpa-size-length-column-differences)**   [*JPA 实体相等*](/web/20220628083227/https://www.baeldung.com/jpa-entity-equality)**   [*JPA @嵌入式和@可嵌入*](/web/20220628083227/https://www.baeldung.com/jpa-embedded-embeddable)**   [*JPA 属性转换器*](/web/20220628083227/https://www.baeldung.com/jpa-attribute-converters)**   [*Hibernate @ NotNull vs @ Column(nullable = false)*](/web/20220628083227/https://www.baeldung.com/hibernate-notnull-vs-nullable)**   [*在 JPA 中定义唯一约束*](/web/20220628083227/https://www.baeldung.com/jpa-unique-constraints)**   [*JPA 实体和可序列化接口*](/web/20220628083227/https://www.baeldung.com/jpa-entities-serializable)************