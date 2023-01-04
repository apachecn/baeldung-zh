# 精品店介绍

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/bootique>

## 1。概述

[`Bootique`](https://web.archive.org/web/20221128114922/http://bootique.io/) 是一个非常轻量级的开源`container-less` JVM 框架，旨在构建下一代可伸缩的微服务。它构建在嵌入式 Jetty 服务器之上，完全支持带有`[jax-rs](https://web.archive.org/web/20221128114922/https://github.com/jax-rs)`的`REST`处理程序。

在本文中，我们将展示如何使用`Bootique`构建一个简单的 web 应用程序。

## 2。Maven 依赖关系

让我们开始使用`Bootique`，将下面的依赖项添加到`pom.xml:`中

```java
<dependency>
    <groupId>io.bootique.jersey</groupId>
    <artifactId>bootique-jersey</artifactId>
    <scope>compile</scope>
</dependency>
<dependency>
    <groupId>io.bootique</groupId>
    <artifactId>bootique-test</artifactId>
    <scope>test</scope>
</dependency> 
```

但是，`Bootique`也需要申报几个`BOM (“Bill of Material”)`进口。这就是为什么需要在`pom.xml:`中增加以下`<dependencyManagement>`部分的原因

```java
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>io.bootique.bom</groupId>
            <artifactId>bootique-bom</artifactId>
            <version>0.23</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

最新版本的`Bootique`可以在[中央 Maven 资源库](https://web.archive.org/web/20221128114922/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22io.bootique%22)中获得。

为了构建一个可运行的 jar，`Bootique`依赖于 [maven-shade-plugin](https://web.archive.org/web/20221128114922/https://maven.apache.org/plugins/maven-shade-plugin/) 。这就是为什么我们还需要添加以下配置:

```java
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-shade-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```

## 3。启动应用程序

启动`Bootique`应用程序最简单的方法是从主方法中调用`[Bootique](https://web.archive.org/web/20221128114922/https://github.com/bootique/bootique/blob/master/bootique/src/main/java/io/bootique/Bootique.java)`的`exec()`方法:

```java
public class App {
    public static void main(String[] args) {
        Bootique.app(args)
          .autoLoadModules()
          .exec();
    }
}
```

**然而，这不会启动嵌入式服务器。**一旦运行以上代码，将显示以下日志:

```java
NAME
      com.baeldung.bootique.App

OPTIONS
      -c yaml_location, --config=yaml_location
           Specifies YAML config location, which can be a file path 
           or a URL.

      -h, --help
           Prints this message.

      -H, --help-config
           Prints information about application modules and their 
           configuration options.

      -s, --server
           Starts Jetty server.
```

这些只不过是与`Bootique`预先捆绑在一起的可用程序参数。

名称是不言自明的；因此，要启动服务器，我们需要传递`–s`或`–server`参数，服务器将在`default port 8080`上启动并运行。

## 4。模块

应用程序是由“模块”的集合组成的。用`Bootique`的术语`“A module is a Java library that contains some code”`来说，这意味着它将每个服务都视为一个模块。它使用`[Google Guice](https://web.archive.org/web/20221128114922/https://github.com/google/guice)` 进行依赖注入。

为了了解其工作原理，让我们创建一个接口:

```java
public interface HelloService {
    boolean save();
}
```

现在，我们需要创建一个实现:

```java
public class HelloServiceImpl implements HelloService {

    @Override
    public boolean save() {
        return true;
    }
}
```

有两种方法可以加载模块。第一种是使用`Guice`的`[Module](https://web.archive.org/web/20221128114922/https://google.github.io/guice/api-docs/latest/javadoc/index.html?com/google/inject/Module.html)`接口，另一种是使用`Bootique`的 [`BQModuleProvider`](https://web.archive.org/web/20221128114922/https://github.com/bootique/bootique/blob/master/bootique/src/main/java/io/bootique/BQModuleProvider.java) ，也就是所谓的`auto-loading`。

### 4.1。Guice 模块

这里，我们可以使用`Guice`的`[Module](https://web.archive.org/web/20221128114922/https://google.github.io/guice/api-docs/latest/javadoc/index.html?com/google/inject/Module.html)`接口来绑定实例:

```java
public class ModuleBinder implements Module {

    @Override
    public void configure(Binder binder) {
        binder
          .bind(HelloService.class)
          .to(HelloServiceImpl.class);
    }
}
```

一旦定义了模块，我们需要将这个定制模块映射到`Bootique`实例:

```java
Bootique
  .app(args)
    .module(module)
    .module(ModuleBinder.class)
  .autoLoadModules()
  .exec();
```

### 4.2。`BQModuleProvider`(自动加载)

这里，我们需要做的就是用`BQModuleProvider`定义前面创建的模块绑定器:

```java
public class ModuleProvider implements BQModuleProvider {

    @Override
    public Module module() {
        return new ModuleBinder();
    }
}
```

**这种技术的优点是我们不需要用`Bootique`实例映射任何模块信息。**

我们只需要在`/resources/META-INF/services/io.bootique.BQModuleProvider`中创建一个文件，写下`ModuleProvider`的全名，包括包名，剩下的由`Bootique`负责:

```java
com.baeldung.bootique.module.ModuleProvider
```

现在，我们可以使用`[@Inject](https://web.archive.org/web/20221128114922/https://google.github.io/guice/api-docs/latest/javadoc/index.html?com/google/inject/Inject.html)`注释在运行时使用服务实例:

```java
@Inject
HelloService helloService;
```

这里需要注意的一点是，由于我们使用了`Bootique`自己的 DI 机制，我们不需要使用`Guice [@ImplementedBy](https://web.archive.org/web/20221128114922/https://google.github.io/guice/api-docs/latest/javadoc/index.html?com/google/inject/ImplementedBy.html)`注释来绑定服务实例。

## 5。休息终点

使用 JAX-RS API 创建 REST 端点很简单:

```java
@Path("/")
public class IndexController {

    @GET
    public String index() {
        return "Hello, baeldung!";
    }

    @POST
    public String save() {
        return "Data Saved!";
    }
}
```

为了将端点映射到`Bootique`自己的`Jersey`实例中，我们需要定义一个 [`JerseyModule`](https://web.archive.org/web/20221128114922/https://github.com/bootique/bootique-jersey/blob/master/bootique-jersey/src/main/java/io/bootique/jersey/JerseyModule.java) :

```java
Module module = binder -> JerseyModule
  .extend(binder)
  .addResource(IndexController.class);
```

## 6。配置

我们可以在基于 YAML 的属性文件中提供内置或自定义配置信息。

例如，如果我们想在自定义端口上启动应用程序，并添加默认的 URI 上下文“hello ”,我们可以使用以下 YAML 配置:

```java
jetty:
    context: /hello
    connector:
        port: 10001
```

现在，在启动应用程序时，我们需要在 config 参数中提供这个文件的位置:

```java
--config=/home/baeldung/bootique/config.yml
```

## 7 .**。记录日志**

开箱即用的`Bootique`带有一个 [`bootique-logback`](https://web.archive.org/web/20221128114922/http://bootique.io/docs/0/bootique-logback-docs/) 模块。要使用该模块，我们需要在`pom.xml`中添加以下依赖项:

```java
<dependency>
    <groupId>io.bootique.logback</groupId>
    <artifactId>bootique-logback</artifactId>
</dependency>
```

这个模块带有一个`[BootLogger](https://web.archive.org/web/20221128114922/https://github.com/bootique/bootique/blob/master/bootique/src/main/java/io/bootique/log/BootLogger.java)`接口，我们可以覆盖它来实现自定义日志记录:

```java
Bootique.app(args)
  .module(module)
  .module(ModuleBinder.class)
    .bootLogger( new BootLogger() {
      @Override
      public void trace( Supplier<String> args ) {
          // ...
      }
      @Override
      public void stdout( String args ) {
          // ...
      }
      @Override
      public void stderr( String args, Throwable thw ) {
          // ...
      }
      @Override
      public void stderr( String args ) {
          // ...
      }
}).autoLoadModules().exec();
```

此外，我们可以在`config.yaml`文件中定义日志配置信息:

```java
log:
    level: warn
    appenders:
    - type: file
      logFormat: '%c{20}: %m%n'
      file: /path/to/logging/dir/logger.log
```

## 8。测试

为了测试，`Bootique`带有 [`bootique-test`](https://web.archive.org/web/20221128114922/https://github.com/bootique/bootique/tree/master/bootique-test) 模块。有两种方法可以测试一个`Bootique`应用程序。

第一种方法是`‘foreground'`方法，它使所有的测试用例运行在主测试线程上。

另一个是`‘background'`方法，它使测试用例在一个隔离的线程池上运行。

使用 [`BQTestFactory`](https://web.archive.org/web/20221128114922/https://github.com/bootique/bootique/blob/master/bootique-test/src/main/java/io/bootique/test/junit/BQTestFactory.java) 可以初始化“前台”环境:

```java
@Rule
public BQTestFactory bqTestFactory = new BQTestFactory();
```

使用 [`BQDaemonTestFactory`](https://web.archive.org/web/20221128114922/https://github.com/bootique/bootique/blob/master/bootique-test/src/main/java/io/bootique/test/junit/BQDaemonTestFactory.java) 可以初始化“后台”环境:

```java
@Rule
public BQDaemonTestFactory bqDaemonTestFactory = new BQDaemonTestFactory();
```

一旦环境工厂准备就绪，我们就可以编写简单的测试用例来测试服务:

```java
@Test
public void givenService_expectBoolen() {
    BQRuntime runtime = bqTestFactory
      .app("--server").autoLoadModules()
      .createRuntime();
    HelloService service = runtime.getInstance( HelloService.class );

    assertEquals( true, service.save() );
}
```

## 9。结论

在本文中，我们展示了如何使用`Bootique`的核心模块构建应用程序。还有其他几个`Bootique` 模块可用，如`[bootique-jooq](https://web.archive.org/web/20221128114922/https://github.com/bootique/bootique-jooq)`、[、`bootique-kotlin`、](https://web.archive.org/web/20221128114922/https://github.com/bootique/bootique-kotlin)[、`bootique-job`、](https://web.archive.org/web/20221128114922/https://github.com/bootique/bootique-job)等。可用模块的完整列表可在[此处](https://web.archive.org/web/20221128114922/https://github.com/bootique)获得。

像往常一样，GitHub 上有完整的源代码。