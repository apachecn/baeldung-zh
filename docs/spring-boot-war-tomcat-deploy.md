# 将 Spring Boot 战争部署到 Tomcat 服务器中

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-war-tomcat-deploy>

## 1.介绍

[Spring Boot](https://web.archive.org/web/20221011052655/https://projects.spring.io/spring-boot/) 是一个配置框架的[约定，它允许我们创建一个 Spring 项目的生产就绪设置，](https://web.archive.org/web/20221011052655/https://en.wikipedia.org/wiki/Convention_over_configuration) [Tomcat](https://web.archive.org/web/20221011052655/https://tomcat.apache.org/) 是最流行的 Java Servlet 容器之一。

默认情况下，Spring Boot 构建了一个独立的 Java 应用程序，可以作为桌面应用程序运行，也可以配置为系统服务，但是在某些环境下，我们无法安装新的服务或手动运行应用程序。

与独立的应用程序不同，Tomcat 是作为一个服务安装的，可以管理同一个应用程序进程中的多个应用程序，避免了为每个应用程序进行特定设置的需要。

在本教程中，我们将创建一个简单的 Spring Boot 应用程序，并使其适应 Tomcat。

## 2.设置 Spring Boot 应用程序

让我们使用一个可用的初学者模板来设置一个简单的 Spring Boot web 应用程序:

```java
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId> 
    <version>2.4.0</version> 
    <relativePath/> 
</parent> 
<dependencies>
    <dependency> 
        <groupId>org.springframework.boot</groupId> 
        <artifactId>spring-boot-starter-web</artifactId> 
    </dependency> 
</dependencies>
```

**除了标准`@SpringBootApplication,`之外，不需要额外的配置**，因为 Spring Boot 会处理默认设置。

然后我们将添加一个简单的 REST 端点来为我们返回一些有效的内容:

```java
@RestController
public class TomcatController {

    @GetMapping("/hello")
    public Collection<String> sayHello() {
        return IntStream.range(0, 10)
          .mapToObj(i -> "Hello number " + i)
          .collect(Collectors.toList());
    }
}
```

最后，我们将使用`mvn spring-boot:run,`执行应用程序，并在`http://localhost:8080/hello`启动浏览器来检查结果。

## 3.制造一场 Spring Boot 战争

Servlet 容器期望应用程序满足一些要部署的契约。对于 Tomcat 来说，契约就是 [Servlet API 3.0](https://web.archive.org/web/20221011052655/https://tomcat.apache.org/tomcat-8.0-doc/servletapi/index.html) 。

为了让我们的应用程序满足这个契约，我们必须在源代码中执行一些小的修改。

首先，我们需要打包一个 WAR 应用程序，而不是一个 JAR。为此，我们将用以下内容更改`pom.xml`:

```java
<packaging>war</packaging>
```

接下来，我们将修改最终的`WAR`文件名，以避免包含版本号:

```java
<build>
    <finalName>${artifactId}</finalName>
    ... 
</build>
```

然后我们将添加 Tomcat 依赖项:

```java
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-tomcat</artifactId>
   <scope>provided</scope>
</dependency>
```

最后，我们将通过实现`SpringBootServletInitializer`接口`:`来初始化 Tomcat 所需的 Servlet 上下文

```java
@SpringBootApplication
public class SpringBootTomcatApplication extends SpringBootServletInitializer {
}
```

为了构建我们的 Tomcat 可部署 WAR 应用程序，我们将执行`the mvn clean package.`，之后，我们的 WAR 文件在`target/spring-boot-deployment.war`生成(假设 Maven `artifactId`是“spring-boot-deployment”)。

我们应该考虑这个新的设置使得我们的 Spring Boot 应用程序成为一个非独立的应用程序(如果我们想让它再次在独立模式下工作，我们可以从 tomcat 依赖项中移除`provided`作用域)。

## 4.将战争部署到 Tomcat

要在 Tomcat 中部署和运行我们的 WAR 文件，我们需要完成以下步骤:

1.  [下载 Apache Tomcat](https://web.archive.org/web/20221011052655/https://tomcat.apache.org/download-90.cgi) 并解压到一个`tomcat`文件夹中
2.  将我们的 WAR 文件从`target/spring-boot-deployment.war`复制到`tomcat/webapps/`文件夹
3.  从终端，导航到`tomcat/bin`文件夹并执行
    1.  `catalina.bat run` (在 Windows 上)
    2.  `catalina.sh run` (在基于 Unix 的系统上)
4.  转到`http://localhost:8080/spring-boot-deployment/hello`

这是一个快速的 Tomcat 设置，所以请查看关于 Tomcat 安装的指南。还有另外的方法[将 WAR 文件部署到 Tomcat](/web/20221011052655/https://www.baeldung.com/tomcat-deploy-war) 。

## 5.结论

在这篇简短的文章中，我们创建了一个简单的 Spring Boot 应用程序，并将其转换成一个可在 Tomcat 服务器上部署的有效 WAR 应用程序。

与往常一样，GitHub 上的[提供了示例的完整源代码。](https://web.archive.org/web/20221011052655/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-deployment)