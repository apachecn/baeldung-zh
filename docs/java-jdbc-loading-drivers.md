# 加载 JDBC 驱动程序

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-jdbc-loading-drivers>

## 1.介绍

JDBC 是一组规范，定义了 Java 数据库连接契约的 API 和 SPI 部分。该标准将 JDBC 驱动程序抽象定义为与数据库交互的主要入口点。

在本教程中，我们将看看加载 JDBC 驱动程序所需的一些基本步骤。

## 2.JDBC 司机

要连接到数据库，我们必须获得一个 JDBC 驱动程序的实例。

我们可以通过指定 JDBC URL 连接字符串，通过`DriverManager`获得它。这样的 URL 包含数据库引擎的类型、数据库名称、主机名和端口，以及特定于数据库供应商的其他连接参数。

**使用连接字符串，我们可以获得一个数据库连接对象，它是与 JDBC 数据库通信的基本单元**:

```java
Connection con = DriverManager.getConnection(
   "jdbc:postgresql://localhost:21500/test?user=fred&password;=secret&ssl;=true"); 
```

如果唯一的指示是指定的 URL，驱动程序管理器如何知道使用哪个驱动程序？

类路径中可能有许多 JDBC 驱动程序，因此必须有一种方法来唯一地区分每个驱动程序。

## 3.传统方法

在 JDBC 版本 4 和 Java SE 1.6 之前，JVM 中没有通用的机制来自动发现和注册服务。因此，需要手动加载名为的 JDBC 驱动程序[类:](/web/20220628114504/https://www.baeldung.com/java-reflection)

```java
Class.forName("oracle.jdbc.driver.OracleDriver");
```

类加载过程触发一个静态初始化例程，该例程向`DriverManager`注册驱动程序实例，并将该类与数据库引擎标识符相关联，如`oracle`或`postgres`。

注册完成后，我们可以在 JDBC URL 中使用这个标识符作为`jdbc:oracle`。

典型的驱动程序注册例程将实例化驱动程序实例，并将其传递给`DriverManager.registerDriver`方法:

```java
public static void register() throws SQLException {
    if (isRegistered()) {
        throw new IllegalStateException("Driver is already registered. It can only be registered once.");
    } else {
        Driver registeredDriver = new Driver();
        DriverManager.registerDriver(registeredDriver);
        Driver.registeredDriver = registeredDriver;
    }
}
```

上面的例子显示了用`DriverManager`注册的 Postgres JDBC 驱动程序。它由 JVM 作为静态初始化器的一部分触发。

**通过设置`jdbc.drivers`系统属性，即使使用传统方法**，也可以部分自动化该步骤:

```java
java -Djdbc.drivers=oracle.jdbc.driver.OracleDriver
```

指定此属性后，驱动程序管理器将自动尝试加载指定的 JDBC 驱动程序。

## 4.JDBC 4 方法

用 Java 1.6 和[服务提供者机制](/web/20220628114504/https://www.baeldung.com/java-spi) 解决了**自动服务发现的问题。它允许服务提供者通过将它们放在包含服务的 JAR 文件中的`META-INF/services`下来声明它们的服务。**

这种机制自动注册驱动程序，因此不再需要手动加载类。然而，即使有了服务提供者，手动加载类也不会导致失败。用最新的 JVM 和 JDBC 4 驱动程序显式调用驱动程序加载是完全合法的。

服务提供者规范只是用声明性的方法代替了手工的类加载。例如，PostgreSQL JDBC 驱动程序在`META-INF/services/`下有一个单独的文件。文件名为`java.sql.Driver`(这是 JDBC 司机的约定俗成的惯例)。它包含 JDBC 驱动程序的完全限定类名，在本例中是`org.postgresql.Driver`。

## 5.结论

在本文中，我们回顾了 JDBC 的基本概念，以及加载 JDBC 驱动程序的各种方法，并对每种方法进行了解释。

和往常一样，这篇文章的完整源代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220628114504/https://github.com/eugenp/tutorials/tree/master/persistence-modules/core-java-persistence)