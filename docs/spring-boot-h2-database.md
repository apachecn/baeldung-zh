# Spring Boot 与 H2 数据库

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-h2-database>

## 1。概述

在本教程中，我们将探索使用 H2 与 Spring Boot。就像其他数据库一样，在 Spring Boot 生态系统中有对它的完全内在支持。

## 延伸阅读:

## [内存数据库列表](/web/20221011074419/https://www.baeldung.com/java-in-memory-databases)

A quick review of how to configure some of the more popular in-memory databases for a Java application.[Read more](/web/20221011074419/https://www.baeldung.com/java-in-memory-databases) →

## [Spring Boot 与冬眠](/web/20221011074419/https://www.baeldung.com/spring-boot-hibernate)

A quick, practical intro to integrating Spring Boot and Hibernate/JPA.[Read more](/web/20221011074419/https://www.baeldung.com/spring-boot-hibernate) →

## 2。依赖性

让我们从`[h2](https://web.archive.org/web/20221011074419/https://search.maven.org/search?q=g:com.h2database%20a:h2)`和`[spring-boot-starter-data-jpa](https://web.archive.org/web/20221011074419/https://search.maven.org/search?q=a:spring-boot-starter-data-jpa%20g:org.springframework.boot)` 的依赖关系开始:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <scope>runtime</scope>
</dependency>
```

## 3。数据库配置

默认情况下，Spring Boot 将应用程序配置为**使用用户名`sa`和空密码**连接到内存存储。

但是，我们可以通过向`application.properties`文件添加以下属性来更改这些参数:

```java
spring.datasource.url=jdbc:h2:mem:testdb
spring.datasource.driverClassName=org.h2.Driver
spring.datasource.username=sa
spring.datasource.password=password
spring.jpa.database-platform=org.hibernate.dialect.H2Dialect
```

或者，我们也可以通过向`application.yaml`文件添加相应的属性，将 YAML 用于应用程序的数据库配置:

```java
spring:
  datasource:
    url: jdbc:h2:mem:mydb
    username: sa
    password: password
    driverClassName: org.h2.Driver
  jpa:
    spring.jpa.database-platform: org.hibernate.dialect.H2Dialect
```

根据设计，内存中的数据库是不稳定的，会导致应用程序重启后数据丢失。

我们可以通过使用基于文件的存储来改变这种行为。为此，我们需要更新`spring.datasource.url`属性:

```java
spring.datasource.url=jdbc:h2:file:/data/demo
```

类似地，在`application.yaml`中，我们可以为基于文件的存储添加相同的属性:

```java
spring:
  datasource:
    url: jdbc:h2:file:/data/demo
```

数据库也可以[在其他模式](https://web.archive.org/web/20221011074419/http://www.h2database.com/html/features.html#connection_modes)下运行。

## 4。数据库操作

在 Spring Boot 用 H2 执行 CRUD 操作与用其他 SQL 数据库是一样的，我们在 [Spring Persistence](/web/20221011074419/https://www.baeldung.com/persistence-with-spring-series) 系列中的教程在这方面做得很好。

### 4.1.数据源初始化

我们可以使用基本的 SQL 脚本来初始化数据库。为了演示这一点，让我们在`src/main/resources`目录下添加一个`data.sql` 文件:

```java
INSERT INTO countries (id, name) VALUES (1, 'USA');
INSERT INTO countries (id, name) VALUES (2, 'France');
INSERT INTO countries (id, name) VALUES (3, 'Brazil');
INSERT INTO countries (id, name) VALUES (4, 'Italy');
INSERT INTO countries (id, name) VALUES (5, 'Canada');
```

这里，脚本用一些示例数据填充了我们模式中的`countries`表。

Spring Boot 将自动获取这个文件，并在嵌入式内存数据库中运行它，比如我们配置的 H2 实例。这是一个**为测试或初始化目的播种数据库的好方法**。

我们可以通过将`spring.sql.init.mode`属性设置为`never`来禁用这个默认行为。另外，[也可以配置多个 SQL 文件](/web/20221011074419/https://www.baeldung.com/spring-boot-sql-import-files#spring-jdbc-support)来加载初始数据。

我们关于[加载初始数据](/web/20221011074419/https://www.baeldung.com/spring-boot-data-sql-and-schema-sql)的文章更详细地讨论了这个主题。

### 4.2.休眠和`data.sql`

默认情况下，**`data.sql`脚本在休眠初始化**之前执行。这使得基于脚本的初始化与其他数据库迁移工具保持一致，例如 [Flyway](/web/20221011074419/https://www.baeldung.com/database-migrations-with-flyway) 和 [Liquibase](/web/20221011074419/https://www.baeldung.com/liquibase-refactor-schema-of-java-app) 。因为我们每次都要重新创建 Hibernate 生成的模式，所以我们需要设置一个额外的属性:

```java
spring.jpa.defer-datasource-initialization=true
```

这个**修改默认的 Spring Boot 行为，并在 Hibernate** 生成模式后填充数据。此外，在用`data.sql`填充之前，我们还可以使用`schema.sql`脚本来构建 Hibernate 生成的模式。但是，不建议混合使用不同的模式生成机制。

## 5。访问 H2 控制台

H2 数据库有一个嵌入式 GUI 控制台，用于浏览数据库内容和运行 SQL 查询。默认情况下，Spring 中不启用 H2 控制台。

要启用它，我们需要将以下属性添加到`application.properties`:

```java
spring.h2.console.enabled=true
```

如果我们使用 YAML 配置，我们需要将属性添加到`application.yaml`:

```java
spring:
  h2:
    console.enabled: true
```

然后，在启动应用程序后，我们可以导航到`http://localhost:8080/h2-console`，这将为我们呈现一个登录页面。

在登录页面上，我们将提供在`application.properties`中使用的相同凭证:

[![h2 console - login](img/764e52e95284d0fac98348f4e0a9470c.png)](/web/20221011074419/https://www.baeldung.com/wp-content/uploads/2019/04/Screenshot-2019-04-13-at-5.21.34-PM-e1555173105246-1024x496.png)

连接后，我们将看到一个全面的网页，页面左侧列出了所有表格和一个用于运行 SQL 查询的文本框:

[![h2 console - SQL Statement](img/aaec45e501b25313a4e1859e99a74a20.png)](/web/20221011074419/https://www.baeldung.com/wp-content/uploads/2019/04/Screenshot-2019-04-13-at-5.25.16-PM.png)

web 控制台具有自动完成功能，可以建议 SQL 关键字。控制台是轻量级的，这使得它可以方便地直观检查数据库或直接执行原始 SQL。

此外，我们可以通过在项目的`application.properties`中用我们想要的值指定以下属性来进一步配置控制台:

```java
spring.h2.console.path=/h2-console
spring.h2.console.settings.trace=false
spring.h2.console.settings.web-allow-others=false
```

同样，当使用 YAML 配置时，我们可以将上述属性添加为:

```java
spring:
  h2:
    console.path: /h2-console
    console.settings.trace: false 
    spring.h2.console.settings.web-allow-others: false
```

在上面的代码片段中，我们将控制台路径设置为`/h2-console`，这相对于我们正在运行的应用程序的地址和端口。因此，如果我们的应用程序在`http://localhost:9001`运行，控制台将在`http://localhost:9001/h2-console.`可用

此外，我们将`spring.h2.console.settings.trace`设置为`false`以防止跟踪输出，并且我们还可以通过设置`spring`来禁用远程访问。`h2.console.settings.web-allow-others`到`false`。

## 6。结论

H2 数据库与 Spring Boot 完全兼容。我们已经看到了如何配置它，以及如何使用 H2 控制台来管理我们正在运行的数据库。

完整的源代码可以在 [GitHub](https://web.archive.org/web/20221011074419/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-boot-persistence-h2) 上找到。