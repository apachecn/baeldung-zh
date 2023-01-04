# 将 JAR 解压缩到指定的目录

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/extract-jar-to-a-specified-directory>

## 1.概观

通常，当我们在 Java 项目中需要 [JAR](/web/20221231082751/https://www.baeldung.com/java-view-jar-contents) 文件时，我们将它们作为外部库放在类路径中，而不提取它们。然而，我们有时想把它们放到我们的文件系统中。

在本教程中，我们将探索如何在命令行中将一个 JAR 文件解压到一个指定的目录。**我们将使用 Linux 和 Bash 作为例子来说明每种方法**。

## 2.问题简介

在本教程中，我们将使用[番石榴](/web/20221231082751/https://www.baeldung.com/guava-guide)的 [JAR 文件](https://web.archive.org/web/20221231082751/https://github.com/google/guava/releases)作为例子。撰写本文时最新发布的是番石榴- `31.1-jre.jar`。

Java 提供了`jar`命令来创建、更新、查看和提取 JAR 文件。接下来，让我们使用`jar`命令提取 Guava 的 JAR 文件:

```java
$ tree /tmp/test/jarTest
/tmp/test/jarTest
└── guava-31.1-jre.jar

0 directories, 1 file

$ pwd
/tmp/test/jarTest

$ jar xf guava-31.1-jre.jar 

$ tree /tmp/test/jarTest   
/tmp/test/jarTest
├── com
│   └── google
│       ├── common
│       │   ├── annotations
│       │   │   ├── Beta.class
...
27 directories, 2027 files 
```

正如上面的输出所示，**我们可以使用`jar xf`、**提取一个 JAR 文件，默认情况下`jar`命令会将它提取到当前目录。

但是，**`jar`****命令不支持将文件解压到指定目录。**

接下来，让我们看看如何在行动中实现这一点。

## 3.在提取之前输入目标目录

我们已经知道，默认情况下,`jar`命令将给定的 JAR 文件提取到当前工作目录。

于是，一个解决问题的想法出现了:如果我们想要将 JAR 文件提取到指定的目标，**我们可以首先进入目标目录，然后启动`jar`命令。**

接下来，我们来确定这个想法是否如预期的那样起作用。

### 3.1.测试这个想法

首先，让我们创建一个新目录`/tmp/test/newTarget`:

```java
$ mkdir /tmp/test/newTarget

$ tree /tmp/test/newTarget
/tmp/test/newTarget
0 directories, 0 files 
```

接下来，让我们进入目标目录，然后提取 Guava 的 JAR 文件:

```java
$ cd /tmp/test/newTarget && jar xf /tmp/test/jarTest/guava-31.1-jre.jar 
$ tree /tmp/test/newTarget
/tmp/test/newTarget
├── com
│   └── google
│       ├── common
│       │   ├── annotations
│       │   │   ├── Beta.class
...
```

正如上面的例子所示，这个想法是可行的。我们已经将 Guava 的 JAR 文件解压缩到我们想要的目录中。

然而，如果我们重新审视我们已经执行的命令，我们会发现有一些不便之处:

*   如果是新目录，我们必须首先手动创建目标目录。
*   当`cd`命令改变我们当前的工作目录时，我们必须传递 JAR 文件及其绝对路径。
*   在我们执行完命令后，我们会被留在目标目录中，而不是我们开始的地方。

接下来，让我们看看如何改进我们的解决方案。

### 3.2.创建`xJarTo()`功能

我们可以创建一个 shell 函数来自动创建目录并修改 JAR 文件的路径。让我们先来看看这个函数，然后了解它是如何工作的:

```java
#!/bin/bash

xJarTo() {
    the_pwd="$(pwd)"
    the_jar="$1"
    the_dir="$2"

    if [[ "$the_jar" =~ ^[^/].* ]]; then
        the_jar="${the_pwd}/$the_jar"
    fi
    echo "Extracting $the_jar to $the_dir ..."
    mkdir -p "$the_dir"
    cd "$the_dir" && jar xf "$the_jar"
    cd "$the_pwd"
} 
```

该函数首先将用户的当前工作目录(`pwd`)存储在`the_pwd`变量中。

然后，它检查 JAR 文件的路径:

*   如果它以“`/`”开头——绝对路径，那么我们就直接使用用户输入
*   否则–相对路径。我们需要预先考虑用户的当前工作目录，以构建要使用的绝对路径

因此，该函数适应绝对和相对 JAR 路径。

接下来，我们在进入目标目录之前执行`mkdir -p`命令。**`-p`选项告诉`mkdir`命令在给定的路径中创建丢失的目录(如果有的话)。**

然后，我们使用`cd`进入目标目录，并使用准备好的 JAR 文件路径执行`jar` `xf`命令。

最后，我们导航回用户的当前工作目录。

### 3.3.测试功能

现在我们已经了解了`xJarTo()`函数是如何工作的，让我们来测试一下`[source](/web/20221231082751/https://www.baeldung.com/linux/source-command)`函数，看看它是否如预期的那样工作。

首先，让我们测试这样一个场景，目标目录是新的，JAR 文件是一个绝对路径:

```java
$ pwd
/tmp/test

$ xJarTo /tmp/test/jarTest/guava-31.1-jre.jar /tmp/a_new_dir
Extracting /tmp/test/jarTest/guava-31.1-jre.jar to /tmp/a_new_dir ...

$ tree /tmp/a_new_dir
/tmp/a_new_dir
├── com
│   └── google
│       ├── common
│       │   ├── annotations
│       │   │   ├── Beta.class
...
$ pwd
/tmp/test
```

如上面的输出所示，该函数按预期工作。新目录是动态创建的，提取的内容在`/tmp/a_new_dir`下。进一步来说，我们在调用了函数之后仍然在`/tmp/test`之下。

接下来，让我们测试相对 JAR 路径场景:

```java
$ pwd
/tmp/test/jarTest

$ xJarTo guava-31.1-jre.jar /tmp/another_new_dir
Extracting /tmp/test/jarTest/guava-31.1-jre.jar to /tmp/another_new_dir ...

$ tree /tmp/another_new_dir
/tmp/another_new_dir
├── com
│   └── google
│       ├── common
│       │   ├── annotations
│       │   │   ├── Beta.class
...

$ pwd
/tmp/test/jarTest
```

这一次，由于我们在`/tmp/test/jarTest`下，我们将`guava-31.1-jre.jar`直接传递给函数。结果这个函数也完成了工作。

## 4.使用其他解压缩命令

JAR 文件使用 ZIP 压缩。所以，**所有的解压工具都可以解压 JAR 文件**。如果 unzip 实用程序支持解压到指定的目录，它就解决了这个问题。

这种方法要求我们的系统上至少有一个可用的解压缩工具。否则，我们需要安装它。值得一提的是，**在系统上安装包通常需要`root`或 [`sudo`](/web/20221231082751/https://www.baeldung.com/linux/sudo-command) 权限**。

那么接下来，让我们以 [`unzip`](https://web.archive.org/web/20221231082751/https://linux.die.net/man/1/unzip) 为例，展示一下流行的 [`yum`和`apt`](/web/20221231082751/https://www.baeldung.com/linux/yum-and-apt) 包管理器的安装命令:

```java
# apt
sudo apt install unzip

# yum
sudo yum install unzip
```

**`unzip`命令支持`-d`选项，将 ZIP 存档解压到不同的目录**:

```java
$ unzip guava-31.1-jre.jar -d /tmp/unzip_new_dir
$ tree /tmp/unzip_new_dir
/tmp/unzip_new_dir
├── com
│   └── google
│       ├── common
│       │   ├── annotations
│       │   │   ├── Beta.class
... 
```

正如我们看到的，`unzip -d`将 JAR 文件解压到指定的目录中。

## 5.结论

在本文中，我们学习了两种将 JAR 文件解压到指定目录的方法。

为此，我们可以创建一个 shell 函数来包装标准的 JAR 命令。或者，如果我们在系统上有其他可用的解压缩工具，比如`unzip`，我们可以使用它们相应的选项将 JAR 内容提取到所需的目录。