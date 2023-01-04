# JVM 参数 InitialRAMPercentage、MinRAMPercentage 和 MaxRAMPercentage

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-jvm-parameters-rampercentage>

## 1.概观

在本教程中，我们将讨论几个 [JVM 参数](/web/20220908144053/https://www.baeldung.com/jvm-parameters)，我们可以用它们来设置 JVM 的 RAM 百分比。

Java 8 中引入的参数`InitialRAMPercentage`、`MinRAMPercentage`和`MaxRAMPercentage`有助于配置 Java 应用程序的堆大小。

## 2.`-XX:InitialRAMPercentage`

JVM 参数`InitialRAMPercentage` 允许我们**配置 Java 应用程序的初始堆大小**。它是物理服务器或容器总内存的百分比**，作为双精度值传递。**

例如，如果我们为一个 1 GB 全内存的物理服务器设置- `XX:InitialRAMPercentage=50.0` ，那么初始堆大小大约是 500 MB(1gb 的 50%)。

首先，让我们检查 JVM 中`IntialRAMPercentage` 的默认值:

```java
$ docker run openjdk:8 java -XX:+PrintFlagsFinal -version | grep -E "InitialRAMPercentage"
   double InitialRAMPercentage                      = 1.562500                            {product}

openjdk version "1.8.0_292"
OpenJDK Runtime Environment (build 1.8.0_292-b10)
```

然后，让我们为 JVM 设置 50%的初始堆大小:

```java
$ docker run -m 1GB openjdk:8 java -XX:InitialRAMPercentage=50.0 -XX:+PrintFlagsFinal -version | grep -E "InitialRAMPercentage"
   double InitialRAMPercentage                     := 50.000000                           {product}

openjdk version "1.8.0_292"
OpenJDK Runtime Environment (build 1.8.0_292-b10) 
```

值得注意的是，当我们配置`-Xms`选项时，JVM 会忽略`InitialRAMPercentage`。

## 3.`-XX:MinRAMPercentage`

与名字不同，`MinRAMPercentage` 参数允许**为运行少量内存**(小于 200MB)的 JVM 设置最大堆大小。

首先，我们将探索`MinRAMPercentage`的默认值:

```java
$ docker run openjdk:8 java -XX:+PrintFlagsFinal -version | grep -E "MinRAMPercentage"
   double MinRAMPercentage                      = 50.000000                            {product}

openjdk version "1.8.0_292"
OpenJDK Runtime Environment (build 1.8.0_292-b10)
```

然后，让我们使用参数为总内存为 100MB 的 JVM 设置最大堆大小:

```java
$ docker run -m 100MB openjdk:8 java -XX:MinRAMPercentage=80.0 -XshowSettings:VM -version

VM settings:
    Max. Heap Size (Estimated): 77.38M
    Ergonomics Machine Class: server
    Using VM: OpenJDK 64-Bit Server VM

openjdk version "1.8.0_292"
OpenJDK Runtime Environment (build 1.8.0_292-b10) 
```

此外，JVM 在为小型内存服务器/容器设置最大堆大小时会忽略`MaxRAMPercentage`参数:

```java
$ docker run -m 100MB openjdk:8 java -XX:MinRAMPercentage=80.0 -XX:MaxRAMPercentage=50.0 -XshowSettings:vm -version
VM settings:
    Max. Heap Size (Estimated): 77.38M
    Ergonomics Machine Class: server
    Using VM: OpenJDK 64-Bit Server VM

openjdk version "1.8.0_292"
OpenJDK Runtime Environment (build 1.8.0_292-b10)
```

## 4. `-XX:MaxRAMPercentage`

`MaxRAMPercentage`参数允许**为运行大量内存**(大于 200 MB)的 JVM 设置最大堆大小。

首先，让我们探索一下`MaxRAMPercentage`的默认值:

```java
$ docker run openjdk:8 java -XX:+PrintFlagsFinal -version | grep -E "MaxRAMPercentage"
   double MaxRAMPercentage                      = 25.000000                            {product}

openjdk version "1.8.0_292"
OpenJDK Runtime Environment (build 1.8.0_292-b10)
```

然后，对于总内存为 500 MB 的 JVM，我们可以使用参数将最大堆大小设置为 60%:

```java
$ docker run -m 500MB openjdk:8 java -XX:MaxRAMPercentage=60.0 -XshowSettings:vm -version
VM settings:
    Max. Heap Size (Estimated): 290.00M
    Ergonomics Machine Class: server
    Using VM: OpenJDK 64-Bit Server VM

openjdk version "1.8.0_292"
OpenJDK Runtime Environment (build 1.8.0_292-b10)
```

类似地，对于大型内存服务器/容器，JVM 会忽略`MinRAMPercentage`参数:

```java
$ docker run -m 500MB openjdk:8 java -XX:MaxRAMPercentage=60.0 -XX:MinRAMPercentage=30.0 -XshowSettings:vm -version
VM settings:
    Max. Heap Size (Estimated): 290.00M
    Ergonomics Machine Class: server
    Using VM: OpenJDK 64-Bit Server VM

openjdk version "1.8.0_292"
OpenJDK Runtime Environment (build 1.8.0_292-b10)
```

## 5.结论

在这篇短文中，我们讨论了使用 JVM 参数 `InitialRAMPercentage` 、 `MinRAMPercentage` 和 `MaxRAMPercentage` 来设置 JVM 将用于堆的 RAM 百分比。

首先，我们检查了 JVM 上设置的标志的默认值。然后，我们使用 JVM 参数来设置初始和最大堆大小。