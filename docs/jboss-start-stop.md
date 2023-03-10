# JBoss 服务器–如何启动和停止？

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jboss-start-stop>

## 1。简介

在本教程中，我们将了解如何启动和停止 JBoss 应用服务器。

首先，我们将探索这个服务器的操作模式。此外，我们将讨论如何在每种操作模式下启动和停止服务器。

JBoss 是由 RedHat 开发的开源应用服务器，现在被称为 WildFly。我们可以从官方的[野生动物园网站](https://web.archive.org/web/20220525001826/http://www.wildfly.org/downloads/)下载。

我们可以用两种不同的模式操作这个服务器。然而，这些模式之间的关键区别在于服务器的管理。

## 2。JBoss 独立服务器

**在此模式下**，每个独立的服务器实例都是一个独立的进程。因此，我们必须单独管理这些服务器。

换句话说，如果需要多服务器配置，我们可以启动独立服务器的多个实例。

然而，我们需要在每台服务器上单独部署应用程序。

### 2.1。偏好和配置

独立启动脚本，即用于 OSX/Linux 的`standalone.sh`和用于 Windows 的`standalone.bat`，利用:

*   `standalone.conf/standalone.conf.bat`:定义独立服务器实例的 JVM 首选项
*   `standalone.xml`:定义服务器的默认配置；我们可以在`$JBOSS_HOME/standalone/configuration`下找到。

JBoss 还在同一目录中提供了一些替代配置:

*   `standalone-ee8.xml`:与 `standalone.xml,`相同，但启用了 EE8 技术
*   `standalone-ha.xml`:Java Enterprise Edition 7 web profile 认证配置，具有高可用性
*   `standalone-full.xml` : Java Enterprise Edition 7 全概要认证配置，包括所有必需的 EE 7 技术
*   `standalone-full-ha.xml` : Java Enterprise Edition 7 全配置文件认证配置，具有高可用性

### 2.2。启动服务器

让我们在 OSX/Linux 中打开一个终端，或者在 Windows 中打开一个命令提示符，然后导航到`$JBOSS_HOME/bin`目录。

此外，我们将通过运行以下命令，使用默认配置启动独立服务器:

```java
standalone.sh
```

在 OSX/Linux 或 Windows 中:

```java
standalone.bat
```

类似地，我们可以通过执行以下命令来启动具有备用配置(比如 EE8 功能)的独立服务器:

```java
standalone.sh --server-config=standalone-ee8.xml
```

在 OSX/Linux 或 Windows 中:

```java
standalone.bat --server-config=standalone-ee8.xml
```

此外，为了检查启动是否成功，我们可以打开浏览器并导航到`http://localhost:8080/`。它会显示默认的欢迎页面。

### 2.3。停止服务器

要停止服务器，我们可以简单地按“CTRL+C”。

另外，`jboss-cli`可以用于向服务器的运行实例发出命令。例如，我们可以用它来关闭服务器。

让我们打开一个新的终端或命令提示符并运行:

```java
./jboss-cli.sh --connect command:shutdown
```

在 OSX/Linux 和 Windows 中:

```java
./jboss-cli.bat --connect command:shutdown
```

## 3。托管域服务器

在这种模式下，我们可以从一个控制点管理服务器的多个实例。这些服务器在逻辑上是单个域的成员。这里，单个域控制器进程充当中央管理控制点。

默认情况下，JBoss 提供了很少的服务器实例。我们可以在`$JBOSS_HOME/domain/servers`目录下找到这些实例。

### 3.1。偏好和配置

OSX/Linux 的域启动脚本`domain.sh`和 Windows 的`domain.bat`利用了:

*   `domain.conf/domain.conf.bat`:定义域下服务器的 JVM 首选项
*   `domain.xml`:定义域的配置；我们可以在`$JBOSS_HOME/domain/configuration`下找到。

此外，我们可以定义我们的自定义配置来操作这些服务器，类似于独立服务器的替代配置。

### 3.2。启动服务器

在受管域下启动服务器的过程与独立服务器相同。然而，我们将使用`domain.sh/domain.bat`来代替`standalone.sh/domain.bat,`。

因此，这将在单个域下启动多个服务器实例。

### 3.3。停止服务器

要停止所有的服务器，我们可以简单地按“CTRL+C”。此外，我们可以使用`jboss-cli`停止特定的服务器。

让我们打开一个新的终端或命令提示符并运行:

```java
jboss-cli.sh --connect
```

在 OSX/Linux 或 Windows 中:

```java
jboss-cli.bat --connect
```

目前，我们已连接到域控制器。在这里，我们可以向服务器的多个实例发出命令。例如，要查看该域下的所有服务器:

```java
/host=master:read-children-names(child-type=server-config)
```

类似地，要停止服务器的特定实例，我们将执行:

```java
/host=master/server-config=<server-name>:stop
```

因此，我们可以检查该服务器的状态:

```java
/host=master/server-config=<server-name>:read-resource(include-runtime=true)
```

## 4。结论

在这篇简短的指南中，我们探讨了如何使用不同的配置启动和停止应用服务器。

为了进一步阅读，我们有一篇文章描述了[在 JBoss 应用服务器](/web/20220525001826/https://www.baeldung.com/jboss-war-deploy)上部署`war`文件的过程。