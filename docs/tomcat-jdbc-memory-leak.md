# Tomcat 警告“为了防止内存泄漏，JDBC 驱动程序已被强制取消注册”

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/tomcat-jdbc-memory-leak>

## 1.概观

在本教程中，我们将看看 Tomcat 警告消息，通知我们，它强行取消注册一个 JDBC 驱动程序。我们将探究该信息的含义、其根本原因以及我们可以做些什么来减轻它。

## 2.信息和意义

该消息的一个版本可能如下:

```java
SEVERE: A web application registered the JBDC driver [oracle.jdbc.driver.OracleDriver]
  but failed to unregister it when the web application was stopped.
  To prevent a memory leak, the JDBC Driver has been forcibly unregistered.
```

通过上面的内容，Tomcat 通知我们，当我们部署 web 应用程序时，JDBC 驱动程序类`OracleDriver`被注册了，但是当同一个应用程序被取消部署时，它没有被取消注册。

我们有多种方法可以[加载和注册一个 JDBC 驱动](/web/20220813183430/https://www.baeldung.com/java-jdbc-loading-drivers)，它本质上是一个扩展了`java.sql.Driver`接口的类。Tomcat 使用 [Java 服务提供者接口](/web/20220813183430/https://www.baeldung.com/java-spi) (SPI)和**自动加载它可以在 web 应用程序的`WEB-INF/lib`目录**下找到的任何 JDBC 4.0 兼容驱动程序类。

当我们取消部署一个 web 应用程序时，我们还必须取消注册它带来的任何驱动程序。否则，它们仍然在 Tomcat 中注册。这造成了内存泄漏，直到我们关闭整个 web 服务器。

从版本 6.0.24 开始，Tomcat 会检测这种类型的泄漏，并强制注销所有泄漏的驱动程序。但是，它仍然通知我们这个问题，如果我们在另一个不支持这个功能的 web 服务器上部署相同的应用程序，这是非常有用的。

## 3.根本原因和潜在问题

该问题的原因在于 JDBC 驱动程序的不正确实现。它应该侦听应用程序取消部署事件并注销自己。

当 Java SPI 加载 JDBC 驱动程序时，它使用当前上下文[类加载器](/web/20220813183430/https://www.baeldung.com/java-classloaders)来加载它。因为驱动程序在应用程序的`WEB-INF/lib`下，SPI 使用它的类加载器加载它。**以这种方式加载的驱动程序向`DriverManager`类注册，这是一个 JVM 单例。如果没有发生这种情况，就会在加载的类中引入内存泄漏。**

当我们取消部署 web 应用程序时，它的类装入器被垃圾收集。**另一方面，`DriverManager`仍然引用 JDBC 驱动程序阻止垃圾收集**。如果我们再次部署同一个 web 应用程序，就会创建一个新的类加载器，SPI 会第二次加载同一个 JDBC 驱动程序。这实际上是一种内存泄漏。

## 4.缓解措施

有多种方法可以缓解这个问题。

### 4.1.使用更新的 Tomcat 版本

从版本 6.0.24 开始，Tomcat 会自动为我们处理这个问题。**这意味着我们可以安全地忽略警告消息**。

### 4.2.关机时手动注销

我们可以在任何应用程序[关机回调](/web/20220813183430/https://www.baeldung.com/spring-shutdown-callbacks)上手动注销驱动程序。在标准情况下，我们的应用程序将加载一个 JDBC 驱动程序，我们可以用一行代码来完成:

```java
DriverManager.deregisterDriver(DriverManager.getDrivers().nextElement());
```

需要注意的是，虽然 Tomcat 调用了取消注册操作，但是`DriverManager`方法被称为取消注册。

### 4.3.移动 JDBC 罐子

官方的处理方式是将 JDBC 驱动程序 jar 文件从应用程序的`WEB-INF/lib`移动到 Tomcat 的/ `lib`目录。由于/ `lib`目录下的所有 jar 都在类路径上，Tomcat 仍然会自动加载驱动程序，但是是在它自己的类加载器下。

当我们部署应用程序时，Tomcat 不会加载任何驱动程序实现，因为在`WEB-INF/lib`下没有任何驱动程序实现。这意味着我们可以安全地取消部署和重新部署它，而无需加载任何新内容，从而防止任何泄漏。

## 5.结论

在本文中，我们讨论了来自 Tomcat 的 JDBC 驱动程序强制注销警告消息的含义。我们还研究了它的根本原因以及修复它的可能方法。