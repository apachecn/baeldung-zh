# Hibernate 5 命名策略配置

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/hibernate-naming-strategy>

## 1。概述

Hibernate 5 为 Hibernate 实体提供了两种不同的命名策略:隐式命名策略和物理命名策略。

在本教程中，我们将了解如何配置这些命名策略，以便将实体映射到定制的表名和列名。

对于不熟悉 Hibernate 的读者，请务必在这里查看我们的[介绍文章](/web/20220524120435/https://www.baeldung.com/hibernate-save-persist-update-merge-saveorupdate)。

## 2。依赖性

在本教程中，我们将使用[基本 Hibernate 核心依赖关系](https://web.archive.org/web/20220524120435/https://search.maven.org/search?q=g:org.hibernate%20AND%20a:hibernate-core):

```java
<dependency>
    <groupId>org.hibernate</groupId>
    <artifactId>hibernate-core</artifactId>
    <version>5.3.6.Final</version>
</dependency>
```

## 3。隐性命名策略

Hibernate 使用逻辑名将实体或属性名映射到表或列名。这个名字**可以用两种方式**定制:可以通过使用*隐式命名策略*自动导出，也可以通过使用注释*显式定义。*

*ImplicitNamingStrategy* 控制 Hibernate 如何从我们的 Java 类名和属性名中获得一个逻辑名。**我们可以从四种内置策略中选择，或者创建自己的策略。**

对于这个例子，我们将使用默认策略，*ImplicitNamingStrategyJpaCompliantImpl。*使用这种策略，逻辑名将与我们的 Java 类名和属性名相同。

**如果我们想为一个特定的实体偏离这个策略，我们可以使用注释来进行定制**。我们可以使用*@表*注释来定制一个*@实体*的名称。对于一个属性，我们可以使用 *@Column* 注释:

```java
@Entity
@Table(name = "Customers")
public class Customer {

    @Id
    @GeneratedValue
    private Long id;

    private String firstName;

    private String lastName;

    @Column(name = "email")
    private String emailAddress;

    // getters and setters

}
```

使用这种配置，`Customer`实体及其属性的逻辑名称将是:

```java
Customer -> Customers
firstName -> firstName
lastName -> lastName
emailAddress -> email
```

## 4。物理命名策略

既然我们已经配置了逻辑名称，让我们来看看物理名称。

Hibernate 使用物理命名策略将我们的逻辑名称映射到 SQL 表及其列。

**默认情况下，物理名称将与我们在上一节**中指定的逻辑名称**相同。**如果我们想要定制物理名称，我们可以创建一个定制的 *PhysicalNamingStrategy* 类。

例如，我们可能希望在 Java 代码中使用 camel case 名称，但是我们希望在数据库中使用下划线分隔的实际表名和列名。

现在，我们可以结合使用注释和自定义的 *ImplicitNamingStrategy* 来正确映射这些名称，但是 Hibernate 5 提供了 *PhysicalNamingStrategy* 来简化这个过程。它从上一节中获取我们的逻辑名称，并允许我们在一个地方定制它们。

让我们看看这是如何做到的。

首先，我们将创建一个策略，将我们的 camel case 名称转换为使用我们更标准的 SQL 格式:

```java
public class CustomPhysicalNamingStrategy implements PhysicalNamingStrategy {

    @Override
    public Identifier toPhysicalCatalogName(final Identifier identifier, final JdbcEnvironment jdbcEnv) {
        return convertToSnakeCase(identifier);
    }

    @Override
    public Identifier toPhysicalColumnName(final Identifier identifier, final JdbcEnvironment jdbcEnv) {
        return convertToSnakeCase(identifier);
    }

    @Override
    public Identifier toPhysicalSchemaName(final Identifier identifier, final JdbcEnvironment jdbcEnv) {
        return convertToSnakeCase(identifier);
    }

    @Override
    public Identifier toPhysicalSequenceName(final Identifier identifier, final JdbcEnvironment jdbcEnv) {
        return convertToSnakeCase(identifier);
    }

    @Override
    public Identifier toPhysicalTableName(final Identifier identifier, final JdbcEnvironment jdbcEnv) {
        return convertToSnakeCase(identifier);
    }

    private Identifier convertToSnakeCase(final Identifier identifier) {
        final String regex = "([a-z])([A-Z])";
        final String replacement = "$1_$2";
        final String newName = identifier.getText()
          .replaceAll(regex, replacement)
          .toLowerCase();
        return Identifier.toIdentifier(newName);
    }
}
```

最后，我们可以告诉 Hibernate 使用我们的新策略:

```java
hibernate.physical_naming_strategy=com.baeldung.hibernate.namingstrategy.CustomPhysicalNamingStrategy
```

使用我们针对`Customer`实体的新策略，物理名称将是:

```java
Customer -> customers
firstName -> first_name
lastName -> last_name
emailAddress -> email
```

## 5。结论

在这篇简短的文章中，我们学习了隐式命名策略和物理命名策略之间的关系。

我们还看到了如何定制实体的隐式和物理名称及其属性。

你可以在 Github 上查看这个教程[的源代码。](https://web.archive.org/web/20220524120435/https://github.com/eugenp/tutorials/tree/master/persistence-modules/hibernate5)