# java.sql.Connection 是线程安全的吗？

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-sql-connection-thread-safety>

## 1.概观

当我们从事多线程项目时，我们知道**如果多个线程共享没有考虑到[线程安全](/web/20221227104853/https://www.baeldung.com/java-thread-safety)而实现的对象，线程可能会表现得出乎意料**。

我们中的许多人可能都遭受过线程安全问题的困扰。所以，问题是，“这个类是线程安全的吗？”经常会想到。

Java 应用程序通过 JDBC 访问关系数据库并利用多线程是很常见的。在这个快速教程中，我们将讨论`java.sql.Connection`是否是线程安全的。

## 2.`java.sql.Connection`界面

当我们从应用程序通过 JDBC 访问数据库时，我们将直接或间接地使用 [`java.sql.Connection`](https://web.archive.org/web/20221227104853/https://docs.oracle.com/en/java/javase/11/docs/api/java.sql/java/sql/Connection.html) 对象。我们依靠这些连接对象来执行数据库操作。因此，`java.sql.Connection`在 JDBC 是一个相当重要的类型。

多线程需要同时与数据库对话也是一种常见的情况。因此，我们经常听到这样的问题，“`java.sql.Connection`是线程安全的吗？”

在接下来的几节中，我们将进一步研究这个问题。此外，我们将讨论在多个线程中使用`java.sql.Connection`对象的适当方法，以便多个线程可以同时访问数据库。

## 3.螺纹安全和`java.sql.Connection`

首先，我们快速说一下[线程安全](/web/20221227104853/https://www.baeldung.com/java-thread-safety)。**线程安全是一种编程方法。也就是说，这是一个与实现相关的概念。**因此，我们可以使用不同的技术来实现线程安全——例如，无状态实现、不可变实现等等。

现在，我们来看看`java.sql.Connection`。首先，它是一个接口，不包含任何实现。因此，**如果我们笼统地问:“`java.sql.Connection `是线程安全的吗？”**我们必须检查实现这个接口的类，以决定实现是否是线程安全的。

嗯，我马上想到了几个问题:哪些类实现了这个接口？它们是线程安全的吗？

通常，我们不会在应用程序代码中实现`java.sql.Connection`接口。 **JDBC 驱动程序将实现这个接口**，这样我们就可以连接到特定的数据库，比如 SQL Server 或 Oracle。

因此，`Connection`实现的线程安全完全依赖于 JDBC 驱动程序。

接下来，我们将探索几个数据库 JDBC 驱动程序作为例子。

## 4.`java.sql.Connection`实现示例

Microsoft SQL Server 和 Oracle 数据库是两种广泛使用的关系数据库产品。

在这一节中，我们将看看这两个数据库的 JDBC 驱动程序，并讨论它们的`java.sql.Connection`接口实现是否是线程安全的。

### 4.1.Microsoft SQLServer

微软 SQL Server 驱动程序类 [`SQLServerConnection`](https://web.archive.org/web/20221227104853/https://www.javadoc.io/doc/com.microsoft.sqlserver/mssql-jdbc/latest/com.microsoft.sqlserver.jdbc/com/microsoft/sqlserver/jdbc/SQLServerConnection.html) 实现了`java.sql.Connection`接口，根据其 Javadoc:

> `SQLServerConnection`不是线程安全的，但是从单个连接创建的多个语句可以在并发线程中同时处理。

所以，这意味着**我们不应该在线程间共享一个`SQLServerConnection`对象，但是我们可以共享从同一个`SQLServerConnection`对象**创建的语句。

接下来，我们来看看另一个知名的数据库产品，Oracle 数据库。

### 4.2.Oracle 数据库

官方的 Oracle JDBC 驱动程序以线程安全的方式实现了`java.sql.Connection`接口。

Oracle 在其[官方文件](https://web.archive.org/web/20221227104853/https://docs.oracle.com/cd/B19306_01/java.102/b14355/apxtips.htm#i1005436)中陈述了其`Connection`实现的线程安全:

> Oracle JDBC 驱动程序为使用 Java 多线程的应用程序提供全面支持，并针对这些应用程序进行了高度优化…
> 
> 但是，Oracle 强烈反对在多个线程之间共享一个数据库连接。避免允许多个线程同时访问一个连接…

根据上面的描述，我们可以说 Oracle 的连接实现是线程安全的。然而，**在多个线程之间共享一个连接对象是“强烈不鼓励的”**。

因此，从 SQL Server 和 Oracle 的例子中，我们知道我们不能假设`java.sql.Connection`实现是线程安全的。那么，我们可能会问，如果我们希望多个线程同时访问一个数据库，什么是合适的方法？让我们在下一节中解决这个问题。

## 5.使用连接池

当我们从应用程序访问数据库时，我们首先需要建立到数据库的连接。这被认为是昂贵的操作。为了提高性能，通常我们会使用一个[连接池](/web/20221227104853/https://www.baeldung.com/java-connection-pooling)。

让我们快速了解一下连接池在多线程场景中是如何工作的。

连接池保存多个连接对象。我们可以配置池的大小。

当多个线程需要并发访问一个数据库时，它们从连接池中请求连接对象。

如果池中仍有空闲连接，线程将获得一个连接对象，并开始其数据库操作。线程完成工作后，它会将连接返回到池中。

如果池中没有空闲连接，线程将等待另一个线程将一个连接对象返回到池中。

因此，**连接池允许多个线程使用不同的连接对象并发访问数据库，而不是共享同一个**。

更进一步，通过这种方式，我们不必关心`Connection`接口的实现是否是线程安全的。

## 6.结论

在本文中，我们讨论了一个常见问题:`java.sql.Connection`是线程安全的吗？

因为`java.sql.Connection`是一个接口，所以不容易预测实现是否是线程安全的。

此外，我们已经指出，如果多个线程需要并发访问数据库，连接池是处理连接的适当方式。