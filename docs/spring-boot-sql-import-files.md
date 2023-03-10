# 具有多个 SQL 导入文件的 Spring Boot

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-sql-import-files>

## 1。概述

Spring Boot 允许我们将样本数据导入我们的数据库——主要是为集成测试准备数据。开箱即用，有两种可能。**我们可以使用`import.sql` (Hibernate 支持)或者`data.sql` (Spring JDBC 支持)文件来加载数据**。

然而，有时我们想把一个大的 SQL 文件分割成几个小的，例如，为了更好的可读性或者在模块之间共享一些带有 init 数据的文件。

在本教程中，我们将展示如何使用 Hibernate 和 Spring JDBC 来实现这一点。

## 2。休眠支持

我们可以用属性 **`spring.jpa.properties.hibernate.hbm2ddl.import_files`** 定义包含**样本数据的文件进行加载。它可以在测试资源文件夹中的`application.properties`文件中设置。**

在这种情况下，我们希望只为 JUnit 测试加载样本数据。该值必须是要导入的逗号分隔的文件列表:

```java
spring.jpa.properties.hibernate.hbm2ddl.import_files=import_active_users.sql,import_inactive_users.sql
```

该配置将从两个文件加载样本数据:`import_active_users.sql`和`import_inactive_users.sql`。这里重要的是，我们**必须使用前缀`spring.jpa.properties`将值(JPA 配置)** **传递给 EntityManagerFactory** 。

接下来，我们将展示如何利用 Spring JDBC 支持来实现这一点。

## 3。弹簧 JDBC 支持

初始数据和 **Spring JDBC 支持的配置与 Hibernate 非常相似。我们必须使用 spring . SQL . init . data-locations**属性:

```java
spring.sql.init.data-locations=import_active_users.sql,import_inactive_users.sql
```

如上所述设置该值会产生与 Hibernate 支持中相同的结果。然而，这个解决方案的一个显著的**优势是可以使用 ant 风格的模式**来定义价值:

```java
spring.sql.init.data-locations=import_*_users.sql 
```

上面的值告诉 Spring 搜索名称与`import_*_users.sql `模式匹配的所有文件，并导入其中的数据。

该属性是在 Spring Boot 2.5.0 中引入的；**在 Spring Boot 的早期版本中，我们需要使用`spring.datasource.data` 属性。**

## 4。结论

在这篇短文中，我们展示了如何配置一个 Spring Boot 应用程序来从定制的 SQL 文件中加载初始数据。

最后，我们展示了两种可能性——冬眠和春天 JDBC。它们都工作得很好，选择哪一个取决于开发者。

与往常一样，本文中使用的完整代码示例可以在 Github 上的[处获得。](https://web.archive.org/web/20221127164113/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-boot-persistence)