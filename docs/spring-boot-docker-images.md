# 用 Spring Boot 创建码头工人图像

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-docker-images>

## 1.介绍

随着越来越多的组织转向容器和虚拟服务器，Docker 正在成为软件开发工作流中更重要的一部分。为此， [Spring Boot](/web/20220727020704/https://www.baeldung.com/spring-boot) 2.3 中一个伟大的新特性是能够轻松地为 Spring Boot 应用程序创建 Docker 映像。

在本教程中，我们将看看如何为 Spring Boot 应用程序创建 Docker 图像。

## 2.传统码头建筑

用 Spring Boot 构建 Docker 映像的传统方式是使用 Docker 文件。下面是一个简单的例子:

```java
FROM openjdk:8-jdk-alpine
EXPOSE 8080
ARG JAR_FILE=target/demo-app-1.0.0.jar
ADD ${JAR_FILE} app.jar
ENTRYPOINT ["java","-jar","/app.jar"]
```

然后我们可以使用`docker build`命令创建一个 Docker 映像。这对于大多数应用程序来说都很好，但是也有一些缺点。

首先，我们使用 Spring Boot 创造的脂肪罐。**这会影响启动时间，尤其是在集装箱环境中**。我们可以通过添加 jar 文件的展开内容来节省启动时间。

第二，Docker 图像是分层构建的。Spring Boot fat jars 的性质导致所有应用程序代码和第三方库都被放在一个层中。**这意味着即使只有一行代码发生变化，整个层都必须重新构建**。

通过在构建之前分解 jar，应用程序代码和第三方库都有了自己的层。这允许我们利用 Docker 的缓存机制。现在，当一行代码发生变化时，只需要重新构建相应的层。

考虑到这一点，让我们看看 Spring Boot 是如何改进创建 Docker 图像的过程的。

## 3.构建包

**build pack 是一个提供框架和应用依赖的工具**。

例如，给定一个 Spring Boot 胖罐子，一个 buildpack 将为我们提供 Java 运行时。这允许我们跳过 Docker 文件，自动获得一个合理的 Docker 图像。

Spring Boot 包括对构建包的 Maven 和 Gradle 支持。例如，用 Maven 构建，我们将运行命令:

```java
./mvnw spring-boot:build-image
```

让我们看看一些相关的输出，看看发生了什么:

```java
[INFO] Building jar: target/demo-0.0.1-SNAPSHOT.jar
...
[INFO] Building image 'docker.io/library/demo:0.0.1-SNAPSHOT'
...
[INFO]  > Pulling builder image 'gcr.io/paketo-buildpacks/builder:base-platform-api-0.3' 100%
...
[INFO]     [creator]     ===> DETECTING
[INFO]     [creator]     5 of 15 buildpacks participating
[INFO]     [creator]     paketo-buildpacks/bellsoft-liberica 2.8.1
[INFO]     [creator]     paketo-buildpacks/executable-jar    1.2.8
[INFO]     [creator]     paketo-buildpacks/apache-tomcat     1.3.1
[INFO]     [creator]     paketo-buildpacks/dist-zip          1.3.6
[INFO]     [creator]     paketo-buildpacks/spring-boot       1.9.1
...
[INFO] Successfully built image 'docker.io/library/demo:0.0.1-SNAPSHOT'
[INFO] Total time:  44.796 s
```

第一行显示我们构建了标准的 fat jar，就像任何典型的 maven 包一样。

下一行开始构建 Docker 映像。紧接着，我们看到构建将[包拉入](https://web.archive.org/web/20220727020704/https://paketo.io/)构建器。

Packeto 是云原生构建包的实现。它负责分析我们的项目，并确定所需的框架和库。在我们的例子中，它确定我们有一个 Spring Boot 项目，并添加所需的构建包。

最后，我们看到生成的 Docker 映像和总构建时间。注意我们第一次构建时，我们花了相当多的时间下载构建包并创建不同的层。

buildpacks 的一个重要特性是 Docker 映像是多层的。因此，如果我们只更改我们的应用程序代码，后续的构建会快得多:

```java
...
[INFO]     [creator]     Reusing layer 'paketo-buildpacks/executable-jar:class-path'
[INFO]     [creator]     Reusing layer 'paketo-buildpacks/spring-boot:web-application-type'
...
[INFO] Successfully built image 'docker.io/library/demo:0.0.1-SNAPSHOT'
...
[INFO] Total time:  10.591 s
```

## 4.分层罐

在某些情况下，我们可能不喜欢使用构建包——也许我们的基础设施已经绑定到另一个工具，或者我们已经有了想要重用的定制 docker 文件。

由于这些原因，Spring Boot 也支持使用分层 jar 构建 Docker 映像。为了理解它是如何工作的，让我们来看一个典型的 Spring Boot 胖罐子布局:

```java
org/
  springframework/
    boot/
  loader/
...
BOOT-INF/
  classes/
...
lib/
...
```

脂肪罐由 3 个主要部分组成:

*   启动 Spring 应用程序所需的引导类
*   应用代码
*   第三方库

对于分层的 jar，结构看起来很相似，但是我们得到了一个新的`layers.idx`文件，它将 fat jar 中的每个目录映射到一个层:

```java
- "dependencies":
  - "BOOT-INF/lib/"
- "spring-boot-loader":
  - "org/"
- "snapshot-dependencies":
- "application":
  - "BOOT-INF/classes/"
  - "BOOT-INF/classpath.idx"
  - "BOOT-INF/layers.idx"
  - "META-INF/"
```

开箱即用，Spring Boot 提供了四层:

*   `dependencies`:来自第三方的典型依赖
*   `snapshot-dependencies`:来自第三方的快照依赖关系
*   `resources`:静态资源
*   `application`:应用代码和资源

**目标是将应用程序代码和第三方库放入反映它们更改频率的层中**。

例如，应用程序代码可能是变化最频繁的，因此它有自己的层。此外，每一层都可以独立发展，只有当一层发生变化时，才会为 Docker 映像重新构建它。

现在我们已经了解了新的分层 jar 结构，让我们看看如何利用它来制作 Docker 图像。

### 4.1.创建分层罐子

首先，我们必须设置我们的项目来创建一个分层的 jar。对于 Maven，这意味着向 POM 的 Spring Boot 插件部分添加一个新的配置:

```java
<plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
    <configuration>
        <layers>
            <enabled>true</enabled>
        </layers>
    </configuration>
</plugin>
```

有了这个配置，Maven `package`命令(及其任何依赖命令)将使用前面提到的四个默认层生成一个新的分层 jar。

### 4.2.查看和提取图层

接下来，我们需要从 jar 中提取这些层，以便 Docker 映像拥有正确的层。

要检查任何分层 jar 的层，我们可以运行命令:

```java
java -Djarmode=layertools -jar demo-0.0.1.jar list
```

然后，为了提取它们，我们将运行:

```java
java -Djarmode=layertools -jar demo-0.0.1.jar extract
```

### 4.3.创建 Docker 图像

将这些层合并到 Docker 映像中的最简单方法是使用 Docker 文件:

```java
FROM adoptopenjdk:11-jre-hotspot as builder
ARG JAR_FILE=target/*.jar
COPY ${JAR_FILE} application.jar
RUN java -Djarmode=layertools -jar application.jar extract

FROM adoptopenjdk:11-jre-hotspot
COPY --from=builder dependencies/ ./
COPY --from=builder snapshot-dependencies/ ./
COPY --from=builder spring-boot-loader/ ./
COPY --from=builder application/ ./
ENTRYPOINT ["java", "org.springframework.boot.loader.JarLauncher"]
```

这个 Docker 文件从我们的 fat jar 中提取图层，然后将每个图层复制到 Docker 映像中。**每个`COPY`指令在最终的 Docker 图像**中产生一个新层。

如果我们构建这个 Docker 文件，我们可以看到分层 jar 中的每一层都作为自己的层添加到 Docker 映像中:

```java
...
Step 6/10 : COPY --from=builder dependencies/ ./
 ---> 2c631b8f9993
Step 7/10 : COPY --from=builder snapshot-dependencies/ ./
 ---> 26e8ceb86b7d
Step 8/10 : COPY --from=builder spring-boot-loader/ ./
 ---> 6dd9eaddad7f
Step 9/10 : COPY --from=builder application/ ./
 ---> dc80cc00a655
...
```

## 5.结论

在本教程中，我们看到了用 Spring Boot 构建 Docker 图像的各种方法。使用 buildpacks，我们可以获得合适的 Docker 映像，而无需样板文件或定制配置。或者，再努力一点，我们可以使用分层的 jar 来获得更定制的 Docker 图像。

本教程中的所有例子都可以在 GitHub 上找到[。](https://web.archive.org/web/20220727020704/https://github.com/eugenp/tutorials/tree/master/docker-modules/docker-spring-boot)

关于使用 Java 和 Docker 的更多信息，请查看 jib 上的[教程。](/web/20220727020704/https://www.baeldung.com/jib-dockerizing)