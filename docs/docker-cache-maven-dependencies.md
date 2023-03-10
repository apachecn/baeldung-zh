# 用 Docker 缓存 Maven 依赖项

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/ops/docker-cache-maven-dependencies>

## 1.介绍

在本教程中，我们将展示如何在 Docker 中构建 Maven 项目。首先，我们将从一个简单的单模块 Java 项目开始，展示如何利用 Docker 中的多阶段构建来对构建过程进行 Docker 化。接下来，我们将展示如何使用 Buildkit 来缓存多个构建之间的依赖关系。最后，我们将介绍如何在多模块应用中利用层缓存。

## 2.多阶段分层构建

在本文中，我们将创建一个简单的 Java 应用程序，并将 [Guava](/web/20221103202654/https://www.baeldung.com/guava-guide) 作为依赖项。我们将使用 [maven-assembly 插件](/web/20221103202654/https://www.baeldung.com/executable-jar-with-maven)创建一个胖罐子。代码和 Maven 配置将在本文中省略，因为它们不是主题。

多阶段构建是优化 Docker 构建流程的一个好方法。它们使我们能够将整个过程保存在一个文件中，并帮助我们将 Docker 图像保持得尽可能小。在第一阶段，我们将运行一个 Maven 构建并创建我们的 fat JAR，在第二阶段，我们将复制这个 JAR 并定义一个入口点:

```java
FROM maven:alpine as build
ENV HOME=/usr/app
RUN mkdir -p $HOME
WORKDIR $HOME
ADD . $HOME
RUN mvn package

FROM openjdk:8-jdk-alpine 
COPY --from=build /usr/app/target/single-module-caching-1.0-SNAPSHOT-jar-with-dependencies.jar /app/runner.jar
ENTRYPOINT java -jar /app/runner.jar
```

这种方法让我们保持最终的 Docker 映像较小，因为它不包含 Maven 可执行文件或我们的源代码。

让我们创建 Docker 图像:

```java
docker build -t maven-caching .
```

接下来，让我们从图像开始一个容器:

```java
docker run maven-caching
```

当我们修改代码并重新运行构建时，我们会注意到 Maven `package`任务之前的所有命令都被缓存并立即执行。**由于我们的代码比项目依赖项更改得更频繁，我们可以使用 Docker 层缓存将依赖项下载和代码编译分开，以缩短构建时间**:

```java
FROM maven:alpine as build
ENV HOME=/usr/app
RUN mkdir -p $HOME
WORKDIR $HOME
ADD pom.xml $HOME
RUN mvn verify --fail-never
ADD . $HOME
RUN mvn package

FROM openjdk:8-jdk-alpine 
COPY --from=build /usr/app/target/single-module-caching-1.0-SNAPSHOT-jar-with-dependencies.jar /app/runner.jar
ENTRYPOINT java -jar /app/runner.jar
```

当我们只修改我们的代码时，运行后续的构建会快得多，因为 Docker 会从缓存中提取层。

## 3.使用 BuildKit 缓存

Docker 版本 18.09 引入了 BuildKit，作为对现有构建系统的一次革新。彻底改革的目的是提高性能、存储管理和安全性。我们可以利用 BuildKit 来保持多个构建之间的状态。这样，Maven 不会每次都下载依赖项，因为我们有永久存储。为了在 Docker 安装中启用 BuildKit，我们需要编辑`daemon.json`文件:

```java
...
{
"features": {
    "buildkit": true
}}
...
```

启用 BuildKit 后，我们可以将 docker 文件更改为:

```java
FROM maven:alpine as build
ENV HOME=/usr/app
RUN mkdir -p $HOME
WORKDIR $HOME
ADD . $HOME
RUN --mount=type=cache,target=/root/.m2 mvn -f $HOME/pom.xml clean package

FROM openjdk:8-jdk-alpine
COPY --from=build /usr/app/target/single-module-caching-1.0-SNAPSHOT-jar-with-dependencies.jar /app/runner.jar
ENTRYPOINT java -jar /app/runner.jar
```

**当我们更改代码或 `pom.xml`文件时，Docker 会一直执行 ADD 并运行 Maven 命令。**第一次运行的构建时间将会是最长的，因为 Maven 必须下载依赖项。**后续运行将使用本地依赖项，执行速度更快。**

这种方法需要维护 Docker 卷作为依赖项的存储。有时，我们必须强迫 Maven 用 Dockerfile 中的`-U`标志更新我们的依赖关系。

## 4.多模块 Maven 项目的缓存

在前面的小节中，我们展示了如何利用不同的方法来加快单模块 Maven 项目的 Docker 映像的构建时间。对于更复杂的应用程序，这些方法不是最佳的。多模块 Maven 项目通常有一个模块是我们应用程序的入口点。一个或多个模块包含我们的逻辑，并被列为依赖项。

由于子模块被列为依赖项，它们会阻止 Docker 进行层缓存，并触发 Maven 再次下载所有依赖项。**这个解决方案在大多数情况下都很好**，**，但是正如我们所说的，它可能需要不时地强制更新以获取更新的子模块。**为了避免这种情况，我们可以将项目分成不同的层，并使用 Maven 增量构建:

```java
FROM maven:alpine as build
ENV HOME=/usr/app
RUN mkdir -p $HOME
WORKDIR $HOME

ADD pom.xml $HOME
ADD core/pom.xml $HOME/core/pom.xml
ADD runner/pom.xml $HOME/runner/pom.xml

RUN mvn -pl core verify --fail-never
ADD core $HOME/core
RUN mvn -pl core install
RUN mvn -pl runner verify --fail-never
ADD runner $HOME/runner
RUN mvn -pl core,runner package

FROM openjdk:8-jdk-alpine
COPY --from=build /usr/app/runner/target/runner-0.0.1-SNAPSHOT-jar-with-dependencies.jar /app/runner.jar
ENTRYPOINT java -jar /app/runner.jar
```

在这个 Dockerfile 文件中，我们复制了所有的`pom.xml` 文件，并递增地构建每个子模块，然后我们最终打包整个应用程序。经验法则是，我们构建的子模块在链的后面会更频繁地改变。

## 5.结论

在本文中，我们介绍了如何使用 Docker 构建 Maven 项目。首先，我们介绍了如何利用分层来缓存不经常改变的部分。接下来，我们讲述了如何使用 BuildKit 来保持构建之间的状态。最后，我们展示了如何用增量构建来构建多模块 Maven 项目。和往常一样，完整的代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221103202654/https://github.com/eugenp/tutorials/tree/master/docker-modules/docker-caching)