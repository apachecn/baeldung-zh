# Spring Boot 开发工具概述

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-devtools>

## 1。简介

Spring Boot 让我们能够快速设置和运行服务。

为了进一步增强开发体验，Spring 发布了 spring-boot-devtools 工具——作为 Spring Boot 1.3 的一部分。本文将尝试介绍使用新功能可以获得的好处。

我们将讨论以下主题:

*   属性默认值
*   自动重启
*   实时重装
*   全局设置
*   远程应用程序

### 1.1。在项目中添加 Spring-Boot-dev tools

在项目中添加`spring-boot-devtools`就像添加任何其他的 spring-boot 模块一样简单。在现有的 spring-boot 项目中，添加以下依赖项:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
</dependency>
```

对项目进行一次干净的构建，现在你就可以和 spring-boot-devtools 集成在一起了。最新版本可以从[这里](https://web.archive.org/web/20220626082835/https://search.maven.org/classic/#artifactdetails%7Corg.springframework.boot%7Cspring-boot-devtools%7C1.5.2.RELEASE%7Cjar)获取，所有版本都可以在[这里](https://web.archive.org/web/20220626082835/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.springframework.boot%22%20AND%20a%3A%22spring-boot-devtools%22)找到。

## 2。属性默认值

Spring-boot 做了很多自动配置，包括默认启用缓存来提高性能。一个这样的例子是缓存模板引擎使用的模板，例如`thymeleaf`。但是在开发过程中，更重要的是尽快看到变化。

可以使用`application.properties`文件中的属性`spring.thymeleaf.cache=false` 禁用`thymeleaf`的默认缓存行为。我们不需要手动完成这项工作，引入这个`spring-boot-devtools`会自动完成这项工作。

## 3。自动重启

在典型的应用程序开发环境中，开发人员会进行一些更改，构建项目并部署/启动应用程序以使新的更改生效，或者尝试利用`JRebel`等。

使用`spring-boot-devtools,`这个过程也是自动化的。每当类路径中的文件改变时，使用`spring-boot-devtools`的应用程序将导致应用程序重启。此功能的好处是验证所做更改所需的时间大大减少:

```java
19:45:44.804 ... - Included patterns for restart : []
19:45:44.809 ... - Excluded patterns for restart : [/spring-boot-starter/target/classes/, /spring-boot-autoconfigure/target/classes/, /spring-boot-starter-[\w-]+/, /spring-boot/target/classes/, /spring-boot-actuator/target/classes/, /spring-boot-devtools/target/classes/]
19:45:44.810 ... - Matching URLs for reloading : [file:/.../target/test-classes/, file:/.../target/classes/]

 :: Spring Boot ::        (v1.5.2.RELEASE)

2017-03-12 19:45:45.174  ...: Starting Application on machine with PID 7724 (<some path>\target\classes started by user in <project name>)
2017-03-12 19:45:45.175  ...: No active profile set, falling back to default profiles: default
2017-03-12 19:45:45.510  ...: Refreshing org.springframework.boot[[email protected]](/web/20220626082835/https://www.baeldung.com/cdn-cgi/l/email-protection)385c3ca3: startup date [Sun Mar 12 19:45:45 IST 2017]; root of context hierarchy
```

从日志中可以看出，产生应用程序的线程不是`main`线程，而是`restartedMain`线程。项目中的任何更改，无论是 java 文件更改，都将导致项目自动重启:

```java
2017-03-12 19:53:46.204  ...: Closing org.springframework.boot[[email protected]](/web/20220626082835/https://www.baeldung.com/cdn-cgi/l/email-protection)385c3ca3: startup date [Sun Mar 12 19:45:45 IST 2017]; root of context hierarchy
2017-03-12 19:53:46.208  ...: Unregistering JMX-exposed beans on shutdown

 :: Spring Boot ::        (v1.5.2.RELEASE)

2017-03-12 19:53:46.587  ...: Starting Application on machine with PID 7724 (<project path>\target\classes started by user in <project name>)
2017-03-12 19:53:46.588  ...: No active profile set, falling back to default profiles: default
2017-03-12 19:53:46.591  ...: Refreshing org.springframework.boot[[email protected]](/web/20220626082835/https://www.baeldung.com/cdn-cgi/l/email-protection)acaf4a1: startup date [Sun Mar 12 19:53:46 IST 2017]; root of context hierarchy
```

## 4。实时重装

`spring-boot-devtools`模块包括一个嵌入式 LiveReload 服务器，用于在资源发生变化时触发浏览器刷新。

为了在浏览器中实现这一点，我们需要安装 LiveReload 插件，其中一个实现就是 Chrome 的[远程实时重新加载](https://web.archive.org/web/20220626082835/https://chrome.google.com/webstore/detail/remotelivereload/jlppknnillhjgiengoigajegdpieppei?hl=en-GB)。

## 5。全局设置

`spring-boot-devtools`提供了一种配置不与任何应用程序耦合的全局设置的方法。这个文件被命名为`.spring-boot-devtools.properties`，它位于$HOME。

## 6。远程应用程序

### 6.1。通过 HTTP(远程调试隧道)进行远程调试

`spring-boot-devtools`通过 HTTP 提供开箱即用的远程调试功能，要拥有此功能，需要将`spring-boot-devtools`打包为应用程序的一部分。这可以通过在 maven 的插件中禁用`excludeDevtools`配置来实现。

这里有一个简单的例子:

```java
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <configuration>
                <excludeDevtools>false</excludeDevtools>
            </configuration>
        </plugin>
    </plugins>
</build>
```

现在，为了通过 HTTP 进行远程调试，必须采取以下步骤:

1.  An application being deployed and started on the server, should be started with Remote Debugging enabled:

    ```java
    -Xdebug -Xrunjdwp:server=y,transport=dt_socket,suspend=n
    ```

    我们可以看到，这里没有提到远程调试端口。因此，java 将选择一个随机端口

2.  对于同一个项目，打开`Launch configurations`，选择以下选项:
    选择主类:`org.springframework.boot.devtools.RemoteSpringApplication`
    在程序参数中，添加应用程序的 URL，例如`http://localhost:8080`
3.  通过 spring-boot 应用程序调试的默认端口是 8000，可以通过:

    ```java
    spring.devtools.remote.debug.local-port=8010
    ```

    覆盖
4.  现在创建一个远程调试配置，将端口设置为通过属性配置的 **8010** 或 **8000** ，如果坚持默认值

下面是日志的样子:

```java
 .   ____          _                                              __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _          ___               _      \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` |        | _ \___ _ __  ___| |_ ___ \ \ \ \
 \\/  ___)| |_)| | | | | || (_| []::::::[]   / -_) '  \/ _ \  _/ -_) ) ) ) )
  '  |____| .__|_| |_|_| |_\__, |        |_|_\___|_|_|_\___/\__\___|/ / / /
 =========|_|==============|___/===================================/_/_/_/
 :: Spring Boot Remote ::  (v1.5.2.RELEASE)

2017-03-12 22:24:11.089  ...: Starting RemoteSpringApplication v1.5.2.RELEASE on machine with PID 10476 (..\org\springframework\boot\spring-boot-devtools\1.5.2.RELEASE\spring-boot-devtools-1.5.2.RELEASE.jar started by user in project)
2017-03-12 22:24:11.097  ...: No active profile set, falling back to default profiles: default
2017-03-12 22:24:11.357  ...: Refreshing org.spring[[email protected]](/web/20220626082835/https://www.baeldung.com/cdn-cgi/l/email-protection)11e21d0e: startup date [Sun Mar 12 22:24:11 IST 2017]; root of context hierarchy
2017-03-12 22:24:11.869  ...: The connection to http://localhost:8080 is insecure. You should use a URL starting with 'https://'.
2017-03-12 22:24:11.949  ...: LiveReload server is running on port 35729
2017-03-12 22:24:11.983  ...: Started RemoteSpringApplication in 1.24 seconds (JVM running for 1.802)
2017-03-12 22:24:34.324  ...: Remote debug connection opened
```

### 6.2。远程更新

远程客户端监控应用程序类路径的更改，就像远程重启功能一样。类路径中的任何变化都会导致更新的资源被推送到远程应用程序，并触发重启。

当远程客户端启动并运行时，会推送更改，因为只有在那时才可能监控已更改的文件。

日志中是这样的:

```java
2017-03-12 22:33:11.613  INFO 1484 ...: Remote debug connection opened
2017-03-12 22:33:21.869  INFO 1484 ...: Uploaded 1 class resource
```

## 7。结论

通过这篇简短的文章，我们已经展示了如何利用`spring-boot-devtools`模块，通过自动化许多活动，使开发人员体验更好，并减少开发时间。