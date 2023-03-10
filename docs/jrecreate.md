# 探索再创造

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jrecreate>

## 1.EJDK 简介

EJDK(Embedded Java Development Kit)是由 Oracle 推出的，用于解决为所有可用的嵌入式平台提供二进制文件的问题。我们可以从[甲骨文网站](https://web.archive.org/web/20220627171553/http://www.oracle.com/technetwork/java/embedded/downloads/java-embedded-java-se-download-359230.html)下载最新的 EJDK。

简单地说，它包含了创建特定于平台的 JRE 的工具。

## 2.`jrecreate`

`EJDK`为 Windows 提供`jrecreate.bat`，为 Unix/Linux 平台提供`jrecreate.sh`。该工具有助于为我们希望使用的平台组装定制的 JRE，并介绍给:

*   尽量减少 Oracle 为每个平台发布的二进制文件
*   使为其他平台创建定制的 JRE 变得容易

以下语法用于执行`jrecreate` 命令；在 Unix/Linux 中:

```java
$jrecreate.sh -<option>/--<option> <argument-if-any>
```

在 Windows 中:

```java
$jrecreate.bat -<option>/--<option> <argument-if-any>
```

注意，我们可以为单个 JRE 创建添加多个选项。现在，让我们看看该工具的一些可用选项。

## 3.`jrecreate` 的选项

### 3.1。目的地

`destination`选项是必需的，它指定应该在其中创建目标 JRE 的目录:

```java
$jrecreate.sh -d /SampleJRE
```

运行上述命令时，将在指定位置创建一个默认的 JRE。命令行输出将是:

```java
Building JRE using Options {
    ejdk-home: /installDir/ejdk1.8.0/bin/..
    dest: /SampleJRE
    target: linux_i586
    vm: all
    runtime: jre
    debug: false
    keep-debug-info: false
    no-compression: false
    dry-run: false
    verbose: false
    extension: []
}

Target JRE Size is 55,205 KB (on disk usage may be greater).
Embedded JRE created successfully
```

从上面的结果中，我们可以看到目标 JRE 是在指定的目标目录中创建的。所有其他选项都采用默认值。

### 3.2。配置文件

`profile`选项用于管理目标 JRE 的大小。概要文件定义了要包含的 API 的功能。如果未指定配置文件选项，默认情况下，该工具将包括所有 JRE APIs:

```java
$jrecreate.sh -d /SampleJRECompact1/ -p compact1
```

将创建一个带有`compact1` 概要文件的 JRE。我们也可以在命令中用`––profile`代替`-p`。命令行输出将显示以下结果:

```java
Building JRE using Options {
    ejdk-home: /installDir/ejdk1.8.0/bin/..
    dest: /SampleJRECompact1
    target: linux_i586
    vm: minimal
    runtime: compact1 profile
    debug: false
    keep-debug-info: false
    no-compression: false
    dry-run: false
    verbose: false
    extension: []
}

Target JRE Size is 10,808 KB (on disk usage may be greater).
Embedded JRE created successfully
```

在上面的结果中，注意到`runtime` 选项的值为`compact1`。还要注意，结果 JRE 的大小不到 11MB，比上一个示例中的 55MB 有所下降。

配置文件设置有三个选项:`compact1`、`compact2,` 和`compact3.`

### 3.3。JVM

`jvm`选项用于根据用户的需求用特定的 JVM 定制我们的目标 JRE。默认情况下，如果没有指定`profile`和`jvm`选项，它将包含所有可用的 JVM(客户机、服务器和最小 JVM ):

```java
$jrecreate.sh -d /SampleJREClientJVM/ --vm client
```

将创建一个带有`client` jvm 的 JRE。命令行输出将显示以下结果:

```java
Building JRE using Options {
    ejdk-home: /installDir/ejdk1.8.0/bin/..
    dest: /SampleJREClientJVM
    target: linux_i586
    vm: Client
    runtime: jre
    debug: false
    keep-debug-info: false
    no-compression: false
    dry-run: false
    verbose: false
    extension: []
}

Target JRE Size is 46,217 KB (on disk usage may be greater).
Embedded JRE created successfully
```

在上面的结果中，注意到`vm` 选项的值是`Client`。我们还可以用这个选项指定其他 JVM，比如`server` 和`minimal` 。

### 3.4。分机

`extension`选项用于包含目标 JRE 的各种允许的扩展。默认情况下，不会添加扩展名:

```java
$jrecreate.sh -d /SampleJRESunecExt/ -x sunec
```

将创建一个带有`extension` sunec(椭圆曲线加密的安全提供者)的 JRE。我们也可以在命令中用`––extension`代替`-x`。命令行输出将显示以下结果:

```java
Building JRE using Options {
    ejdk-home: /installDir/ejdk1.8.0/bin/..
    dest: /SampleJRESunecExt
    target: linux_i586
    vm: all
    runtime: jre
    debug: false
    keep-debug-info: false
    no-compression: false
    dry-run: false
    verbose: false
    extension: [sunec]
}

Target JRE Size is 55,462 KB (on disk usage may be greater).
Embedded JRE created successfully
```

在上面的结果中，注意到`extension` 选项的值是`sunec`。使用此选项可以添加多个扩展。

### 3.5。其他选项

除了上面讨论的主要选项，`jrecreate` 还为用户提供了更多的选项:

*   `––help`:显示 jrecreate 工具的命令行选项概要
*   `––debug`:创建支持调试的 JRE
*   `––keep-debug-info`:保存来自类和未签名的 JAR 文件的调试信息
*   `––dry-run`:在不实际创建 JRE 的情况下执行一次试运行
*   `––no-compression`:用未签名的未压缩格式的 JAR 文件创建一个 JRE
*   `––verbose`:显示所有`jrecreate`命令的详细输出

## 4.结论

在本教程中，我们学习了 EJDK 的基础知识，以及如何使用`jrecreate`工具来生成特定于平台的 JRE。