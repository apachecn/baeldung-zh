# 在 Docker Compose 中使用 PostgreSQL 运行 Spring Boot

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-postgresql-docker>

## 1.介绍

在本教程中，我们希望使用流行的开源数据库 PostgreSQL 运行一个 Spring Boot 应用程序。在之前的[文章](/web/20221126222426/https://www.baeldung.com/docker-compose)、**中，我们研究了 [Docker Compose](/web/20221126222426/https://www.baeldung.com/docker-compose) 一次处理多个容器**。因此，我们将使用 Docker Compose 来运行 Spring Boot 和 PostgreSQL ，而不是将 PostgreSQL 作为一个单独的应用程序来安装。

## 2.创建 Spring Boot 项目

**让我们去 [Spring Initializer](https://web.archive.org/web/20221126222426/https://start.spring.io/) 创建我们的 Spring Boot 项目**。我们将添加`PostgreSQL Driver`和`Spring Data JPA`模块。下载生成的 ZIP 文件并将其解压缩到一个文件夹后，我们就可以运行新的应用程序了:

```java
./mvnw spring-boot:run
```

应用程序失败，因为它无法连接到数据库:

```java
***************************
APPLICATION FAILED TO START
***************************

Description:

Failed to configure a DataSource: 'url' attribute is not specified and no embedded datasource could be configured.

Reason: Failed to determine a suitable driver class 
```

## 3\. Dockerfile

在使用 Docker Compose 启动 PostgreSQL 之前，**我们需要将我们的 Spring Boot 应用程序转换成 Docker 映像**。第一步是将应用程序打包成一个 JAR 文件:

```java
./mvnw clean package -DskipTests
```

在这里，我们首先在打包应用程序之前清理我们以前的构建。此外，我们跳过测试，因为没有 PostgreSQL 测试会失败。

我们现在在`target`目录中有一个应用程序 JAR 文件。该文件的名称中包含项目名称和版本号，并以`-SNAPSHOT.jar`结尾。所以它的名字可能是`docker-spring-boot-postgres-0.0.1-SNAPSHOT.jar`。

让我们创建新的 `src/main/docker`目录。之后，我们将应用程序 JAR 文件复制到那里:

```java
cp target/docker-spring-boot-postgres-0.0.1-SNAPSHOT.jar src/main/docker
```

最后，我们在同一个目录中创建这个`Dockerfile`:

```java
FROM adoptopenjdk:11-jre-hotspot
ARG JAR_FILE=*.jar
COPY ${JAR_FILE} application.jar
ENTRYPOINT ["java", "-jar", "application.jar"]
```

这个文件描述了 Docker 应该如何运行我们的 Spring Boot 应用程序。它使用来自 [AdoptOpenJDK](https://web.archive.org/web/20221126222426/https://adoptopenjdk.net/) 的 Java 11，并将应用程序 JAR 文件复制到`application.jar`。然后，它运行这个 JAR 文件来启动我们的 Spring Boot 应用程序。

## 4.坞站合成文件

现在让我们编写 Docker 编写文件`docker-compose.yml`，并将其保存在`src/main/docker`中:

```java
version: '2'

services:
  app:
    image: 'docker-spring-boot-postgres:latest'
    build:
      context: .
    container_name: app
    depends_on:
      - db
    environment:
      - SPRING_DATASOURCE_URL=jdbc:postgresql://db:5432/compose-postgres
      - SPRING_DATASOURCE_USERNAME=compose-postgres
      - SPRING_DATASOURCE_PASSWORD=compose-postgres
      - SPRING_JPA_HIBERNATE_DDL_AUTO=update

  db:
    image: 'postgres:13.1-alpine'
    container_name: db
    environment:
      - POSTGRES_USER=compose-postgres
      - POSTGRES_PASSWORD=compose-postgres
```

**我们的应用程序的名称是`app.` 它是两个服务中的第一个**(第 4-15 行):

*   Spring Boot 码头工人图像的名称是`docker-spring-boot-postgres:latest`(第 5 行)。Docker 从当前目录中的`Dockerfile`构建映像(第 6-7 行)
*   容器名是`app`(第 8 行)。要看`db`服务(10 号线)。这就是为什么它开始于`db`集装箱之后
*   我们的应用程序使用`db` PostgreSQL 容器作为数据源(第 12 行)。数据库名、用户名和密码都是`compose-postgres`(第 12-14 行)
*   Hibernate 将自动创建或更新任何需要的数据库表(第 15 行)

**PostgreSQL 数据库名为`db`，是第二个服务**(第 17-22 行):

*   我们使用 PostgreSQL 13.1(第 18 行)
*   容器名为`db` (第 19 行)
*   用户名和密码都是`compose-postgres`(第 21-22 行)

## 5.使用 Docker Compose 运行

**让我们用 Docker Compose 运行我们的 Spring Boot 应用程序和 PostgreSQL】:**

```java
docker-compose up
```

首先，这将为我们的 Spring Boot 应用程序构建 Docker 映像。接下来，它将启动一个 PostgreSQL 容器。最后，它将启动我们的应用程序 Docker image。这一次，我们的应用程序运行良好:

```java
Starting DemoApplication v0.0.1-SNAPSHOT using Java 11.0.9 on f94e79a2c9fc with PID 1 (/application.jar started by root in /)
[...]
Finished Spring Data repository scanning in 28 ms. Found 0 JPA repository interfaces.
[...]
Started DemoApplication in 4.751 seconds (JVM running for 6.512)
```

正如我们所看到的，Spring Data 没有找到存储库接口。没错，我们还没有创建！

如果我们想停止所有的容器，我们需要先按[Ctrl-C]。然后我们可以停止 Docker 编写:

```java
docker-compose down
```

## 6.创建客户实体和存储库

为了在我们的应用程序中使用 PostgreSQL 数据库，**我们将创建一个简单的客户实体**:

```java
@Entity
@Table(name = "customer")
public class Customer {

    @Id
    @GeneratedValue
    private long id;

    @Column(name = "first_name", nullable = false)
    private String firstName;

    @Column(name = "last_name", nullable = false)
    private String lastName;
```

`Customer`有一个生成的`id`属性和两个强制属性:`firstName`和`lastName`。

现在，**我们可以为这个实体**编写存储库接口:

```java
public interface CustomerRepository extends JpaRepository<Customer, Long> { }
```

通过简单地扩展`JpaRepository`，我们继承了创建和查询我们的`Customer`实体的方法。

最后，我们将在应用程序中使用这些方法:

```java
@SpringBootApplication
public class DemoApplication {
    @Autowired 
    private CustomerRepository repository; 

    @EventListener(ApplicationReadyEvent.class)
    public void runAfterStartup() {
        List allCustomers = this.repository.findAll(); 
        logger.info("Number of customers: " + allCustomers.size());

        Customer newCustomer = new Customer(); 
        newCustomer.setFirstName("John"); 
        newCustomer.setLastName("Doe"); 
        logger.info("Saving new customer..."); 
        this.repository.save(newCustomer); 

        allCustomers = this.repository.findAll(); 
        logger.info("Number of customers: " + allCustomers.size());
    }
}
```

*   我们通过依赖注入来访问我们的`Customer`库
*   我们查询存储库中现有客户的数量——这将是零
*   然后我们创建并保存一个客户
*   当我们再次查询现有客户时，我们希望找到我们刚刚创建的客户

## 7.再次运行 Docker 编写

**要运行更新后的 Spring Boot 应用程序，我们需要首先重新构建它**。因此，我们在项目根目录中再次执行这些命令:

```java
./mvnw clean package -DskipTests
cp target/docker-spring-boot-postgres-0.0.1-SNAPSHOT.jar src/main/docker
```

我们如何用这个更新的应用程序 JAR 文件重建 Docker 映像？最好的方法是删除现有的 Docker 图像，我们在`docker-compose.yml`中指定了它的名称。这迫使 Docker 在下次启动 Docker 合成文件时重新构建图像:

```java
cd src/main/docker
docker-compose down
docker rmi docker-spring-boot-postgres:latest
docker-compose up
```

因此，在停止我们的容器后，我们删除应用程序 Docker 映像。然后，我们再次启动 Docker 合成文件，这将重新构建应用程序映像。

下面是应用程序的输出:

```java
Finished Spring Data repository scanning in 180 ms. Found 1 JPA repository interfaces.
[...]
Number of customers: 0
Saving new customer...
Number of customers: 1
```

Spring Boot 找到了我们空的客户存储库。因此，我们开始时没有客户，但后来成功地创建了一个。

## 8.结论

在这个简短的教程中，我们从为 PostgreSQL 创建一个 Spring Boot 应用程序开始。接下来，我们编写了一个 Docker 合成文件，用 PostgreSQL 容器运行我们的应用程序容器。

最后，我们创建了一个客户实体和存储库，它允许我们将客户保存到 PostgreSQL。

像往常一样，本教程的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221126222426/https://github.com/eugenp/tutorials/tree/master/docker-modules/docker-spring-boot-postgres)