# Liquibase 与 Flyway

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/liquibase-vs-flyway>

## 1.介绍

在这个持续集成和自动重构数据库开发的时代，我们需要[进化数据库设计](https://web.archive.org/web/20220707143806/https://martinfowler.com/articles/evodb.html)的技术。像 [Liquibase](/web/20220707143806/https://www.baeldung.com/liquibase-refactor-schema-of-java-app) 和 [Flyway](/web/20220707143806/https://www.baeldung.com/database-migrations-with-flyway) 这样的工具遵循这些技术，并提供了一种迭代开发方法。在本文中，我们将研究 Liquibase 和 Flyway 之间的差异和相似之处。

请注意，没有一种工具对所有用例都是完美的。每种工具都有自己的优势。

## 2.Liquibase 和 Flyway 之间的相似之处

由于 Liquibase 和 Flyway 实现了进化数据库的设计原则，所以它们提供了许多相似的功能。两种工具:

*   在一定程度上是开源的，有助于管理、跟踪和部署数据库模式变化。
*   对数据库模式更改使用版本化迁移方法。
*   基于 Java，并为 Java 框架提供了广泛的支持，如 [Spring Boot](/web/20220707143806/https://www.baeldung.com/spring-boot) 和[垂直 x](/web/20220707143806/https://www.baeldung.com/vertx) 。
*   支持与 Maven 和 Gradle 等构建工具的集成。
*   可以通过提供的脚本从命令行独立运行。
*   支持各种各样的数据库。

现在我们将讨论这些工具产品之间的差异。

## 3.Liquibase 和 Flyway 之间的差异

### 3.1.定义变更

Flyway 使用 SQL 来定义变更。另一方面，Liquibase 可以灵活地指定不同格式的变更，包括 XML、YAML 和 JSON 等 SQL。使用 Liquibase，我们可以使用与数据库无关的语言，并轻松地将模式更改应用于不同的数据库类型。

**Flyway 是围绕一个线性数据库版本系统**构建的，该系统会随着每次版本变更而增加。这有时会与并行开发产生冲突。Flyway 脚本的文件名定义了[迁移](https://web.archive.org/web/20220707143806/https://flywaydb.org/documentation/concepts/migrations.html)的类型。例如，迁移应该有一个前缀约定，如`V`(版本化)、U(撤销)和 R(可重复)。它后面是版本号和分隔符 __(两个下划线)，后面是描述和后缀`.sql`，比如`V01__Add_New_Column.sql`。

Liquibase 迁移不需要遵循任何文件名约定。**在 Liquibase 中，变更由一个称为主 changelog** 的分类帐管理，它将被定义为包括所有的迁移。

### 3.2.存储更改

**这两个工具都将部署的变更存储在一个表中。** Flyway 迁移存储在数据库模式中，默认表名为`flyway_schema_history`。类似地，Liquibase 将其部署的迁移存储在一个名为`databasechangelog`的表中。这两个工具都支持覆盖默认配置来更改表名。

### 3.3.变更的执行顺序

在 Flyway 中管理变更的顺序相对困难。使用 Flyway，顺序取决于文件名中的版本号和迁移类型。相反，Liquibase 使用一个名为`master_changelog`的单独文件，其中的更改按照定义的顺序进行部署。

### 3.4.回滚更改

现在，让我们讨论数据库迁移的一个主要方面。每当一个坏的更改在应用程序中引起灾难性的问题时，就需要回滚。 Liquibase 提供了一种回滚一切或撤销特定迁移的方法(仅在付费版本上可用)。

Flyway 还有一个撤销迁移，可以用一个以`U`开头的文件名，后跟需要撤销的版本来部署。但是这个功能只有它的付费版本才有。

这两个工具都提供了不错的回滚功能，但是只考虑免费版本，Flyway 提供了一个好用的解决方案。

### 3.5.变更的选择性部署

在一些用例中，我们只需要将变更部署到一个环境中。当我们必须有选择地部署变更时，Liquibase 在这方面胜出。Flyway 也能够做到这一点，但是您必须为每个环境或数据库设置不同的配置文件。使用 Liquibase，我们可以轻松地添加[标签和上下文](https://web.archive.org/web/20220707143806/https://www.liquibase.com/blog/contexts-vs-labels?_ga=2.161326136.488045199.1647282742-629483959.1647282742)以确保在特定位置的部署。

### 3.6.基于 Java 的迁移

虽然这两个工具都是非常面向 Java 的，但是只有 Flyway 提供基于 Java 的迁移。Flyway 允许在 Java 文件中定义迁移。这在某些情况下可能是有利的。

### 3.7.快照和比较数据库

**Liquibase 允许用户对数据库的当前状态**进行快照。我们可以用这个状态与另一个数据库进行比较。这在故障转移和数据库复制等场景中非常有用。另一方面，Flyway 不支持任何快照功能。

### 3.8.有条件部署

**Liquibase 提供了一个叫做前置条件的附加功能。**前提条件允许用户根据数据库的当前状态应用更改。变更集只有在通过这些先决条件后才会执行。

另一方面，Flyway 不支持这一点。但是通过过程，我们可以在大多数基于 SQL 的数据库中应用条件。

## 4.结论

在本文中，我们比较了两个最常用的数据库迁移工具 Liquibase 和 Flyway。在选择工具时，总是要做出权衡。这两种工具相互比较时，没有明显的缺点或优点。您可以根据您的应用需求选择 Liquibase 或 Flyway，并选中大多数复选框。