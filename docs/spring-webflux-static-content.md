# Spring WebFlux 中的静态内容

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-webflux-static-content>

## 1。 **概述**

有时，我们必须在 web 应用程序中提供静态内容。它可能是图像、HTML、CSS 或 JavaScript 文件。

在本教程中，我们将展示如何使用 [Spring WebFlux](/web/20220626201307/https://www.baeldung.com/spring-webflux) 提供静态内容。我们还假设我们的 web 应用程序将使用 [Spring Boot](/web/20220626201307/https://www.baeldung.com/spring-boot-start) 进行配置。

## 2。覆盖默认配置

默认情况下，Spring Boot 从以下位置提供静态内容:

*   `/public`
*   `/static`
*   `/resources`
*   `/META-INF/resources`

来自这些路径的所有文件都在`/[resource-file-name]`路径下提供。

如果我们想改变 Spring WebFlux 的默认路径，我们需要将这个属性添加到我们的`application.properties`文件中:

```java
spring.webflux.static-path-pattern=/assets/**
```

现在，静态资源将位于`/assets/[resource-file-name]`下。

**请注意，当`@EnableWebFlux`注释存在时，此**T3 将不起作用。

## 3。 **路由示例**

还可以使用 WebFlux 路由机制提供静态内容。

让我们看一个服务于`index.html`文件的路由定义的例子:

```java
@Bean
public RouterFunction<ServerResponse> htmlRouter(
  @Value("classpath:/public/index.html") Resource html) {
    return route(GET("/"), request
      -> ok().contentType(MediaType.TEXT_HTML).syncBody(html)
    );
}
```

在`RouterFunction`的帮助下，我们还可以从自定义位置提供静态内容。

让我们看看如何使用`/img/**` 路径提供来自`src/main/resources/img`目录的图像:

```java
@Bean
public RouterFunction<ServerResponse> imgRouter() {
    return RouterFunctions
      .resources("/img/**", new ClassPathResource("img/"));
}
```

## 4。 **自定义 Web 资源路径示例**

另一种服务存储在自定义位置的静态资产的方法，而不是默认的`src/main/resources`路径，是使用 [`maven-resources-plugin`](https://web.archive.org/web/20220626201307/https://search.maven.org/search?q=g:org.apache.maven.plugins%20AND%20a:maven-resources-plugin) 和一个额外的 Spring WebFlux 属性。

首先，让我们将插件添加到我们的`pom.xml`:

```java
<plugin>
    <artifactId>maven-resources-plugin</artifactId>
    <version>3.1.0</version>
    <executions>
        <execution>
            <id>copy-resources</id>
            <phase>validate</phase>
            <goals>
                <goal>copy-resources</goal>
            </goals>
            <configuration>
                <resources>
                    <resource>
                        <directory>src/main/assets</directory>
                        <filtering>true</filtering>
                    </resource>
                </resources>
                <outputDirectory>${basedir}/target/classes/assets</outputDirectory>
            </configuration>
        </execution>
    </executions>
</plugin> 
```

然后，我们只需设置静态位置属性:

```java
spring.resources.static-locations=classpath:/assets/
```

在这些操作之后，`index.html`将在`http://localhost:8080/index.html` URL 下可用。

## 5。 **结论**

在本文中，我们学习了如何在 Spring WebFlux 中提供静态内容。

和往常一样，GitHub 上的[提供了示例代码。](https://web.archive.org/web/20220626201307/https://github.com/eugenp/tutorials/tree/master/spring-5-reactive-modules/spring-5-reactive-2)