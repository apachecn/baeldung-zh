# 在多个 Spring Boot 应用程序中访问同一内存中的 H2 数据库

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-access-h2-database-multiple-apps>

## 1.概观

在这个快速教程中，我们将演示**如何从多个 Spring Boot 应用**访问同一个内存中的 H2 数据库。

为此，我们将创建两个不同的 Spring Boot 应用程序。第一个 Spring Boot 应用程序将启动内存中的 H2 实例，而第二个应用程序将通过 TCP 访问第一个应用程序的嵌入式 H2 实例。

## 2.背景

众所周知，内存数据库速度更快，并且经常在应用程序中以嵌入式模式使用。但是，内存中的数据库不会在服务器重新启动时保存数据。

要了解更多背景知识，请查看我们关于最常用的内存数据库和自动化测试中内存数据库的[用法](/web/20220524024434/https://www.baeldung.com/spring-jpa-test-in-memory-database)的文章。

## 3.Maven 依赖项

**本文中的两个 Spring Boot 应用程序需要相同的依赖关系:**

```java
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>
    <dependency>
        <groupId>com.h2database</groupId>
        <artifactId>h2</artifactId>
    </dependency>
</dependencies>
```

## 4.设置 H2 数据源

首先，让我们定义最重要的组件——内存中 H2 数据库的 Spring bean 并通过 TCP 端口公开它:

```java
@Bean(initMethod = "start", destroyMethod = "stop")
public Server inMemoryH2DatabaseaServer() throws SQLException {
    return Server.createTcpServer(
      "-tcp", "-tcpAllowOthers", "-tcpPort", "9090");
}
```

由参数`initMethod` 和`destroyMethod`定义的方法被 Spring 调用来启动和停止 H2 数据库。

`-tcp`参数指示 H2 使用 TCP 服务器启动 H2。我们指定了在`createTcpServer`方法的第三和第四个参数中使用的 TCP 端口。

参数`tcpAllowOthers` 为运行在同一主机或远程主机上的外部应用程序的访问打开 H2。

接下来，让我们通过向`application.properties`文件添加一些属性来覆盖由 Spring Boot 的自动配置特性创建的默认数据源:

```java
spring.datasource.url=jdbc:h2:mem:mydb
spring.datasource.driverClassName=org.h2.Driver
spring.datasource.username=sa
spring.datasource.password=
spring.jpa.hibernate.ddl-auto=create
```

覆盖这些属性很重要，因为**我们将需要在想要共享同一个 H2 数据库的其他应用程序**中使用相同的属性和值。

## 5.引导第一个 Spring Boot 应用程序

接下来，为了引导我们的 Spring Boot 应用程序，我们将创建一个带有`@SpringBootApplication `注释的类:

```java
@SpringBootApplication
public class SpringBootApp {
    public static void main(String[] args) {
        SpringApplication.run(SpringBootApp.class, args);
    }
}
```

为了测试一切都连接正确，让我们添加代码来创建一些测试数据。

我们将定义一个名为`initDb`的方法，并用`@PostConstruct `对其进行注释，这样一旦主类初始化，Spring 容器就会自动调用这个方法:

```java
@PostConstruct
private void initDb() {
    String sqlStatements[] = {
      "drop table employees if exists",
      "create table employees(id serial,first_name varchar(255),last_name varchar(255))",
      "insert into employees(first_name, last_name) values('Eugen','Paraschiv')",
      "insert into employees(first_name, last_name) values('Scott','Tiger')"
    };

    Arrays.asList(sqlStatements).forEach(sql -> {
        jdbcTemplate.execute(sql);
    });

    // Query test data and print results
}
```

## 6.第二个 Spring Boot 应用

现在让我们看看客户机应用程序的组件，它需要与上面定义的相同的 Maven 依赖项。

首先，我们将覆盖数据源属性。我们需要确保 JDBC URL 中的端口号与 H2 在第一个应用程序中监听传入连接的端口号相同。

下面是客户端应用程序的`application.properties`文件:

```java
spring.datasource.url=jdbc:h2:tcp://localhost:9090/mem:mydb
spring.datasource.driverClassName=org.h2.Driver
spring.datasource.username=sa
spring.datasource.password=
spring.jpa.hibernate.ddl-auto=create
```

最后，我们创建了客户端 Spring Boot 应用程序的主类。

同样为了简单起见，我们用`@PostConstruct annotation:`定义了一个包含`an initDb`方法的`@SpringBootApplication `

```java
@SpringBootApplication
public class ClientSpringBootApp {
    public static void main(String[] args) {
        SpringApplication.run(ClientSpringBootApp.class, args);
    }

    @PostConstruct
    private void initDb() {
        String sqlStatements[] = {
          "insert into employees(first_name, last_name) values('Donald','Trump')",
          "insert into employees(first_name, last_name) values('Barack','Obama')"
        };

        Arrays.asList(sqlStatements).forEach(sql -> {
            jdbcTemplate.execute(sql);
        });

        // Fetch data using SELECT statement and print results
    } 
}
```

## 7.抽样输出

现在，当我们逐一运行这两个应用程序时，我们可以检查控制台日志并确认第二个应用程序按预期打印了数据。

下面是第一个 Spring Boot 应用程序的**控制台日志:**

```java
****** Creating table: Employees, and Inserting test data ******
drop table employees if exists
create table employees(id serial,first_name varchar(255),last_name varchar(255))
insert into employees(first_name, last_name) values('Eugen','Paraschiv')
insert into employees(first_name, last_name) values('Scott','Tiger')
****** Fetching from table: Employees ******
id:1,first_name:Eugen,last_name:Paraschiv
id:2,first_name:Scott,last_name:Tiger
```

下面是第二个 Spring Boot 应用程序的**控制台日志:**

```java
****** Inserting more test data in the table: Employees ******
insert into employees(first_name, last_name) values('Donald','Trump')
insert into employees(first_name, last_name) values('Barack','Obama')
****** Fetching from table: Employees ******
id:1,first_name:Eugen,last_name:Paraschiv
id:2,first_name:Scott,last_name:Tiger
id:3,first_name:Donald,last_name:Trump
id:4,first_name:Barack,last_name:Obama
```

## 8.结论

在这篇简短的文章中，我们看到了如何从多个 Spring Boot 应用程序中访问同一个内存中的 H2 数据库实例。

与往常一样，GitHub 上的[提供了工作代码示例。](https://web.archive.org/web/20220524024434/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-boot-persistence-h2)