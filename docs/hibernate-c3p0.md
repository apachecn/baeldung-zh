# 将 c3p0 与 Hibernate 配合使用

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/hibernate-c3p0>

## 1。概述

建立数据库连接是相当昂贵的。数据库[连接池](/web/20220524025913/https://www.baeldung.com/java-connection-pooling)是降低这种支出的一种行之有效的方法。

在本教程中，我们将讨论如何将 [c3p0](https://web.archive.org/web/20220524025913/https://www.mchange.com/projects/c3p0/) 与 Hibernate 一起使用来共享连接。

## 2。什么是 c3p0？

**c3p0 是** **一个 Java 库，为管理数据库连接**提供了一种便捷的方式。

简而言之，它通过创建一个连接池来实现这一点。它还有效地处理使用后的`Statement` s 和`ResultSet` s 的清理。这种清理是必要的，以确保资源使用得到优化，并且不会发生可避免的死锁。

**该库与各种传统的 JDBC 驱动程序无缝集成。**此外，它还提供了一个层，使基于 DriverManager 的 JDBC 驱动程序适应新的`javax.sql.DataSource `方案。

而且，因为 Hibernate 支持通过 JDBC 连接到数据库，**一起使用 Hibernate 和 c3p0 很简单。**

## 3。用休眠配置 c3p0】

现在让我们看看如何配置现有的 Hibernate 应用程序，以使用 c3p0 作为其数据库连接管理器。

### 3.1。Maven 依赖关系

首先，我们首先需要添加 [`hibernate-c3p0`](https://web.archive.org/web/20220524025913/https://mvnrepository.com/artifact/org.hibernate/hibernate-c3p0) maven 依赖关系:

```java
<dependency>
    <groupId>org.hibernate</groupId>
    <artifactId>hibernate-c3p0</artifactId>
    <version>5.3.6.Final</version>
</dependency>
```

使用 Hibernate 5，只需添加上述依赖关系就足以启用 c3p0。**只要没有指定其他 JDBC 连接池管理器，就会出现这种情况。**

因此，在我们添加了依赖项之后，我们可以运行我们的应用程序并检查日志:

```java
Initializing c3p0-0.9.5.2 [built 08-December-2015 22:06:04 -0800; debug? true; trace: 10]
Initializing c3p0 pool... [[email protected]](/web/20220524025913/https://www.baeldung.com/cdn-cgi/l/email-protection) [ ... default settings ... ]
```

如果正在使用另一个 JDBC 连接池管理器，我们可以强制我们的应用程序使用 c3p0。我们只需要在属性文件中将`provider_class`设置为`C3P0ConnectionProvider`:

```java
hibernate.connection.provider_class=org.hibernate.connection.C3P0ConnectionProvider
```

### 3.2。连接池属性

最终，我们将需要覆盖默认配置。我们可以向`hibernate.cfg.xml`文件添加自定义属性:

```java
<property name="hibernate.c3p0.min_size">5</property>
<property name="hibernate.c3p0.max_size">20</property>
<property name="hibernate.c3p0.acquire_increment">5</property>
<property name="hibernate.c3p0.timeout">1800</property>
```

同样，`hibernate.properties`文件可以包含相同的设置:

```java
hibernate.c3p0.min_size=5
hibernate.c3p0.max_size=20
hibernate.c3p0.acquire_increment=5
hibernate.c3p0.timeout=1800
```

属性指定了它在任何给定时间应该保持的最小连接数。默认情况下，它将保持至少三个连接。此设置还定义了池的初始大小。

属性指定了它在任何给定时间可以保持的最大连接数。**默认情况下，它会保持最多 15 个连接**。

`acquire_increment`属性指定当连接池中的可用连接用完时，它应该尝试获取多少个连接。默认情况下，它将尝试获取三个新连接。

`timeout`属性指定一个未使用的连接在被丢弃之前将被保留的秒数。**默认情况下，池中的连接永远不会过期。**

我们可以通过再次检查日志来验证新的池设置:

```java
Initializing c3p0-0.9.5.2 [built 08-December-2015 22:06:04 -0800; debug? true; trace: 10]
Initializing c3p0 pool... [[email protected]](/web/20220524025913/https://www.baeldung.com/cdn-cgi/l/email-protection) [ ... new settings ... ]
```

这些是基本的连接池属性。此外，其他配置属性可以在[官方指南](https://web.archive.org/web/20220524025913/https://www.mchange.com/projects/c3p0/#configuration_properties)中找到。

## 5。结论

在本文中，我们讨论了如何在 Hibernate 中使用 c3p0。我们已经查看了一些常见的配置属性，并将 c3p0 添加到一个测试应用程序中。

对于大多数环境，我们建议使用连接池管理器，如 c3p0 或 [HikariCP](/web/20220524025913/https://www.baeldung.com/hikaricp) ，而不是传统的 JDBC 驱动程序。

像往常一样，本教程的完整源代码可以在 [GitHub](https://web.archive.org/web/20220524025913/https://github.com/eugenp/tutorials/tree/master/persistence-modules/hibernate5) 上获得。