# 如何在 Docker 容器中配置 Java 堆的大小

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/ops/docker-jvm-heap-size>

## 1.概观

当我们在一个容器中运行 Java 时，我们可能希望对它进行调优，以充分利用可用资源。

在本教程中，我们将看到如何在运行 Java 进程的容器中设置 [JVM 参数](/web/20220727020703/https://www.baeldung.com/jvm-parameters)。尽管下面的内容适用于任何 JVM 设置，但我们将重点关注常见的 [`-Xmx` 和`-Xms`标志。](https://web.archive.org/web/20220727020703/https://docs.oracle.com/en/java/javase/11/tools/java.html)

我们还将探讨一些常见的问题，如在某些版本的 Java 上运行的容器化程序，以及如何在一些流行的容器化 Java 应用程序中设置标志。

## 2.Java 容器中的默认堆设置

JVM 非常擅长确定合适的默认内存设置。

在过去，[JVM 不知道分配给容器](https://web.archive.org/web/20220727020703/https://developers.redhat.com/blog/2017/03/14/java-inside-docker/)的内存和 CPU。于是，Java 10 引入了新的设置:`+UseContainerSupport`(默认启用)修复[根源](https://web.archive.org/web/20220727020703/https://bugs.java.com/bugdatabase/view_bug.do?bug_id=JDK-8146115)，开发者在 [8u191](https://web.archive.org/web/20220727020703/https://www.oracle.com/technetwork/java/javase/8u191-relnotes-5032181.html#JDK-8146115) 将修复回移植到 Java 8。JVM 现在根据分配给容器的内存来计算它的内存。

但是，在某些应用中，我们可能仍然希望更改默认设置。

### 2.1.自动内存计算

**当我们不设置`-Xmx` 和 `-Xmx`参数时，JVM 根据系统规范**来确定堆的大小。

让我们看看堆大小:

```java
$ java -XX:+PrintFlagsFinal -version | grep -Ei "maxheapsize|maxram"
```

这将输出:

```java
openjdk version "15" 2020-09-15
OpenJDK Runtime Environment AdoptOpenJDK (build 15+36)
OpenJDK 64-Bit Server VM AdoptOpenJDK (build 15+36, mixed mode, sharing)
   size_t MaxHeapSize      = 4253024256      {product} {ergonomic}
 uint64_t MaxRAM           = 137438953472 {pd product} {default}
    uintx MaxRAMFraction   = 4               {product} {default}
   double MaxRAMPercentage = 25.000000       {product} {default}
   size_t SoftMaxHeapSize  = 4253024256   {manageable} {ergonomic}
```

这里，我们看到 JVM 将其堆大小设置为大约可用 RAM 的 25%。在本例中，它在一个 16GB 的系统上分配了 4GB。

出于测试的目的，让我们创建一个以兆字节为单位打印堆大小的程序:

```java
public static void main(String[] args) {
  int mb = 1024 * 1024;
  MemoryMXBean memoryBean = ManagementFactory.getMemoryMXBean();
  long xmx = memoryBean.getHeapMemoryUsage().getMax() / mb;
  long xms = memoryBean.getHeapMemoryUsage().getInit() / mb;
  LOGGER.log(Level.INFO, "Initial Memory (xms) : {0}mb", xms);
  LOGGER.log(Level.INFO, "Max Memory (xmx) : {0}mb", xmx);
}
```

让我们将该程序放在一个名为`PrintXmxXms.java`的文件中的空目录中。

我们可以在我们的主机上测试它，假设我们已经安装了 JDK。在 Linux 系统中，我们可以编译我们的程序，并从在该目录下打开的终端运行它:

```java
$ javac ./PrintXmxXms.java
$ java -cp . PrintXmxXms
```

在具有 16Gb RAM 的系统上，输出为:

```java
INFO: Initial Memory (xms) : 254mb
INFO: Max Memory (xmx) : 4,056mb
```

现在，让我们在一些容器中尝试一下。

### 2.2.在 JDK 8u191 之前

让我们将下面的`Dockerfile` 添加到包含 Java 程序的文件夹中:

```java
FROM openjdk:8u92-jdk-alpine
COPY *.java /src/
RUN mkdir /app \
    && ls /src \
    && javac /src/PrintXmxXms.java -d /app
CMD ["sh", "-c", \
     "java -version \
      && java -cp /app PrintXmxXms"]
```

这里我们使用的容器使用的是 Java 8 的旧版本，比最新版本中可用的容器支持还早。让我们建立它的形象:

```java
$ docker build -t oldjava .
```

`Dockerfile`中的`CMD` 行是我们运行容器时默认执行的流程。**由于我们没有提供`-Xmx`或`-Xms` JVM 标志，内存设置将被默认。**

让我们运行这个容器:

```java
$ docker run --rm -ti oldjava
openjdk version "1.8.0_92-internal"
OpenJDK Runtime Environment (build 1.8.0_92-...)
OpenJDK 64-Bit Server VM (build 25.92-b14, mixed mode)
Initial Memory (xms) : 198mb
Max Memory (xmx) : 2814mb 
```

现在让我们将容器内存限制为 1GB。

```java
$ docker run --rm -ti --memory=1g oldjava
openjdk version "1.8.0_92-internal"
OpenJDK Runtime Environment (build 1.8.0_92-...)
OpenJDK 64-Bit Server VM (build 25.92-b14, mixed mode)
Initial Memory (xms) : 198mb
Max Memory (xmx) : 2814mb
```

如我们所见，输出完全相同。这证明了旧的 JVM 不考虑容器内存分配。

### 2.3.JDK 8u130 之后

对于相同的测试程序，让我们通过更改`Dockerfile`的第一行来使用一个更新的 JVM 8:

```java
FROM openjdk:8-jdk-alpine
```

我们可以再测试一次:

```java
$ docker build -t newjava .
$ docker run --rm -ti newjava
openjdk version "1.8.0_212"
OpenJDK Runtime Environment (IcedTea 3.12.0) (Alpine 8.212.04-r0)
OpenJDK 64-Bit Server VM (build 25.212-b04, mixed mode)
Initial Memory (xms) : 198mb
Max Memory (xmx) : 2814mb
```

这里，它再次使用整个 docker 主机内存来计算 JVM 堆大小。但是，如果我们为容器分配 1GB 的 RAM:

```java
$ docker run --rm -ti --memory=1g newjava
openjdk version "1.8.0_212"
OpenJDK Runtime Environment (IcedTea 3.12.0) (Alpine 8.212.04-r0)
OpenJDK 64-Bit Server VM (build 25.212-b04, mixed mode)
Initial Memory (xms) : 16mb
Max Memory (xmx) : 247mb 
```

这一次，JVM 根据容器可用的 1GB RAM 计算堆大小。

现在我们了解了 JVM 如何计算它的缺省值，以及为什么我们需要一个最新的 JVM 来获得正确的缺省值，让我们来看看定制设置。

## 3.常用基本图像中的内存设置

### 3.1.OpenJDK 和 AdoptOpenJDK

与其直接在容器的命令中硬编码 JVM 标志，不如使用一个环境变量，比如`JAVA_OPTS`。我们在`Dockerfile`中使用这个变量，但是它可以在容器启动时被修改:

```java
FROM openjdk:8u92-jdk-alpine
COPY src/ /src/
RUN mkdir /app \
 && ls /src \
 && javac /src/com/baeldung/docker/printxmxxms/PrintXmxXms.java \
    -d /app
ENV JAVA_OPTS=""
CMD java $JAVA_OPTS -cp /app \ 
    com.baeldung.docker.printxmxxms.PrintXmxXms
```

现在让我们构建图像:

```java
$ docker build -t openjdk-java .
```

我们可以通过指定`JAVA_OPTS`环境变量来选择运行时的内存设置:

```java
$ docker run --rm -ti -e JAVA_OPTS="-Xms50M -Xmx50M" openjdk-java
INFO: Initial Memory (xms) : 50mb
INFO: Max Memory (xmx) : 48mb 
```

我们应该注意到,`-Xmx`参数和 JVM 报告的最大内存之间有细微的差别。这是因为`Xmx `设置了内存分配池的最大大小，它包括堆、垃圾收集器的幸存者空间和其他池。

### 3.2.Tomcat 9

Tomcat 9 容器有自己的启动脚本，所以要设置 JVM 参数，我们需要使用这些脚本。

`bin/catalina.sh`脚本要求我们**在环境变量`CATALINA_OPTS`** 中设置内存参数。

让我们首先[创建一个 war 文件](/web/20220727020703/https://www.baeldung.com/spring-boot-war-tomcat-deploy)来部署到 Tomcat。

然后，我们将使用一个简单的`Dockerfile`将其容器化，在这里我们声明了`CATALINA_OPTS`环境变量:

```java
FROM tomcat:9.0
COPY ./target/*.war /usr/local/tomcat/webapps/ROOT.war
ENV CATALINA_OPTS="-Xms1G -Xmx1G"
```

然后我们构建容器映像并运行它:

```java
$ docker build -t tomcat .
$ docker run --name tomcat -d -p 8080:8080 \
  -e CATALINA_OPTS="-Xms512M -Xmx512M" tomcat
```

我们应该注意，当我们运行这个函数时，如果我们不提供这个值，我们就向`CATALINA_OPTS.`传递一个新值，不过，我们在`Dockerfile`的第 3 行给出了一些默认值。

我们可以检查应用的运行时参数，并验证我们的选项`-Xmx`和`-Xms`是否存在:

```java
$ docker exec -ti tomcat jps -lv
1 org.apache.catalina.startup.Bootstrap <other options...> -Xms512M -Xmx512M
```

## 4.使用构建插件

Maven 和 Gradle 提供了插件，允许我们在没有`Dockerfile`的情况下创建容器图像。生成的图像通常可以在运行时通过环境变量进行参数化。

我们来看几个例子。

### 4.1.使用 Spring Boot

从 Spring Boot 2.3 开始，Spring Boot [Maven](https://web.archive.org/web/20220727020703/https://docs.spring.io/spring-boot/docs/current/maven-plugin/reference/htmlsingle/#build-image) 和 [Gradle](https://web.archive.org/web/20220727020703/https://docs.spring.io/spring-boot/docs/current/gradle-plugin/reference/htmlsingle/#build-image) 插件可以[构建一个没有`Dockerfile`](https://web.archive.org/web/20220727020703/https://spring.io/guides/topicals/spring-boot-docker/) 的高效容器。

对于 Maven，我们将它们添加到 spring-boot-maven-plugin 中的<`configuration>`块:

```java
<?xml version="1.0" encoding="UTF-8"?>
<project  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <groupId>com.baeldung.docker</groupId>
  <artifactId>heapsizing-demo</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <!-- dependencies... -->
  <build> 
    <plugins> 
      <plugin> 
        <groupId>org.springframework.boot</groupId> 
        <artifactId>spring-boot-maven-plugin</artifactId> 
        <configuration>
          <image>
            <name>heapsizing-demo</name>
          </image>
   <!-- 
    for more options, check:
    https://docs.spring.io/spring-boot/docs/2.4.2/maven-plugin/reference/htmlsingle/#build-image 
   -->
        </configuration>
      </plugin>
    </plugins>
  </build>
</project>
```

要构建项目，请运行:

```java
$ ./mvnw clean spring-boot:build-image
```

这将产生一个名为`<artifact-id>:<version>. `的图像，在本例中为`demo-app:0.0.1-SNAPSHOT`。在引擎盖下，Spring Boot 使用[云本地构建包](https://web.archive.org/web/20220727020703/https://buildpacks.io/)作为底层的容器化技术。

**该插件对 JVM 的内存设置进行硬编码。**然而，我们仍然可以通过设置环境变量`JAVA_OPTS `或`JAVA_TOOL_OPTIONS:`来覆盖它们

```java
$ docker run --rm -ti -p 8080:8080 \
  -e JAVA_TOOL_OPTIONS="-Xms20M -Xmx20M" \
  --memory=1024M heapsizing-demo:0.0.1-SNAPSHOT
```

输出如下所示:

```java
Setting Active Processor Count to 8
Calculated JVM Memory Configuration: [...]
[...]
Picked up JAVA_TOOL_OPTIONS: -Xms20M -Xmx20M 
[...]
```

### 4.2.使用 Google JIB

就像 Spring Boot maven 插件一样， [Google JIB](/web/20220727020703/https://www.baeldung.com/jib-dockerizing) 无需`Dockerfile`就能创建高效的 Docker 图像。Maven 和 Gradle 插件的配置方式类似。Google JIB 还使用环境变量`JAVA_TOOL_OPTIONS`作为 JVM 参数的覆盖机制。

我们可以在任何能够生成可执行 jar 文件的 Java 框架中使用 Google JIB Maven 插件。例如，可以在 Spring Boot 应用程序中使用它来代替`spring-boot-maven`插件来生成容器图像:

```java
<?xml version="1.0" encoding="UTF-8"?>
<project  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">

    <!-- dependencies, ... -->

    <build>
        <plugins>
            <!-- [ other plugins ] -->
            <plugin>
                <groupId>com.google.cloud.tools</groupId>
                <artifactId>jib-maven-plugin</artifactId>
                <version>2.7.1</version>
                <configuration>
                    <to>
                        <image>heapsizing-demo-jib</image>
                    </to>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project> 
```

该映像是使用 maven `jib:DockerBuild`目标构建的:

```java
$ mvn clean install && mvn jib:dockerBuild 
```

我们现在可以像往常一样运行它:

```java
$ docker run --rm -ti -p 8080:8080 \
-e JAVA_TOOL_OPTIONS="-Xms50M -Xmx50M" heapsizing-demo-jib
Picked up JAVA_TOOL_OPTIONS: -Xms50M -Xmx50M
[...]
2021-01-25 17:46:44.070  INFO 1 --- [           main] c.baeldung.docker.XmxXmsDemoApplication  : Started XmxXmsDemoApplication in 1.666 seconds (JVM running for 2.104)
2021-01-25 17:46:44.075  INFO 1 --- [           main] c.baeldung.docker.XmxXmsDemoApplication  : Initial Memory (xms) : 50mb
2021-01-25 17:46:44.075  INFO 1 --- [           main] c.baeldung.docker.XmxXmsDemoApplication  : Max Memory (xmx) : 50mb
```

## 5.结论

在本文中，我们讨论了使用最新的 JVM 来获得在容器中运行良好的默认内存设置的需求。

然后，我们研究了在定制容器映像中设置`-Xms `和`-Xmx `的最佳实践，以及如何使用现有的 Java 应用程序容器来设置其中的 JVM 选项。

最后，我们看到了如何利用构建工具来管理 Java 应用程序的容器化。

与往常一样，GitHub 上的[提供了示例的源代码。](https://web.archive.org/web/20220727020703/https://github.com/eugenp/tutorials/tree/master/docker-modules/docker-containers)