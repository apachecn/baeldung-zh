# Quasar 协同程序简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-quasar-coroutines>

## 1.介绍

协程是 Java 线程的替代，因为它们提供了一种在非常高的并发级别上执行可中断任务的方法，但是在 Project Loom 完成之前，我们必须依靠库支持来获得它。

在本教程中，我们将看看 Quasar，一个提供协同例程支持的库。

## 2.设置

我们将使用最新版本的 Quasar，它需要 Java 11 或更高版本。但是，示例应用程序也可以与兼容 Java 7 和 8 的 Quasar 的早期版本一起工作。

Quasar 提供了三个依赖项,我们需要将它们包含在我们的构建中:

```java
<dependency>
    <groupId>co.paralleluniverse</groupId>
    <artifactId>quasar-core</artifactId>
    <version>0.8.0</version>
</dependency>
<dependency>
    <groupId>co.paralleluniverse</groupId>
    <artifactId>quasar-actors</artifactId>
    <version>0.8.0</version>
</dependency>
<dependency>
    <groupId>co.paralleluniverse</groupId>
    <artifactId>quasar-reactive-streams</artifactId>
    <version>0.8.0</version>
</dependency> 
```

Quasar 的实现依赖于字节码插装来正确工作。为了执行字节码插装，我们有两种选择:

*   在编译时，或者
*   在运行时使用 Java 代理

使用 Java 代理是首选方式，因为它没有任何特殊的构建要求，可以在任何设置下工作。

### 2.1.用 Maven 指定 Java 代理

为了用 Maven 运行 Java 代理，我们需要包含`[maven-dependency-plugin](https://web.archive.org/web/20221208143839/https://maven.apache.org/plugins/maven-dependency-plugin/)`来总是运行`properties`目标:

```java
<plugin>
    <artifactId>maven-dependency-plugin</artifactId>
    <version>3.1.1</version>
    <executions>
        <execution>
            <id>getClasspathFilenames</id>
            <goals>
               <goal>properties</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

`properties` 目标**将生成一个指向类路径上`quasar-core.jar`位置的属性。**

为了执行我们的应用程序，我们将使用 [`exec-maven-plugin`](https://web.archive.org/web/20221208143839/https://www.mojohaus.org/exec-maven-plugin/) :

```java
<plugin>
    <groupId>org.codehaus.mojo</groupId>
    <artifactId>exec-maven-plugin</artifactId>
    <version>3.0.0</version>
    <configuration>
        <workingDirectory>target/classes</workingDirectory>
        <executable>echo</executable>
        <arguments>
            <argument>-javaagent:${co.paralleluniverse:quasar-core:jar}</argument>
            <argument>-classpath</argument> <classpath/>
            <argument>com.baeldung.quasar.QuasarHelloWorldKt</argument>
        </arguments>
    </configuration>
</plugin>
```

为了使用该插件并启动我们的应用程序，我们将运行 Maven:

```java
mvn compile dependency:properties exec:exec
```

## 3.实现协程

为了实现协程，我们将使用 Quasar 库中的`Fibers`。 **`Fibers`提供轻量级线程**，将由 JVM 而不是操作系统管理。因为它们只需要很少的 RAM，给 CPU 带来的负担也小得多，所以我们可以在应用程序中拥有数百万个这样的内存，而不必担心性能问题。

为了启动`fiber,`,我们创建了一个`Fiber<T>`类的实例，它将包装我们想要执行的代码并调用`start`方法:

```java
new Fiber<Void>(() -> {
    System.out.println("Inside fiber coroutine...");
}).start();
```

## 4.结论

在本文中，我们介绍了如何使用 Quasar 库实现协程。我们在这里看到的只是一个最小的工作示例，类星体库能够做得更多。

请在 GitHub 上找到所有的源代码[。](https://web.archive.org/web/20221208143839/https://github.com/eugenp/tutorials/tree/master/libraries-concurrency)