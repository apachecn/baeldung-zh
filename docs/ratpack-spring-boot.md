# Ratpack 与 Spring Boot 集成

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/ratpack-spring-boot>

## 1。概述

之前，我们已经介绍过 [Ratpack](/web/20220628101320/https://www.baeldung.com/ratpack) 及其与 Google Guice 的[集成。](/web/20220628101320/https://www.baeldung.com/ratpack-google-guice)

在这篇简短的文章中，我们将展示 Ratpack 如何与 Spring Boot 集成。

## 2。Maven 依赖关系

在我们继续之前，让我们将下面的依赖项添加到我们的`pom.xml:`中

```java
<dependency>
    <groupId>io.ratpack</groupId>
    <artifactId>ratpack-spring-boot-starter</artifactId>
    <version>1.4.6</version>
    <type>pom</type>
</dependency>
```

`[ratpack-spring-boot-starter](https://web.archive.org/web/20220628101320/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22ratpack-spring-boot-starter%22)` pom 依赖项自动将 [`ratpack-spring-boot`](https://web.archive.org/web/20220628101320/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22ratpack-spring-boot%22) 和 [`spring-boot-starter`](https://web.archive.org/web/20220628101320/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.springframework.boot%22%20AND%20a%3A%22spring-boot-starter%22) 添加到我们的依赖项中。

## 3。将 Ratpack 与 Spring Boot 集成

我们可以在 Spring Boot 嵌入 Ratpack，作为 Tomcat 或 Undertow 提供的 servlet 容器的替代。我们只需要用 [`@EnableRatpack`](https://web.archive.org/web/20220628101320/https://ratpack.io/manual/current/api/ratpack/spring/config/EnableRatpack.html) 注释一个 Spring 配置类，并声明类型`[Action](https://web.archive.org/web/20220628101320/https://ratpack.io/manual/current/api/ratpack/func/Action.html)<[Chain](https://web.archive.org/web/20220628101320/https://ratpack.io/manual/current/api/ratpack/handling/Chain.html)>`的 beans:

```java
@SpringBootApplication
@EnableRatpack
public class EmbedRatpackApp {

    @Autowired 
    private Content content;

    @Autowired 
    private ArticleList list;

    @Bean
    public Action<Chain> home() {
        return chain -> chain.get(ctx -> ctx.render(content.body()));
    }

    public static void main(String[] args) {
        SpringApplication.run(EmbedRatpackApp.class, args);
    }
}
```

对于那些更熟悉 Spring Boot 的人来说，`Action<Chain>`可以充当网络过滤器和/或控制器。

在提供静态文件时，Ratpack 会在@Autowired [链配置器](https://web.archive.org/web/20220628101320/https://github.com/ratpack/ratpack/blob/master/ratpack-spring-boot/src/main/java/ratpack/spring/config/internal/ChainConfigurers.java#L63)中的`/public`和`/static`下自动注册静态资源的处理程序。

然而，这个“魔术”的[当前实现](https://web.archive.org/web/20220628101320/https://github.com/ratpack/ratpack/blob/master/ratpack-spring-boot/src/main/java/ratpack/spring/config/RatpackProperties.java#L202)依赖于我们的项目设置和开发环境。因此，如果我们要在不同的环境中实现静态资源的稳定服务，我们应该显式地指定`baseDir`:

```java
@Bean
public ServerConfig ratpackServerConfig() {
    return ServerConfig
      .builder()
      .findBaseDir("static")
      .build();
}
```

上面的代码假设我们在 classpath 下有一个`static`文件夹。同样，我们将 [`ServerConfig`](https://web.archive.org/web/20220628101320/https://ratpack.io/manual/current/api/ratpack/server/ServerConfig.html) bean 命名为`ratpackServerConfig`，以覆盖在 [`RatpackConfiguration`](https://web.archive.org/web/20220628101320/https://github.com/ratpack/ratpack/blob/master/ratpack-spring-boot/src/main/java/ratpack/spring/config/RatpackConfiguration.java#L70) 中注册的默认 bean。

然后我们可以用 [ratpack-test](https://web.archive.org/web/20220628101320/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22io.ratpack%22%20AND%20a%3A%22ratpack-test%22) 测试我们的应用程序:

```java
MainClassApplicationUnderTest appUnderTest
  = new MainClassApplicationUnderTest(EmbedRatpackApp.class);

@Test
public void whenSayHello_thenGotWelcomeMessage() {
    assertEquals("hello baeldung!", appUnderTest
      .getHttpClient()
      .getText("/hello"));
}

@Test
public void whenRequestList_thenGotArticles()  {
    assertEquals(3, appUnderTest
      .getHttpClient()
      .getText("/list")
      .split(",").length);
}

@Test
public void whenRequestStaticResource_thenGotStaticContent() {
    assertThat(appUnderTest
      .getHttpClient()
      .getText("/"), containsString("page is static"));
}
```

## 4。集成 Spring Boot 和 Ratpack

首先，我们将在一个 Spring 配置类中注册所需的 beans:

```java
@Configuration
public class Config {

    @Bean
    public Content content() {
        return () -> "hello baeldung!";
    }
}
```

然后我们可以使用`ratpack-spring-boot`提供的便利方法 [`spring(…)`](https://web.archive.org/web/20220628101320/https://ratpack.io/manual/current/api/ratpack/spring/Spring.html#spring-java.lang.Class-java.lang.String...-) 轻松创建一个注册表:

```java
public class EmbedSpringBootApp {

    public static void main(String[] args) throws Exception {
        RatpackServer.start(server -> server
          .registry(spring(Config.class))
          .handlers(chain -> chain.get(ctx -> ctx.render(ctx
            .get(Content.class)
            .body()))));
    }
}
```

在 Ratpack 中，注册表用于请求处理中的处理器间通信。我们在处理程序中使用的 [`Context`](https://web.archive.org/web/20220628101320/https://ratpack.io/manual/current/api/ratpack/handling/Context.html) 对象实现了 [`Registry`](https://web.archive.org/web/20220628101320/https://ratpack.io/manual/current/api/ratpack/registry/Registry.html) 接口。

这里将 Spring Boot 提供的一个 [`ListableBeanFactory`](https://web.archive.org/web/20220628101320/https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/beans/factory/ListableBeanFactory.html) 实例改编成一个`Registry`，以备份`Handler`的`Context`中的注册表。因此，当我们想从`Context`获得一个特定类型的对象时，`ListableBeanFactory`将被用来查找匹配的 beans。

## 5。总结

在本教程中，我们探讨了如何集成 Spring Boot 和 Ratpack。

和往常一样，Github 上的[提供了完整的实现。](https://web.archive.org/web/20220628101320/https://github.com/eugenp/tutorials/tree/master/ratpack)