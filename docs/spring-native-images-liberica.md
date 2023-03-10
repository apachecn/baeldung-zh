# 使用 Spring Native 和 Liberica 工具构建本机映像，并进行速度比较

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-native-images-liberica>

## 1.概观

随着微服务架构越来越受欢迎，巨型单片应用程序正在成为过去。Java 没有停滞不前，而是适应了现代需求。例如，Oracle、Red Hat、BellSoft 和其他贡献者正在积极开发 [GraalVM](/web/20220627153941/https://www.baeldung.com/graal-java-jit-compiler) 项目。此外，微服务特定框架 [Quarkus](/web/20220627153941/https://www.baeldung.com/spring-boot-vs-quarkus) 于一年前发布。就 Spring Boot 而言，VMware 已经在 Spring Native 项目上工作了两年。

因此，由于 VMware 和 BellSoft 之间的合作，Spring Native 成为了一个端到端的原生映像解决方案，其中包括基于 GraalVM 源代码的工具 [Liberica 原生映像工具包](https://web.archive.org/web/20220627153941/https://bell-sw.com/pages/liberica-native-image-kit/)。Spring Native 和 Liberica NIK 允许开发人员创建 Spring Boot 应用程序的本地可执行文件，以优化资源消耗和最小化启动时间。

在本教程中，我们将通过以三种方式构建和运行同一个应用程序来探索如何在 Spring Boot 应用程序中使用本机映像技术——作为经典的 JAR 文件；作为具有 Liberica JDK 和 Spring Native 的原生图像容器；以及作为具有 Liberica 原生图像套件的原生图像。然后我们再对比一下他们的启动速度。在任何情况下，我们都将使用 Spring Native 项目中的 [petclinic](https://web.archive.org/web/20220627153941/https://github.com/spring-projects/spring-petclinic) JDBC 应用程序作为例子。

## 2.JDK 自由基金会成立

首先，让我们为您的系统安装 Java 运行时。我们可以访问 [Liberica JDK 下载页面](https://web.archive.org/web/20220627153941/https://bell-sw.com/pages/downloads/)并为我们的平台选择版本。让我们使用 JDK 11，x86 Linux 标准 JDK 包。

有两种方法来安装利百丽卡 JDK。一种是通过包管理器或者下载`.tar.gz package`(或者。`zip`适用于 Windows 的软件包)。

后者是一种更先进的方法，但不用担心，它只需要四个步骤。我们首先需要转到我们想要安装的目录:

```java
cd directory_path_name
```

在不离开目录的情况下，我们可以运行:

```java
wget https://download.bell-sw.com/java/11.0.14.1+1/bellsoft-jdk11.0.14.1+1-linux-amd64.tar.gz
```

如果我们没有`wget`命令，可以通过 brew install `wget`安装(针对 Linux 和 Mac)。

这样，我们将把运行时解包到我们所在的目录中:

```java
tar -zxvf bellsoft-jdk11.0.14.1+1-linux-amd64.tar.gz
```

安装完成后，如果我们想节省磁盘空间，可以删除`.tar.gz`文件。

最后，我们需要通过指向 Liberica JDK 目录来设置`JAVA_HOME`变量:

```java
export JAVA_HOME=$(pwd)/jdk-11.0.14.1
```

**请注意:macOS 和 Windows 用户可以参考 [Liberica JDK 安装指南](https://web.archive.org/web/20220627153941/https://bell-sw.com/pages/liberica_install_guide-11.0.14/)获取说明。**

## 3.获取 Spring 原生项目

我们可以通过运行以下命令获得带有 petclinic 应用程序示例的 Spring Native 项目:

```java
git clone https://github.com/spring-projects-experimental/spring-native.git
```

## 4.构建 JAR 文件

我们希望使用整个 Spring 原生项目中的一个样本，所以通过运行以下命令转到 spring petclinic JDBC 的目录:

```java
export PROJECT_DIR=$(pwd)/spring-native/samples/petclinic-jdbc && cd $PROJECT_DIR
```

要构建 JAR 文件，我们可以应用以下命令:

```java
./mvnw clean install
```

这将为我们提供一个 24 MB 的`target/petclinic-jdbc-0.0.1-SNAPSHOT.jar`。我们将通过运行以下命令来测试它:

```java
java -jar target/petclinic-jdbc-0.0.1-SNAPSHOT.jar
```

## 5.用 Liberica JDK 构建一个本地图像容器

现在让我们将我们的应用程序容器化。

确保我们的 [Docker 守护进程](/web/20220627153941/https://www.baeldung.com/ops/docker-cannot-connect)正在运行。注意，如果我们使用 Windows 或 macOS x86，我们需要为 Docker 分配至少 8 GB 的内存。从 Spring petclinic JDBC 应用程序目录中，我们需要输入命令:

```java
./mvnw spring-boot:build-image
```

这将使用 Spring Boot 构建本机映像容器，我们可以使用:

```java
docker run -it docker.io/library/petclinic-jdbc:0.0.1-SNAPSHOT
```

如果我们与苹果 M1 合作，由于缺少 Docker 的必要构建包，这一步对我们来说将是不可用的。然而，最新版本的 Liberica 原生映像包与 Apple Silicon 完全兼容，因此我们可以进入下一步，使用 NIK 构建原生映像。

## 6.与 NIK 自由党一起建立本土形象

我们将使用 Liberica 本机映像包构建 petclinic 本机映像的另一个版本。下面我们可以找到为 Linux 安装 NIK 的步骤。对于 macOS 或 Windows，让我们参考 [Liberica NIK 安装指南](https://web.archive.org/web/20220627153941/https://bell-sw.com/pages/liberica_install_guide-native-image-kit-21.3.0/)。

我们首先需要转到我们想要安装的目录:

```java
cd directory_path_name
```

然后我们为我们的平台下载 Liberica NIK 核心软件。它包含 Liberica VM 和基于 GraalVM 的原生映像工具包，无需其他语言，因此是构建 Java 原生映像的一个很好的工具。

在我们的例子中，我们将获得 NIK Java 11 Linux 版:

```java
wget https://download.bell-sw.com/vm/22.0.0.2/bellsoft-liberica-vm-openjdk11-22.0.0.2-linux-amd64.tar.gz
```

然后，我们通过运行以下命令来解压文件:

```java
tar -xzf bellsoft-liberica-vm-openjdk11-22.0.0.2-linux-amd64.tar.gz
```

通过指向 NIK 利贝雷卡定义$JAVA_HOME 变量:

```java
export JAVA_HOME=$(pwd)/bellsoft-liberica-vm-openjdk11-22.0.0.2
```

现在，我们转到 petclinic JDBC 应用程序目录:

```java
cd $PROJECT_DIR
```

我们可以通过运行以下命令来创建本机映像:

```java
./mvnw -Pnative install
```

它包括构建的“本地”概要文件，并产生大小为 102.3 MB 的`target/petclinic-jdbc`二进制文件。

## 7.比较启动时间

现在让我们测试我们的应用程序和图像的速度。我们使用英特尔酷睿 i7-8750H CPU 电脑和固态硬盘来运行它们:

*   JAR 文件大约在 3.3 秒后启动
*   我们建造的第一个容器在大约 0.07 秒后启动
*   用 NIK 核心制作的原生映像在 0.068 秒内启动。

## 8.结论

Spring 原生映像已经构建好了，即使在项目还处于测试阶段时也能很好地工作。启动时间大幅减少。

当 Spring Native 与 Liberica 原生映像工具包一起发布时，我们可以期待更好的结果，作为构建原生映像的端到端解决方案。