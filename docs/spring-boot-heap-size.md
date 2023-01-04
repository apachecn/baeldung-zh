# 启动 Spring Boot 应用程序时配置堆大小

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-heap-size>

## 1.介绍

在本教程中，我们将学习如何在启动 Spring Boot 应用程序时配置[堆大小](/web/20221022100505/https://www.baeldung.com/java-stack-heap)。我们将配置`-Xms`和`-Xmx`设置，它们对应于起始堆大小和最大堆大小。

然后，当在命令行上使用`mvn`启动应用程序时，我们将首先使用 Maven 来配置堆大小。我们还将看看如何使用 Maven 插件设置这些值。接下来，我们将应用程序打包到一个`jar`文件中，并使用提供给`java -jar`命令的 [JVM 参数](/web/20221022100505/https://www.baeldung.com/jvm-parameters)运行它。

最后，我们将创建一个`.conf`文件，设置`JAVA_OPTS`和[使用 Linux System V Init 技术将我们的应用程序作为服务](/web/20221022100505/https://www.baeldung.com/spring-boot-app-as-a-service)运行。

## 2.逃离 Maven

### 2.1.传递 JVM 参数

让我们首先创建一个简单的 REST 控制器，它返回一些基本的内存信息，我们可以用这些信息来验证我们的设置:

```java
@GetMapping("memory-status")
public MemoryStats getMemoryStatistics() {
    MemoryStats stats = new MemoryStats();
    stats.setHeapSize(Runtime.getRuntime().totalMemory());
    stats.setHeapMaxSize(Runtime.getRuntime().maxMemory());
    stats.setHeapFreeSize(Runtime.getRuntime().freeMemory());
    return stats;
}
```

让我们使用`mvn spring-boot:run`来运行它，以获得一个基线。一旦我们的应用程序启动，我们可以使用`[curl](/web/20221022100505/https://www.baeldung.com/curl-rest)`来调用我们的 REST 控制器:

```java
curl http://localhost:8080/memory-status
```

我们的结果会因我们的机器而异，但看起来会像这样:

```java
{"heapSize":333447168,"heapMaxSize":5316280320,"heapFreeSize":271148080}
```

对于 Spring Boot 2.x，我们可以使用`-Dspring-boot.run`将参数传递给我们的应用程序。

让我们用`-Dspring-boot.run.jvmArguments`将起始堆大小和最大堆大小传递给我们的应用程序:

```java
mvn spring-boot:run -Dspring-boot.run.jvmArguments="-Xms2048m -Xmx4096m"
```

现在，当我们到达终点时，我们应该看到我们指定的堆设置:

```java
{"heapSize":2147483648,"heapMaxSize":4294967296,"heapFreeSize":2042379008}
```

### 2.2.使用 Maven 插件

通过在`pom.xml`文件中配置`spring-boot-maven-plugin`,我们可以避免每次运行应用程序时都必须提供参数:

让我们配置插件来设置我们想要的堆大小:

```java
<plugins>
    <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
        <executions>
            <execution>
                <configuration>            
                    <mainClass>com.baeldung.heap.HeapSizeDemoApplication</mainClass>
                </configuration>
            </execution>
        </executions>
        <configuration>
            <executable>true</executable>
            <jvmArguments>
                -Xms256m
                -Xmx1g
            </jvmArguments>
        </configuration>
    </plugin>
</plugins>
```

现在，我们可以只使用`mvn spring-boot:run`运行我们的应用程序，并在 ping 端点时看到我们指定的 JVM 参数在使用中:

```java
{"heapSize":259588096,"heapMaxSize":1037959168,"heapFreeSize":226205152}
```

**当使用`-Dspring-boot.run.jvmArguments`从 Maven 运行时，我们在插件中配置的任何 JVM 参数将优先于提供的任何参数。**

## 3.使用`java -jar`运行

如果我们从 j `ar`文件运行我们的应用程序，我们可以向`java`命令提供 JVM 参数。

首先，我们必须在 Maven 文件中将打包指定为`jar`:

```java
<packaging>jar</packaging>
```

然后，我们可以将我们的应用程序打包成一个 j `ar`文件:

```java
mvn clean package
```

现在我们有了 j `ar`文件，我们可以用`java -jar`运行它并覆盖堆配置:

```java
java -Xms512m -Xmx1024m -jar target/spring-boot-runtime-2.jar
```

让我们`curl`我们的端点来检查内存值:

```java
{"heapSize":536870912,"heapMaxSize":1073741824,"heapFreeSize":491597032}
```

## 4.使用`.conf`文件

最后，我们将学习如何使用一个`.conf`文件来设置作为 Linux 服务运行的应用程序的堆大小。

让我们首先创建一个文件，它与我们的应用程序文件`jar`同名，扩展名为`.conf`:`spring-boot-runtime-2.conf`。

我们现在可以将它放在 resources 下的一个文件夹中，并将我们的堆配置添加到`JAVA_OPTS`:

```java
JAVA_OPTS="-Xms512m -Xmx1024m"
```

接下来，我们将修改我们的 Maven 构建，将`spring-boot-runtime-2.conf`文件复制到`jar`文件旁边的`target`文件夹中:

```java
<build>
    <finalName>${project.artifactId}</finalName>
    <resources>
        <resource>
            <directory>src/main/resources/heap</directory>
            <targetPath>${project.build.directory}</targetPath>
            <filtering>true</filtering>
            <includes>
                <include>${project.name}.conf</include>
            </includes>
        </resource>
    </resources>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <executions>
                <execution>
                    <configuration>
                        <mainClass>com.baeldung.heap.HeapSizeDemoApplication</mainClass>
                    </configuration>
                </execution>
            </executions>
            <configuration>
                <executable>true</executable>
            </configuration>
        </plugin>
    </plugins>
</build>
```

**我们还需要将`executable`设置为`true`来将我们的应用程序作为服务运行。**

我们可以打包我们的`jar`文件，并使用 Maven 复制我们的`.conf`文件:

```java
mvn clean package spring-boot:repackage
```

让我们创建我们的`init.d`服务:

```java
sudo ln -s /path/to/spring-boot-runtime-2.jar /etc/init.d/spring-boot-runtime-2
```

现在，让我们开始我们的应用程序:

```java
sudo /etc/init.d/spring-boot-runtime-2 start
```

然后，当我们到达我们的端点时，我们应该看到在`.conf`文件中指定的`JAVA_OPT`值被考虑:

```java
{"heapSize":538968064,"heapMaxSize":1073741824,"heapFreeSize":445879544}
```

## 5.结论

在这个简短的教程中，我们研究了如何为运行 Spring Boot 应用程序的三种常见方式覆盖 Java 堆设置。我们从 Maven 开始，既在命令行修改这些值，也在 Spring Boot Maven 插件中设置它们。

接下来，我们使用`java -jar`运行应用程序`jar`文件，并传入 JVM 参数。

最后，我们通过在 fat `jar`旁边设置一个.`conf`文件并创建一个 System V init 服务来运行我们的应用程序，来查看一个可能的产品级解决方案。

从 Spring Boot fat `jar,`中创建服务和守护进程还有其他解决方案，其中许多提供了覆盖 JVM 参数的特定方法。

与往常一样，GitHub 上的示例代码[可用。](https://web.archive.org/web/20221022100505/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-runtime-2)