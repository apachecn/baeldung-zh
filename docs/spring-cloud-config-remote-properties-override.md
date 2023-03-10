# 覆盖 Spring Cloud 配置中的远程属性值

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-cloud-config-remote-properties-override>

## 1.概观

[春云配置](/web/20221208210435/https://www.baeldung.com/spring-cloud-configuration)是春云伞项目的一部分。**它通过集中式服务**管理应用配置数据，因此它与您部署的微服务截然分开。Spring Cloud Config 有自己的属性管理存储库，但也集成了 Git、Consul 和 Eureka 等开源项目。

在本文中，我们将看到在 Spring Cloud Config 中覆盖远程属性值的不同方法，Spring 从 2.4 版带来的限制，以及 3.0 版带来的变化。对于本教程，我们将使用 spring-boot 版本 2.7.2。

## 2.创建 Spring 配置服务器

为了将配置文件具体化，让我们使用 Spring Cloud Config 创建我们的 edge 服务。**它被定位为配置文件**的分发服务器。在本文中，我们将使用文件系统存储库。

### 2.1.创建配置文件

application.properties 文件中定义的配置与所有客户端应用程序共享。还可以为应用程序或给定的配置文件定义特定的配置。

让我们从创建配置文件开始，该文件包含提供给客户端应用程序的属性。我们将把我们的客户端应用程序命名为“baeldung”。在一个`/resources/config`文件夹中，让我们创建一个`baeldung.properties`文件。

### 2.2.添加属性

让我们向我们的`baeldung.properties`文件添加一些属性，然后我们将在我们的客户端应用程序中使用这些属性:

```java
hello=Hello Jane Doe!
welcome=Welcome Jane Doe!
```

让我们在一个`resources/config/application.properties`文件中添加一个共享属性，Spring 将在所有客户端之间共享这个属性:

```java
shared-property=This property is shared accross all client applications
```

### 2.3.Spring-Boot 配置服务器应用程序

现在，让我们创建服务于我们的配置的 Spring 应用程序。我们还需要`[spring-cloud-config-server](https://web.archive.org/web/20221208210435/https://search.maven.org/search?q=spring-cloud-config-server)`依赖关系:

```java
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-config-server</artifactId>
</dependency>
```

安装完成后，让我们创建应用程序并启用配置服务器:

```java
@SpringBootApplication
@EnableConfigServer
public class ConfigServer {
    public static void main(String[] args) {
        SpringApplication.run(ConfigServer.class, args);
    }
}
```

让我们在应用程序的`application.properties`文件中添加以下属性，告诉它在端口 8081 上启动并加载之前定义的配置:

```java
server.port=8081
spring.cloud.config.server.native.searchLocations=classpath:/config
```

现在，让我们启动我们的服务器应用程序，激活本机概要文件并允许我们将文件系统用作配置存储库:

```java
mvn spring-boot:run -Drun.profiles=native
```

我们的服务器现在正在运行并为我们的配置提供服务。让我们验证我们的共享属性是否可访问:

```java
$ curl localhost:8081/unknownclient/default
{
  "name": "unknownclient",
  "profiles": [
    "default"
  ],
  "label": null,
  "version": null,
  "state": null,
  "propertySources": [
    {
      "name": "classpath:/config/application.properties",
      "source": {
        "shared-property": "This property is shared accross all client applications"
      }
    }
  ]
} 
```

以及特定于我们的应用的属性:

```java
$ curl localhost:8081/baeldung/default
{
  "name": "baeldung",
  "profiles": [
    "default"
  ],
  "label": null,
  "version": null,
  "state": null,
  "propertySources": [
    {
      "name": "classpath:/config/baeldung.properties",
      "source": {
        "hello": "Hello Jane Doe!",
        "welcome": "Welcome Jane Doe!"
      }
    },
    {
      "name": "classpath:/config/application.properties",
      "source": {
        "shared-property": "This property is shared accross all client applications"
      }
    }
  ]
} 
```

我们向服务器指示我们的应用程序的名称以及所使用的概要文件，`default`。

我们没有禁用`spring.cloud.config.server.accept-empty`属性，该属性默认为 true。如果应用程序未知(`unknownclient)`)，配置服务器还是会返回共享属性。

## 3.客户端应用程序

现在让我们创建一个客户机应用程序，它在启动时加载服务器提供的配置。

### 3.1.项目设置和相关性

让我们添加`[spring-cloud-starter-config](https://web.archive.org/web/20221208210435/https://search.maven.org/search?q=spring-cloud-starter-config)`依赖项来加载配置，添加`[spring-boot-starter-web](https://web.archive.org/web/20221208210435/https://search.maven.org/search?q=spring-boot-starter-web)`来在我们的`pom.xml`中创建控制器:

```java
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-config</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

### 3.2.创建客户端应用程序

接下来，让我们创建从 spring-cloud-config 服务器读取配置的客户端应用程序:

```java
@SpringBootApplication
public class Client {
    public static void main(String[] args) {
        SpringApplication.run(Client.class, args);
    }
}
```

### 3.3.获取配置

让我们修改我们的`application.properties`文件，告诉 spring-boot 从我们的服务器获取它的配置:

```java
spring.cloud.config.name=baeldung
spring.config.import=optional:configserver:http://localhost:8081
```

我们还指定应用程序的名称为“baeldung ”,以利用专用属性。

### 3.4.添加简单的控制器

现在让我们创建一个控制器，负责显示我们的配置的特定属性，以及共享属性:

```java
@RestController
public class HelloController {

    @Value("${hello}")
    private String hello;

    @Value("${welcome}")
    private String welcome;

    @Value("${shared-property}")
    private String shared;

    @GetMapping("hello")
    public String hello() {
        return this.hello;
    }

    @GetMapping("welcome")
    public String welcome() {
        return this.welcome;
    }

    @GetMapping("shared")
    public String shared() {
        return this.shared;
    }
}
```

现在，我们可以导航到这三个 URL 来验证我们的配置是否被考虑在内:

```java
$ curl http://localhost:8080/hello
Hello Jane Doe!
$ curl http://localhost:8080/welcome
Welcome Jane Doe!
$ curl http://localhost:8080/shared
This property is shared accross all client applications
```

## 4.覆盖服务器端的属性

通过修改服务器配置，可以覆盖为给定应用程序定义的属性。

让我们编辑服务器的`resources/application.properties`文件来覆盖`hello`属性:

```java
spring.cloud.config.server.overrides.hello=Hello Jane Doe – application.properties!
```

让我们再次测试对`/hello`控制器的调用，以验证是否考虑了过载:

```java
$ curl http://localhost:8080/hello
Hello Jane Doe - application.properties!
```

可以在`resources/config/application.properties`文件的共享配置级别添加这个重载。**在这种情况下，它将优先于上面定义的**。

## 5.覆盖客户端的属性

自 Spring Boot 版本 2.4 起，不再可能通过客户端应用程序的`application.properties`文件覆盖属性。

### 5.1.使用弹簧轮廓

然而，我们可以使用[弹簧轮廓](/web/20221208210435/https://www.baeldung.com/spring-profiles)。**在配置文件中本地定义的属性比在服务器级为应用程序定义的优先级更高**。

让我们向我们的客户端应用程序添加一个`application-development.properties`配置文件，并覆盖`hello`属性:

```java
hello=Hello local property!
```

现在，让我们通过激活开发配置文件来启动我们的客户端:

```java
mvn spring-boot:run -Drun.profiles=development
```

我们现在可以再次测试我们的控制器`/hello`来验证过载是否正常工作:

```java
$ curl http://localhost:8080/hello
Hello local property!
```

### 5.2.使用占位符

我们可以使用占位符来给属性赋值。**因此，服务器将提供一个默认值，该值可以被客户端定义的属性**覆盖。

让我们从服务器的`resources/application.properties`文件中移除`hello`属性的重载，并修改`config/baeldung.properties`中的重载，以使用占位符:

```java
hello=${app.hello:Hello Jane Doe!}
```

因此，服务器提供了一个默认值，如果客户机声明了一个名为`app.hello`的属性，这个默认值可以被覆盖。

让我们编辑客户端的`resources/application.properties`文件来添加属性:

```java
app.hello=Hello, overriden local property!
```

让我们再次测试我们的控制器 hello，以验证是否正确考虑了过度充电:

```java
$ curl http://localhost:8080/hello
Hello, overriden local property!
```

注意，如果`hello`属性也在概要文件配置中定义，后者将优先。

## 6.传统配置

从 spring-boot 2.4 开始，可以使用“传统配置”。**这允许我们使用 spring-boot**2.4 版本做出更改之前的旧属性管理系统。

### 6.1.加载外部配置

在 2.4 版之前，外部配置的管理是通过引导来确保的。我们将需要`[spring-cloud-starter-bootstrap](https://web.archive.org/web/20221208210435/https://search.maven.org/search?q=spring-cloud-starter-bootstrap)`依赖。让我们将此添加到我们的`pom.xml`:

```java
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-bootstrap</artifactId>
</dependency>
```

接下来，我们将在 resources 文件夹中创建一个`bootstrap.properties`文件，并配置服务器的访问 URL:

```java
spring.cloud.config.name=baeldung
spring.cloud.config.uri=http://localhost:8081
```

让我们在`application.properties`中启用传统配置:

```java
spring.config.use-legacy-processing=true
```

### 6.2.启用覆盖功能

在服务器端，我们需要指出属性重载是可能的。让我们以这种方式修改我们的`baeldung.properties`文件:

```java
spring.cloud.config.overrideNone=true
```

因此，外部属性不会优先于应用程序 JAR 中定义的属性。

### 6.3.覆盖服务器的属性

我们现在可以通过`application.properties`文件覆盖客户端应用程序中的`hello`属性:

```java
hello=localproperty
```

让我们测试对控制器的调用:

```java
$ curl http://localhost:8080/hello
localproperty
```

### 6.4.遗留配置折旧

**自 Spring Boot 版本 3.0** 起，不再可能启用传统配置。在这种情况下，我们应该使用上面提出的其他方法。

## 7.结论

在本文中，**我们已经看到了在 Spring Cloud Config** 中覆盖远程属性值的不同方法。

可以覆盖服务器上为给定应用程序定义的属性。**也可以使用配置文件或占位符**在客户端级别覆盖属性。我们还研究了如何激活遗留配置以返回到旧的属性管理系统。

和往常一样，源代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20221208210435/https://github.com/eugenp/tutorials/tree/master/spring-cloud-modules/spring-cloud-config)