# spring-boot 的区别:重新打包和 Maven 打包

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-repackage-vs-mvn-package>

## 1.概观

[Apache Maven](/web/20221208143917/https://www.baeldung.com/maven) 是一个广泛使用的项目依赖管理工具和项目构建工具。

在过去的几年里， [Spring Boot](/web/20221208143917/https://www.baeldung.com/spring-boot-start) 已经成为一个非常流行的构建应用程序的框架。还有在 Apache Maven 中提供 Spring Boot 支持的 [Spring Boot Maven 插件](https://web.archive.org/web/20221208143917/https://docs.spring.io/spring-boot/docs/current/maven-plugin/reference/htmlsingle/)。

我们知道，当我们想要使用 Maven 将应用程序打包到 JAR 或 WAR 工件中时，我们可以使用`mvn package`。然而，Spring Boot Maven 插件附带了一个`repackage`目标，它也在一个`mvn`命令中被调用。

有时，两个`mvn`命令会混淆。在本教程中，我们将讨论`mvn package `和`spring-boot:repackage`的区别。

## 2.Spring Boot 应用实例

首先，我们将创建一个简单的 Spring Boot 应用程序作为示例:

```
@SpringBootApplication
public class DemoApplication {
    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }
}
```

为了验证我们的应用程序是否启动并运行，让我们创建一个简单的 REST 端点:

```
@RestController
public class DemoRestController {
    @GetMapping(value = "/welcome")
    public ResponseEntity welcomeEndpoint() {
        return ResponseEntity.ok("Welcome to Baeldung Spring Boot Demo!");
    }
}
```

## 3.Maven 的`package`目标

我们只需要 [`spring-boot-starter-web`](https://web.archive.org/web/20221208143917/https://search.maven.org/search?q=a:spring-boot-starter-web%20g:org.springframework.boot) 依赖项来构建我们的 Spring Boot 应用程序:

```
<artifactId>spring-boot-artifacts-2</artifactId>
<packaging>jar</packaging>
...
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
...
```

**Maven 的`package`目标是将编译后的代码打包成可分发的格式**，在本例中是 JAR 格式:

```
$ mvn package
[INFO] Scanning for projects...
[INFO] ------< com.baeldung.spring-boot-modules:spring-boot-artifacts-2 >------
[INFO] Building spring-boot-artifacts-2 1.0.0-SNAPSHOT
[INFO] --------------------------------[ jar ]---------------------------------
 ... 
[INFO] --- maven-jar-plugin:2.4:jar (default-jar) @ spring-boot-artifacts-2 ---
[INFO] Building jar: /home/kent ... /target/spring-boot-artifacts-2.jar
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
 ...
```

执行`mvn package`命令后，我们可以在`target`目录下找到构建好的 JAR 文件`spring-boot-artifacts-2.jar `。让我们[检查创建的 JAR 文件](/web/20221208143917/https://www.baeldung.com/java-view-jar-contents#reviewing-the-jar-command)的内容:

```
$ jar tf target/spring-boot-artifacts-2.jar
META-INF/
META-INF/MANIFEST.MF
com/
com/baeldung/
com/baeldung/demo/
application.yml
com/baeldung/demo/DemoApplication.class
com/baeldung/demo/DemoRestController.class
META-INF/maven/... 
```

正如我们在上面的输出中看到的，**由`mvn package`命令创建的 JAR 文件只包含来自我们项目的源代码**的资源和编译后的 Java 类。

我们可以在另一个项目中使用这个 JAR 文件作为依赖项。然而，我们不能使用`java -jar JAR_FILE` 来执行 JAR 文件，即使它是一个 Spring Boot 应用程序。这是因为运行时依赖项没有绑定。例如，我们没有 servlet 容器来启动 web 上下文。

要使用简单的`java -jar`命令启动我们的 Spring Boot 应用程序，我们需要构建一个[胖罐子](/web/20221208143917/https://www.baeldung.com/deployable-fat-jar-spring-boot#fat-jar--fat-war)。Spring Boot Maven 插件可以帮助我们。

## 4.Spring Boot Maven 插件的`repackage `目标

现在，让我们来看看`spring-boot:repackage`是做什么的。

### 4.1.添加 Spring Boot Maven 插件

为了执行`repackage`目标，我们需要在我们的`pom.xml`中添加 Spring Boot Maven 插件:

```
<build>
    <finalName>${project.artifactId}</finalName>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```

### 4.2.执行`spring-boot:repackage`目标

现在，让我们清理之前构建的 JAR 文件，并尝试一下`spring-boot:repackage`:

```
$ mvn clean spring-boot:repackage     
 ...
[INFO] --- spring-boot-maven-plugin:2.3.3.RELEASE:repackage (default-cli) @ spring-boot-artifacts-2 ---
[INFO] ------------------------------------------------------------------------
[INFO] BUILD FAILURE
[INFO] ------------------------------------------------------------------------
...
[ERROR] Failed to execute goal org.springframework.boot:spring-boot-maven-plugin:2.3.3.RELEASE:repackage (default-cli) 
on project spring-boot-artifacts-2: Execution default-cli of goal 
org.springframework.boot:spring-boot-maven-plugin:2.3.3.RELEASE:repackage failed: Source file must not be null -> [Help 1]
... 
```

哎呀，没用。这是因为**`spring-boot:repackage`目标将现有的 JAR 或 WAR 档案作为源，并将所有项目运行时依赖项与项目类一起重新打包到最终工件中。这样，可以使用命令行`java -jar JAR_FILE.jar`执行重新打包的工件。**

因此，在执行`spring-boot:repackage`目标之前，我们需要首先构建 JAR 文件:

```
$ mvn clean package spring-boot:repackage
 ...
[INFO] Building spring-boot-artifacts-2 1.0.0-SNAPSHOT
[INFO] --------------------------------[ jar ]---------------------------------
 ...
[INFO] --- maven-jar-plugin:2.4:jar (default-jar) @ spring-boot-artifacts-2 ---
[INFO] Building jar: /home/kent/.../target/spring-boot-artifacts-2.jar
[INFO] 
[INFO] --- spring-boot-maven-plugin:2.3.3.RELEASE:repackage (default-cli) @ spring-boot-artifacts-2 ---
[INFO] Replacing main artifact with repackaged archive
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
 ...
```

### 4.3.重新打包的 JAR 文件的内容

现在，如果我们检查`target`目录，我们将看到重新打包的 JAR 文件和原始 JAR 文件:

```
$ ls -1 target/*jar*
target/spring-boot-artifacts-2.jar
target/spring-boot-artifacts-2.jar.original 
```

让我们检查重新打包的 JAR 文件的内容:

```
$ jar tf target/spring-boot-artifacts-2.jar 
META-INF/
META-INF/MANIFEST.MF
 ...
org/springframework/boot/loader/JarLauncher.class
 ...
BOOT-INF/classes/com/baeldung/demo/
BOOT-INF/classes/application.yml
BOOT-INF/classes/com/baeldung/demo/DemoApplication.class
BOOT-INF/classes/com/baeldung/demo/DemoRestController.class
META-INF/maven/com.baeldung.spring-boot-modules/spring-boot-artifacts-2/pom.xml
META-INF/maven/com.baeldung.spring-boot-modules/spring-boot-artifacts-2/pom.properties
BOOT-INF/lib/
BOOT-INF/lib/spring-boot-starter-web-2.3.3.RELEASE.jar
...
BOOT-INF/lib/spring-boot-starter-tomcat-2.3.3.RELEASE.jar
BOOT-INF/lib/tomcat-embed-core-9.0.37.jar
BOOT-INF/lib/jakarta.el-3.0.3.jar
BOOT-INF/lib/tomcat-embed-websocket-9.0.37.jar
BOOT-INF/lib/spring-web-5.2.8.RELEASE.jar
...
BOOT-INF/lib/httpclient-4.5.12.jar
... 
```

如果我们检查上面的输出，它比用`mvn package` 命令构建的 JAR 文件要长得多。

这里，**在重新打包的 JAR 文件中，我们不仅有项目中编译的 Java 类，还有启动我们的 Spring Boot 应用程序**所需的所有运行时库。例如，一个嵌入式的`tomcat`库被打包到`BOOT-INF/lib `目录中。

接下来，让我们启动我们的应用程序并检查它是否工作:

```
$ java -jar target/spring-boot-artifacts-2.jar 
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/

2020-12-22 23:36:32.704  INFO 115154 [main] com.baeldung.demo.DemoApplication      : Starting DemoApplication on YK-Arch with PID 11515...
...
2020-12-22 23:36:34.070  INFO 115154 [main] o.s.b.w.embedded.tomcat.TomcatWebServer: Tomcat started on port(s): 8080 (http) ...
2020-12-22 23:36:34.078  INFO 115154 [main] com.baeldung.demo.DemoApplication      : Started DemoApplication in 1.766 seconds ... 
```

我们的 Spring Boot 应用程序已经启动并运行。现在，让我们通过调用我们的`/welcome`端点来验证它:

```
$ curl http://localhost:8080/welcome
Welcome to Baeldung Spring Boot Demo!
```

太好了！我们得到了预期的回应。我们的应用程序运行正常。

### 4.4.在 Maven 的包生命周期中执行`spring-boot:repackage`目标

我们可以在我们的`pom.xml`中配置 Spring Boot Maven 插件，以便在 Maven 生命周期的`package`阶段重新打包工件。换句话说，当我们执行`mvn package, `时，`spring-boot:repackage`将被自动执行。

配置非常简单。我们只是将`repackage`目标添加到一个`execution`元素中:

```
<build>
    <finalName>${project.artifactId}</finalName>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <executions>
                <execution>
                    <goals>
                        <goal>repackage</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

现在，让我们再次运行`mvn clean package`:

```
$ mvn clean package
 ...
[INFO] Building spring-boot-artifacts-2 1.0.0-SNAPSHOT
[INFO] --------------------------------[ jar ]---------------------------------
...
[INFO] --- spring-boot-maven-plugin:2.3.3.RELEASE:repackage (default) @ spring-boot-artifacts-2 ---
[INFO] Replacing main artifact with repackaged archive
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
 ...
```

输出显示已经执行了重新打包目标。如果我们检查文件系统，我们会发现重新打包的 JAR 文件被创建:

```
$ ls -lh target/*jar*
-rw-r--r-- 1 kent kent  29M Dec 22 23:56 target/spring-boot-artifacts-2.jar
-rw-r--r-- 1 kent kent 3.6K Dec 22 23:56 target/spring-boot-artifacts-2.jar.original 
```

## 5.结论

在本文中，我们已经讨论了`mvn package`和`spring-boot:repackage`之间的区别。

此外，我们还学习了如何在 Maven 生命周期的`package`阶段执行`spring-boot:repackage`。

和往常一样，这篇文章中的代码都可以在 GitHub 的[上找到。](https://web.archive.org/web/20221208143917/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-artifacts-2)