# spring Data JPA–在没有数据库的情况下运行应用程序

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-data-jpa-run-app-without-db>

## 1.概观

在本教程中，我们将学习如何在没有运行数据库的情况下启动 Spring Boot 应用程序。

默认情况下，如果我们有一个包含 Spring 数据 JPA 的 Spring Boot 应用程序，那么这个应用程序会自动创建一个数据库连接。但是，在应用程序启动时数据库不可用的情况下，可能需要避免这种情况。

## 2.设置

我们将使用一个简单的使用 MySQL 的 Spring Boot 应用程序。让我们看看设置应用程序的步骤。

### 2.1.属国

让我们将 [Spring Data JPA starter](https://web.archive.org/web/20220728150055/https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-data-jpa) 和 [MySql Connector](https://web.archive.org/web/20220728150055/https://mvnrepository.com/artifact/mysql/mysql-connector-java) 依赖项添加到`pom.xml`文件中:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>

<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <scope>runtime</scope>
</dependency> 
```

这会将 JPA、MySQL 连接器和 Hibernate 添加到类路径中。

除此之外，当应用程序启动时，我们希望任务保持运行。为此，让我们将 [Web 启动器](https://web.archive.org/web/20220728150055/https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-web)添加到`pom.xml`文件中:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency> 
```

这将在端口 8080 上启动一个 web 服务器，并保持应用程序运行。

### 2.2.性能

**在启动应用程序**之前，我们需要在`application.properties`文件中设置一些强制属性:

```java
spring.datasource.url=jdbc:mysql://localhost:3306/myDb
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL5Dialect 
```

让我们了解一下我们设置的属性:

*   `spring.datasource.url`:服务器的 URL 和数据库的名称。
*   `spring.datasource.driver-class-name`:驱动类名称。MySQL 连接器提供了这个驱动程序。
*   我们将它设置为 MySQL 5。这告诉 JPA 提供者使用 MySQL 5 方言。

**除此之外，我们需要设置连接数据库所需的用户名和密码**:

```java
spring.datasource.username=root
spring.datasource.password=root
```

### 2.3.启动应用程序

如果我们启动应用程序，我们会看到以下错误:

```java
HHH000342: Could not obtain connection to query metadata
com.mysql.cj.jdbc.exceptions.CommunicationsException: Communications link failure
The last packet sent successfully to the server was 0 milliseconds ago. The driver has not received any packets from the server. 
```

这是因为我们没有运行在指定 URL 的数据库服务器。但是，应用程序的默认行为是执行这两个操作:

*   **JPA 尝试连接到数据库服务器并获取元数据**
*   **如果数据库不存在，Hibernate 会尝试创建一个。**这是由于属性`spring.jpa.hibernate.ddl-auto`默认设置为`create`。

## 3.不使用数据库运行

要在没有数据库的情况下继续，我们需要通过覆盖上面提到的两个属性来修复默认行为。

首先，**让我们** **禁用元数据获取**:

```java
spring.jpa.properties.hibernate.temp.use_jdbc_metadata_defaults=false 
```

然后，**我们禁用自动数据库创建**:

```java
spring.jpa.hibernate.ddl-auto=none 
```

通过设置这个属性，**我们已经禁止了数据库的创建。因此，应用程序没有理由创建连接。**

与之前不同，现在，当我们启动应用程序时，它启动时没有任何错误。除非操作需要与数据库交互，否则不会启动连接。

## 4.结论

在本文中，我们学习了如何在不需要运行数据库的情况下启动 Spring Boot 应用程序。

我们研究了应用程序查找数据库连接的默认行为。然后，我们通过覆盖两个属性来修复默认行为。

和往常一样，本文中使用的代码示例可以在 GitHub 上找到[。](https://web.archive.org/web/20220728150055/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-data-3)