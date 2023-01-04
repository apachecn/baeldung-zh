# Spring Boot 应用即服务

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-app-as-a-service>

## 1。概述

本文探讨了将 Spring Boot 应用程序作为服务运行的一些选项。

首先，我们将解释 web 应用程序的打包选项和系统服务。在随后的章节中，我们将探讨在为基于 Windows 的系统和基于 Linux 的系统设置服务时的不同选择。

最后，我们将参考一些额外的信息来源。

## 2。项目设置和构建说明

### 2.1。包装

传统上，web 应用程序被打包为 Web 应用程序档案(WAR ),并部署到 Web 服务器上。

Spring Boot 应用程序可以打包成 WAR 和 JAR 文件。后者在 JAR 文件中嵌入了一个 web 服务器，这允许您运行应用程序，而不需要安装和配置应用服务器。

### 2.2。Maven 配置

让我们从定义我们的`pom.xml`文件的配置开始:

```java
<packaging>jar</packaging>

<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.4.0.RELEASE</version>
</parent>

<dependencies>
    ....
</dependencies>

<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <configuration>
                <executable>true</executable>
            </configuration>
        </plugin>
    </plugins>
</build>
```

包装必须设置为`jar`。在撰写本文时，我们使用的是 Spring Boot 最新的稳定版本，但 1.3 以后的任何版本都足够了。你可以在这里找到更多关于可用版本[的信息。](https://web.archive.org/web/20220707143823/https://spring.io/projects/spring-boot)

注意，对于`spring-boot-maven-plugin`工件，我们已经将`<executable>`参数设置为`true`。这确保了一个`MANIFEST.MF`文件被添加到 JAR 包中。这个清单包含一个`Main-Class`条目，指定哪个类定义了应用程序的主方法。

### 2.3。构建您的应用程序

在应用程序的根目录中运行以下命令:

```java
$ mvn clean package
```

可执行 JAR 文件现在位于`target`目录中，我们可以通过在命令行上执行以下命令来启动应用程序:

```java
$ java -jar your-app.jar
```

此时，您仍然需要用`-jar`选项调用 Java 解释器。有很多原因可以解释为什么最好通过将应用程序作为服务来调用来启动应用程序。

## 3。在 Linux 上

为了将程序作为后台进程运行，我们可以简单地使用`nohup` Unix 命令，但是由于各种原因，这也不是首选方式。在这个[线程](https://web.archive.org/web/20220707143823/https://stackoverflow.com/questions/958249/whats-the-difference-between-nohup-and-a-daemon)中提供了一个很好的解释。

相反，我们将`daemonize`我们的过程。在 Linux 下，我们可以选择用传统的`System V init`脚本或者用`Systemd`配置文件来配置一个守护进程。前者传统上是最广为人知的选择，但正逐渐被后者取代。

你可以在这里找到更多关于这种差异的细节[。](https://web.archive.org/web/20220707143823/http://www.tecmint.com/systemd-replaces-init-in-linux/)

为了增强安全性，我们首先创建一个运行服务的特定用户，并相应地更改可执行 JAR 文件的权限:

```java
$ sudo useradd baeldung
$ sudo passwd baeldung
$ sudo chown baeldung:baeldung your-app.jar
$ sudo chmod 500 your-app.jar
```

### 3.1。系统 V 初始化

Spring Boot 可执行 JAR 文件使得服务设置过程非常简单:

```java
$ sudo ln -s /path/to/your-app.jar /etc/init.d/your-app
```

上面的命令创建了一个到可执行 JAR 文件的符号链接。您必须使用可执行 JAR 文件的完整路径，否则，符号链接将无法正常工作。此链接使您能够将应用程序作为服务启动:

```java
$ sudo service your-app start
```

该脚本支持标准服务`start`、`stop`、`restart`和`status`命令。此外:

*   它启动在我们刚刚创建的用户`baeldung`下运行的服务
*   它在`/var/run/your-app/your-app.pid`中跟踪应用程序的进程 ID
*   它将控制台日志写到`/var/log/your-app.log`，如果您的应用程序无法正常启动，您可能需要检查这些日志

### 3.2。系统 d

`systemd`服务设置也非常简单。首先，我们使用下面的示例创建一个名为`your-app.service`的脚本，并将其放在`/etc/systemd/system`目录中:

```java
[Unit]
Description=A Spring Boot application
After=syslog.target

[Service]
User=baeldung
ExecStart=/path/to/your-app.jar SuccessExitStatus=143 

[Install] 
WantedBy=multi-user.target
```

记住修改`Description`、`User`和`ExecStart`字段以匹配您的应用。此时，您也应该能够执行前面提到的标准服务命令。

与上一节描述的`System V init`方法相反，应该使用服务脚本中的适当字段显式配置进程 ID 文件和控制台日志文件。一份详尽的选项列表可以在[这里](https://web.archive.org/web/20220707143823/https://www.freedesktop.org/software/systemd/man/systemd.service.html)找到。

### 3.3。新贵

[Upstart](https://web.archive.org/web/20220707143823/http://upstart.ubuntu.com/) 是一个基于事件的服务管理器，是对不同守护进程行为提供更多控制的`System V init`的潜在替代品。

该网站有很好的[安装说明](https://web.archive.org/web/20220707143823/http://upstart.ubuntu.com/getting-started.html)，应该适用于几乎所有的 Linux 发行版。使用 Ubuntu 时，你可能已经安装并配置了它(检查是否有任何作业的名称以`/etc/init`中的“upstart”开头)。

我们创建一个作业`your-app.conf`来启动我们的 Spring Boot 应用程序:

```java
# Place in /home/{user}/.config/upstart

description "Some Spring Boot application"

respawn # attempt service restart if stops abruptly

exec java -jar /path/to/your-app.jar 
```

现在运行“启动你的应用程序”,你的服务就会启动。

Upstart 提供了许多作业配置选项，你可以在这里找到大多数选项[。](https://web.archive.org/web/20220707143823/http://upstart.ubuntu.com/cookbook/)

## 4。在 Windows 上

在这一节中，我们将介绍几个选项，它们可以用来将 Java JAR 作为 Windows 服务运行。

### 4.1。Windows 服务包装器

由于 [Java 服务包装器](https://web.archive.org/web/20220707143823/http://wrapper.tanukisoftware.org/doc/english/index.html)(见下一小节)的 GPL 许可与例如麻省理工学院 Jenkins 的许可相结合的困难， [Windows 服务包装器](https://web.archive.org/web/20220707143823/https://github.com/kohsuke/winsw)项目，也被称为`winsw`，被构思出来。

`Winsw`提供安装/卸载/启动/停止服务的编程方法。此外，它可以用于在 Windows 下作为服务运行任何类型的可执行文件，而 Java 服务包装器，顾名思义，只支持 Java 应用程序。

首先，你在这里下载二进制文件[。接下来，定义我们的 Windows 服务的配置文件`MyApp.xml`应该如下所示:](https://web.archive.org/web/20220707143823/https://repo.jenkins-ci.org/releases/com/sun/winsw/winsw/)

```java
<service>
    <id>MyApp</id>
    <name>MyApp</name>
    <description>This runs Spring Boot as a Service.</description>
    <env name="MYAPP_HOME" value="%BASE%"/>
    <executable>java</executable>
    <arguments>-Xmx256m -jar "%BASE%\MyApp.jar"</arguments>
    <logmode>rotate</logmode>
</service> 
```

最后，您必须将`winsw.exe`重命名为`MyApp.exe`,以便其名称与`MyApp.xml`配置文件相匹配。此后，您可以像这样安装服务:

```java
$ MyApp.exe install
```

同样，你可以用`uninstall`、`start`、`stop`等。

### 4.2。Java 服务包装器

如果您不介意 [Java 服务包装器](https://web.archive.org/web/20220707143823/http://wrapper.tanukisoftware.org/doc/english/index.html)项目的 GPL 许可，这种替代方案可以同样很好地满足您将 JAR 文件配置为 Windows 服务的需求。基本上，Java 服务包装器还要求您在配置文件中指定如何在 Windows 下作为服务运行您的流程。

[这篇文章](https://web.archive.org/web/20220707143823/http://edn.embarcadero.com/article/32068)非常详细地解释了如何在 Windows 下设置这样一个 JAR 文件的执行作为一个服务，所以我们没有必要重复这些信息。

## 5。附加参考

使用 [Apache Commons Daemon](https://web.archive.org/web/20220707143823/https://commons.apache.org/daemon/index.html) 项目的 [Procrun](https://web.archive.org/web/20220707143823/https://commons.apache.org/proper/commons-daemon/procrun.html) ，Spring Boot 应用程序也可以作为 Windows 服务启动。Procrun 是一组应用程序，允许 Windows 用户将 Java 应用程序包装为 Windows 服务。这种服务可以被设置为当机器引导时自动启动，并且将在没有任何用户登录的情况下继续运行。

关于在 Unix 下启动 Spring Boot 应用程序的更多细节可以在[这里](https://web.archive.org/web/20220707143823/https://docs.spring.io/spring-boot/docs/current/reference/html/deployment-install.html)找到。还有关于如何为基于 Redhat 的系统修改 [Systemd 单元文件的详细说明。最后](https://web.archive.org/web/20220707143823/https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/system_administrators_guide/chap-managing_services_with_systemd)

最后，[这个快速的 howto](https://web.archive.org/web/20220707143823/https://coderwall.com/p/ssuaxa/how-to-make-a-jar-file-linux-executable) 描述了如何将一个 Bash 脚本合并到您的 JAR 文件中，以便它本身成为一个可执行文件！

## 6。结论

服务允许您非常有效地管理应用程序状态，正如我们所看到的，Spring Boot 应用程序的服务设置现在比以往任何时候都更容易。

请记住，在运行您的服务时，要遵循关于用户权限的重要而简单的安全措施。