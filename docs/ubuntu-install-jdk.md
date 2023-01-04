# 在 Ubuntu 上安装 Java

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/ubuntu-install-jdk>

## 1。概述

在本教程中，我们将介绍在 Ubuntu 上安装 JDK 的不同方法。然后，我们简单比较一下方法。最后，我们将展示如何在 Ubuntu 系统上管理多个 Java 安装。

作为每种方法的先决条件，我们需要

*   Ubuntu 系统
*   以拥有`sudo`权限的非根用户身份登录

下面描述的指令已经在 Ubuntu 18.10、18.04 LTS、16.04 LTS 和 14.04 LTS 上测试过。对于 Ubuntu 14.04 LTS 版来说，有一些不同，在文本中已经提到了。

请注意，您可以从 OpenJDK 和 Oracle 下载的包以及存储库中可用的包都会定期更新。确切的软件包名称可能会在几个月内改变，但基本的安装方法将保持不变。

## 2。安装 JDK 11

如果我们想使用最新和最好的 JDK 版本，通常手动安装是可行的。这意味着从 OpenJDK 或 Oracle 站点下载一个包，并对其进行设置，使其符合`apt`如何设置 JDK 包的惯例。

### 2.1。手动安装 open JDK 11

首先我们来下载一下最近发布的 OpenJDK 11 的`tar`存档:

```java
$ wget https://download.java.net/java/ga/jdk11/openjdk-11_linux-x64_bin.tar.gz
```

我们将下载包的总和`sha256`与 OpenJDK 网站上提供的总和[进行比较:](https://web.archive.org/web/20221206023446/https://download.java.net/java/ga/jdk11/openjdk-11_linux-x64_bin.tar.gz.sha256)

```java
$ sha256sum openjdk-11_linux-x64_bin.tar.gz
```

让我们提取`tar`档案:

```java
$ tar xzvf openjdk-11_linux-x64_bin.tar.gz
```

接下来，让我们将刚刚提取的`jdk-11`目录移动到`/usr/lib/jvm`的子目录中。下一节中描述的`apt`包也将它们的 JDK 放到这个目录中:

```java
$ sudo mkdir /usr/lib/jvm
$ sudo mv jdk-11 /usr/lib/jvm/openjdk-11-manual-installation/ 
```

现在，我们想要**使`java`和`javac`命令可用**。一种可能是为它们创建符号链接，例如在`/usr/bin`目录中。但是，我们将为它们安装一个替代品。这样，如果我们希望安装其他版本的 JDK，它们可以很好地配合使用:

```java
$ sudo update-alternatives --install /usr/bin/java java /usr/lib/jvm/openjdk-11-manual-installation/bin/java 1
$ sudo update-alternatives --install /usr/bin/javac javac /usr/lib/jvm/openjdk-11-manual-installation/bin/javac 1
```

让我们验证安装:

```java
$ java -version
```

从输出中我们可以看到，我们确实安装了最新版本的 OpenJDK JRE 和 JVM:

```java
openjdk version "11" 2018-09-25
OpenJDK Runtime Environment 18.9 (build 11+28)
OpenJDK 64-Bit Server VM 18.9 (build 11+28, mixed mode) 
```

让我们看看编译器的版本:

```java
$ javac -version
```

```java
javac 11
```

### 2.2。手动安装甲骨文 JDK 11

如果我们想确保使用最新版本的 Oracle JDK，我们可以遵循类似的手动安装工作流，就像 OpenJDK 一样。为了从[甲骨文网站](https://web.archive.org/web/20221206023446/https://www.oracle.com/technetwork/java/javase/downloads/index.html)、**下载 JDK 11 号的`tar`档案，我们必须先接受许可协议**。出于这个原因，通过`wget`下载比 OpenJDK 更复杂一些:

```java
$ wget -c --header "Cookie: oraclelicense=accept-securebackup-cookie" \
http://download.oracle.com/otn-pub/java/jdk/11.0.1+13/90cf5d8f270a4347a95050320eef3fb7/jdk-11.0.1_linux-x64_bin.tar.gz
```

上面的示例下载了 11.0.1 的软件包，每个次要版本的确切下载链接都有所不同。

以下步骤与 OpenJDK 相同:

```java
$ sha256sum jdk-11.0.1_linux-x64_bin.tar.gz
$ tar xzvf jdk-11.0.1_linux-x64_bin.tar.gz
$ sudo mkdir /usr/lib/jvm
$ sudo mv jdk-11.0.1 /usr/lib/jvm/oracle-jdk-11-manual-installation/
$ sudo update-alternatives --install /usr/bin/java java /usr/lib/jvm/oracle-jdk-11-manual-installation/bin/java 1
$ sudo update-alternatives --install /usr/bin/javac javac /usr/lib/jvm/oracle-jdk-11-manual-installation/bin/javac 1
```

验证也是一样。但是输出显示，这一次，我们安装的不是 OpenJDK，而是 Java(TM):

```java
$ java -version
```

```java
java version "11.0.1" 2018-10-16 LTS
Java(TM) SE Runtime Environment 18.9 (build 11.0.1+13-LTS)
Java HotSpot(TM) 64-Bit Server VM 18.9 (build 11.0.1+13-LTS, mixed mode)
```

对于编译器来说:

```java
$ javac -version
```

```java
javac 11.0.1
```

### 2.3。从 PPA 安装甲骨文 JDK 11

目前，Oracle JDK 11 也在 PPA(个人包归档)中提供。这个安装包括 2 个步骤:将存储库添加到我们的系统中，并通过`apt:`从存储库中安装软件包

```java
$ sudo add-apt-repository ppa:linuxuprising/java
$ sudo apt update
$ sudo apt install oracle-java11-installer
```

验证步骤应显示与第 2.2.1 节中手动安装后相同的结果。：

```java
$ java -version
```

```java
java version "11.0.1" 2018-10-16 LTS
Java(TM) SE Runtime Environment 18.9 (build 11.0.1+13-LTS)
Java HotSpot(TM) 64-Bit Server VM 18.9 (build 11.0.1+13-LTS, mixed mode)
```

对于编译器来说:

```java
$ javac -version
```

```java
javac 11.0.1
```

在 Ubuntu 14.04 LTS 上，默认情况下`add-apt-repository`命令不可用。为了添加一个存储库，首先我们需要安装`software-properties-common`包。

```java
$ sudo apt update
$ sudo apt install software-properties-common
```

之后，我们可以继续如上图所示的`add-apt-repository, apt update `和`apt install`。

## 3。安装 JDK 8

### 3.1。在 Ubuntu 16.04 LTS 及更新版本上安装 open JDK 8

JDK 8 是一个已经存在了一段时间的 LTS 版本。出于这个原因，我们可以在大多数受支持的 Ubuntu 版本的“主”存储库中找到 OpenJDK 8 的最新版本。当然，我们也可以访问 OpenJDK 网站，在那里获取一个包，并按照我们在上一节中看到的相同方式安装它。

但是使用`apt`工具和“主”存储库提供了一些好处。默认情况下，所有 Ubuntu 系统上都有“主”库。它得到了 Canonical 的支持——也是维护 Ubuntu 本身的公司。

让我们用`apt`从“主”存储库中安装 OpenJDK 8:

```java
$ sudo apt update
$ sudo apt install openjdk-8-jdk
```

现在，让我们验证安装:

```java
$ java -version
```

结果应该列出一个运行时环境和一个 JVM:

```java
openjdk version "1.8.0_181"
OpenJDK Runtime Environment (build 1.8.0_181-8u181-b13-0ubuntu0.18.04.1-b13)
OpenJDK 64-Bit Server VM (build 25.181-b13, mixed mode)
```

让我们检查一下`javac`可执行文件是否也可用:

```java
$ javac -version
```

现在我们应该会看到如上所示的相同版本号:

```java
javac 1.8.0_181
```

### 3.2。在 LTS Ubuntu 14.04 上安装 open JDK 8

在 Ubuntu 14.04 LTS 版上，OpenJDK 包在“主”存储库中不可用，所以我们将从`openjdk-r` PPA 安装它们。正如我们在上面 2.3 节中看到的，默认情况下`add-apt-repository`命令是不可用的。我们需要`software-properties-common`的包装:

```java
$ sudo apt update
$ sudo apt install software-properties-common
$ sudo add-apt-repository ppa:openjdk-r/ppa
$ sudo apt update
$ sudo apt install openjdk-8-jdk
```

### 3.3。从 PPA 安装甲骨文 JDK 8

“主”存储库不包含任何专有软件。**如果我们想用`apt`安装 Oracle Java，我们必须使用来自 PPA** 的软件包。我们已经了解了如何从`linuxuprising` PPA 安装 Oracle JDK 11。对于 Java 8，我们可以在`webupd8team` PPA 中找到这些包。

首先，我们需要将 PPA `apt`存储库添加到我们的系统中:

```java
$ sudo add-apt-repository ppa:webupd8team/java
```

然后我们可以用通常的方式安装这个包:

```java
$ sudo apt update
$ sudo apt install oracle-java8-installer
```

在安装过程中，我们必须接受 Oracle 的许可协议。让我们验证安装:

```java
$ java -version
```

输出显示了一个 Java(TM) JRE 和 JVM:

```java
java version "1.8.0_181"
Java(TM) SE Runtime Environment (build 1.8.0_181-b13)
Java HotSpot(TM) 64-Bit Server VM (build 25.181-b13, mixed mode)
```

我们还可以验证编译器是否已经安装:

```java
$ javac -version
```

```java
javac 1.8.0_181
```

## 4.安装 JDK 10

不再支持 Java 10 和 Java 9 版本。您可以按照第 2 节中类似的步骤手动安装它们。您可以从以下位置获取软件包:

*   [https://jdk.java.net/archive/](https://web.archive.org/web/20221206023446/https://jdk.java.net/archive/)
*   [https://www . Oracle . com/tech network/Java/javase/archive-139210 . html](https://web.archive.org/web/20221206023446/https://www.oracle.com/technetwork/java/javase/archive-139210.html)

两个站点包含相同的警告:

> 提供这些旧版本的 JDK 是为了帮助开发人员调试旧系统中的问题。它们没有使用最新的安全补丁进行更新，不建议在生产中使用。

### 4.1。手动安装 open JDK 10

我们来看看如何安装 OpenJDK 10.0.1:

```java
$ wget https://download.java.net/java/GA/jdk10/10.0.1/fb4372174a714e6b8c52526dc134031e/10/openjdk-10.0.1_linux-x64_bin.tar.gz
$ sha256sum openjdk-10.0.1_linux-x64_bin.tar.gz
$ tar xzvf openjdk-10.0.1_linux-x64_bin.tar.gz
$ sudo mkdir /usr/lib/jvm
$ sudo mv jdk-10.0.1 /usr/lib/jvm/openjdk-10-manual-installation/
$ sudo update-alternatives --install /usr/bin/java java /usr/lib/jvm/openjdk-10-manual-installation/bin/java 1
$ sudo update-alternatives --install /usr/bin/javac javac /usr/lib/jvm/openjdk-10-manual-installation/bin/javac 1
$ java -version
$ javac -version
```

### 4.2。手动安装 Oracle JDK 10

正如我们在 2.2 节中看到的。为了从甲骨文网站**下载软件包，我们必须首先接受许可协议**。与受支持的版本相反，我们不能通过`wget`和 cookie 下载较旧的 Oracle JDKs。我们需要前往[https://www . Oracle . com/tech network/Java/javase/downloads/Java-archive-javase 10-4425482 . html](https://web.archive.org/web/20221206023446/https://www.oracle.com/technetwork/java/javase/downloads/java-archive-javase10-4425482.html)并下载`tar.gz`文件。之后，我们遵循熟悉的步骤:

```java
$ sha256sum jdk-10.0.2_linux-x64_bin.tar.gz
$ tar xzvf jdk-10.0.2_linux-x64_bin.tar.gz
$ sudo mkdir /usr/lib/jvm
$ sudo mv jdk-10.0.2 /usr/lib/jvm/oracle-jdk-10-manual-installation/
$ sudo update-alternatives --install /usr/bin/java java /usr/lib/jvm/oracle-jdk-10-manual-installation/bin/java 1
$ sudo update-alternatives --install /usr/bin/javac javac /usr/lib/jvm/oracle-jdk-10-manual-installation/bin/javac 1
$ java -version
$ javac -version
```

## 5.安装 JDK 9

### 5.1.手动安装 OpenJDK 9

就像我们在上面看到的 OpenJDK 10.0.1 一样，我们通过`wget`下载 OpenJDK 9 包，并根据约定进行设置:

```java
$ wget https://download.java.net/java/GA/jdk9/9.0.4/binaries/openjdk-9.0.4_linux-x64_bin.tar.gz
$ sha256sum openjdk-9.0.4_linux-x64_bin.tar.gz
$ tar xzvf openjdk-9.0.4_linux-x64_bin.tar.gz
$ sudo mkdir /usr/lib/jvm
$ sudo mv jdk-9.0.4 /usr/lib/jvm/openjdk-9-manual-installation/
$ sudo update-alternatives --install /usr/bin/java java /usr/lib/jvm/openjdk-9-manual-installation/bin/java 1
$ sudo update-alternatives --install /usr/bin/javac javac /usr/lib/jvm/openjdk-9-manual-installation/bin/javac 1
$ java -version
$ javac -version
```

### 5.2.手动安装 Oracle JDK 9

我们再次使用与 JDK 10 号相同的方法。我们需要前往[https://www . Oracle . com/tech network/Java/javase/downloads/Java-archive-javase 9-3934878 . html](https://web.archive.org/web/20221206023446/https://www.oracle.com/technetwork/java/javase/downloads/java-archive-javase9-3934878.html)并下载`tar.gz`文件。之后，我们遵循熟悉的步骤:

```java
$ sha256sum jdk-9.0.4_linux-x64_bin.tar.gz
$ tar xzvf jdk-9.0.4_linux-x64_bin.tar.gz
$ sudo mkdir /usr/lib/jvm
$ sudo mv jdk-9.0.4 /usr/lib/jvm/oracle-jdk-9-manual-installation/
$ sudo update-alternatives --install /usr/bin/java java /usr/lib/jvm/oracle-jdk-9-manual-installation/bin/java 1
$ sudo update-alternatives --install /usr/bin/javac javac /usr/lib/jvm/oracle-jdk-9-manual-installation/bin/javac 1
$ java -version
$ javac -version
```

## 6。比较

我们已经看到了在 Ubuntu 上安装 JDK 的三种不同方式。让我们快速概述一下它们，指出各自的优缺点。

### 6.1。“主”储存库

这就是**【Ubuntu 原生】的安装方式**。一个很大的优势是我们用`apt update`和`apt upgrade`通过“通常的`apt`工作流”更新包。

此外，“主”存储库由 Canonical 维护，Canonical**提供相当快的(如果不是即时的)更新**。例如，OpenJDK 版本 10.0.1 和 10.0.2 都是在发布后一个月内同步的。

### 6.2。PPA

PPAs 是由个人开发者或团队维护的小型软件仓库。这也意味着更新频率取决于维护者。

来自 PPAs 的包裹被认为比“主”仓库中的包裹风险更大。首先，我们必须将 PPA 明确地添加到系统的存储库列表中，表明我们信任它。之后，我们可以通过通常的`apt`工具(`apt update`和`apt upgrade`)来管理包。

### 6.3。手动安装

我们直接从 OpenJDK 或 Oracle 站点下载软件包。虽然这种方法提供了很大的灵活性，但更新是我们的责任。如果我们想拥有最新最伟大的 JDK，这是必经之路。

## 7。探索 JDKs 的其他版本

第 2 节和第 3 节中的示例反映了 LTS Ubuntu 18.04 的当前状态。请记住，JDK 和相应的包会定期更新。因此，知道如何**探索我们当前的可能性**是很有用的。

在这一节中，我们将重点研究“主”存储库中的 OpenJDK 包。如果我们已经用`add-apt-repository`添加了一个 PPA，我们可以用类似的方式用`apt list`和`apt show`来探索它。

为了发现哪些 PPAs 可供选择，我们可以前往 https://launchpad.net/。如果我们在“主”存储库和 PPAs 中找不到我们要找的东西，我们将不得不退回到手动安装。

如果我们想使用一个不受支持的版本，那也是很困难的。在撰写本文时，我们没有在 OpenJDK 和 Oracle 网站上找到任何 Java 9 或 Java 10 的包。

让我们看看“主”存储库中还有哪些其他的 JDK 包:

```java
$ apt list openjdk*jdk
```

在 Ubuntu 18.04 LTS 上，我们可以在两个当前的 LTS Java 版本之间进行选择:

```java
Listing... Done
openjdk-11-jdk/bionic-updates,bionic-security,now 10.0.2+13-1ubuntu0.18.04.2 amd64 [installed,automatic]
openjdk-8-jdk/bionic-updates,bionic-security 8u181-b13-0ubuntu0.18.04.1 amd64
```

另外值得注意的是，虽然这个包名为`openjdk-11-jdk`，但截至本文撰写之时，它实际上安装的是 10.0.2 版本。这种情况可能很快会改变。我们可以看到，如果我们检查包装:

```java
$ apt show openjdk-11-jdk
```

让我们看看输出的“依赖”部分。请注意，这些包(例如 JRE)也是与`openjdk-11-jdk`一起安装的:

```java
Depends: openjdk-11-jre (= 10.0.2+13-1ubuntu0.18.04.2),
openjdk-11-jdk-headless (= 10.0.2+13-1ubuntu0.18.04.2),
libc6 (>= 2.2.5)
```

让我们看看除了默认的 jdk 包之外，我们还有哪些其他的包:

```java
$ apt list openjdk-11*
```

```java
Listing... Done
openjdk-11-dbg/bionic-updates,bionic-security 10.0.2+13-1ubuntu0.18.04.2 amd64
openjdk-11-demo/bionic-updates,bionic-security 10.0.2+13-1ubuntu0.18.04.2 amd64
openjdk-11-doc/bionic-updates,bionic-updates,bionic-security,bionic-security 10.0.2+13-1ubuntu0.18.04.2 all
openjdk-11-jdk/bionic-updates,bionic-security 10.0.2+13-1ubuntu0.18.04.2 amd64
openjdk-11-jdk-headless/bionic-updates,bionic-security 10.0.2+13-1ubuntu0.18.04.2 amd64
openjdk-11-jre/bionic-updates,bionic-security,now 10.0.2+13-1ubuntu0.18.04.2 amd64 [installed,automatic]
openjdk-11-jre-headless/bionic-updates,bionic-security,now 10.0.2+13-1ubuntu0.18.04.2 amd64 [installed,automatic]
openjdk-11-jre-zero/bionic-updates,bionic-security 10.0.2+13-1ubuntu0.18.04.2 amd64
openjdk-11-source/bionic-updates,bionic-updates,bionic-security,bionic-security 10.0.2+13-1ubuntu0.18.04.2 all
```

我们可能会发现其中一些软件包很有用。例如，`openjdk-11-source`包含 Java 核心 API 的类的源文件，而`openjdk-11-dbg`包含调试符号。

除了`openjdk-*`家族，还有`default-jdk` 套餐，值得一探:

```java
$ apt show default-jdk
```

在输出的最后，描述说:

> 此依赖项包指向 Java 运行时，或为此体系结构推荐的 Java 兼容开发工具包…

在 Ubuntu 18.04 LTS 的情况下，它是目前的包`openjdk-11-jdk`。

## 8。概述:Java 版本和包

现在，让我们看看在撰写本文时，不同版本的 Java 是如何安装在 Ubuntu 18.04 LTS 上的:

| 版本 | OpenJDK | Oracle Java |
| --- | --- | --- |
| Eleven | 手动安装 | 在`linuxuprising` PPA 中手动安装
中的`oracle-java11-installer` |
| Ten | 手动安装–不支持 | 手动安装–不支持 |
| nine | 手动安装–不支持 | 手动安装–不支持 |
| eight | `openjdk-8-jdk`在“主”储存库中 | `oracle-java8-installer`在`webupd8team` PPA |

## 9。Ubuntu 系统上的多个 Java 版本

在 Ubuntu 上管理同一软件的多个版本的标准方法是通过 Debian Alternatives 系统。大多数时候，我们通过`update-alternatives`程序创建、维护和展示备选方案。

**当`apt`安装 JDK 软件包时，它会自动添加替代项。**在手动安装的情况下，我们已经看到了如何分别为`java`和`javac`添加替换选项。

让我们看看我们的替代方案:

```java
$ update-alternatives --display java
```

在我们的测试系统上，我们安装了两个不同版本的 OpenJDK，输出列出了两个选项及其各自的优先级:

```java
java - auto mode
link best version is /usr/lib/jvm/java-11-openjdk-amd64/bin/java
link currently points to /usr/lib/jvm/java-11-openjdk-amd64/bin/java
link java is /usr/bin/java
slave java.1.gz is /usr/share/man/man1/java.1.gz
/usr/lib/jvm/java-11-openjdk-amd64/bin/java - priority 1101
slave java.1.gz: /usr/lib/jvm/java-11-openjdk-amd64/man/man1/java.1.gz
/usr/lib/jvm/java-8-openjdk-amd64/jre/bin/java - priority 1081
slave java.1.gz: /usr/lib/jvm/java-8-openjdk-amd64/jre/man/man1/java.1.gz
```

现在我们已经看到了我们的选择，**我们也可以在它们之间切换:**

```java
$ sudo update-alternatives --config java
```

此外，我们还获得了一个交互式输出，我们可以通过键盘在选项之间切换:

```java
There are 2 choices for the alternative java (providing /usr/bin/java).

Selection Path Priority Status
------------------------------------------------------------
* 0 /usr/lib/jvm/java-11-openjdk-amd64/bin/java 1101 auto mode
1 /usr/lib/jvm/java-11-openjdk-amd64/bin/java 1101 manual mode
2 /usr/lib/jvm/java-8-openjdk-amd64/jre/bin/java 1081 manual mode

Press <enter> to keep the current choice[*], or type selection number:
```

如果我们在用不同版本的 Java 编写的多个应用程序上工作，我们可能还需要不同版本的其他软件(例如 Maven，一些应用服务器)。在这种情况下，我们可能要考虑使用更大的抽象，如 Docker 容器。

## 10。结论

总而言之，在本文中，我们已经看到了从“主”存储库、从 PPA 以及手动安装 JDK 的例子。我们简要比较了这三种安装方法。

最后，我们已经看到了如何使用`update-alternatives`管理 Ubuntu 系统上的多个 Java 安装。

下一步，[设置`JAVA_HOME`环境变量](/web/20221206023446/https://www.baeldung.com/java-home-on-windows-7-8-10-mac-os-x-linux)可能是有用的。