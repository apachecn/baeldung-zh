# 如何设置 WildFly 服务器

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/wildfly-server-setup>

## 1.介绍

在本教程中，我们将探索 [JBoss WildFly 应用服务器](https://web.archive.org/web/20221126223525/https://jbossorg.github.io/wildflysite/)的不同服务器模式和配置。WildFly 是一个轻量级应用服务器，具有 CLI 和管理控制台。

不过，在开始之前，**我们需要确保将`JAVA_HOME`变量设置为 JDK** 。8 版以后的任何东西都适用于 WildFly 17。

## 2.服务器模式

WildFly 默认带有独立和域模式。我们先来看看单机。

### 2.1.单独的

在将 WildFly 下载并解压到本地目录后，我们需要执行用户脚本。

对于 Linux，我们会做:

```java
~/bin/add-user.sh
```

或者对于 Windows:

```java
~\bin\add-user.bat
```

`admin`用户已经存在；但是，我们希望将密码更新为不同的值。**更新默认管理员密码始终是最佳做法，即使计划使用另一个帐户进行服务器管理。**

用户界面将提示我们更新选项(a)中的密码:

```java
Enter the details of the new user to add.
Using realm 'ManagementRealm' as discovered from the existing property files.
Username : admin
User 'admin' already exists and is enabled, would you like to...
 a) Update the existing user password and roles
 b) Disable the existing user
 c) Type a new username
(a):
```

更改密码后，我们需要运行适合操作系统的启动脚本:

```java
~\bin\standalone.bat
```

在服务器输出停止后，我们看到服务器正在运行。

我们现在可以按照输出中的提示在 [`http://127.0.0.1:9990`](https://web.archive.org/web/20221126223525/http://127.0.0.1:9990/) 访问管理控制台。如果我们访问服务器 URL，`[http://127.0.0.1:8080/](https://web.archive.org/web/20221126223525/http://127.0.0.1:8080/),`,应该会出现一条消息提示我们服务器正在运行:

```java
Server is up and running!
```

这种快速简单的设置非常适合运行在单个服务器上的单个实例，但是让我们看看多个实例的域模式。

### 2.2.领域

**如上所述，域模式有多个服务器实例由单个主机控制器**管理。默认数量是两台服务器。

与独立实例类似，我们通过`add-user`脚本添加一个用户。然而，启动脚本被恰当地称为`domain`而不是`standalone.`

在运行了启动过程的相同步骤之后，我们现在来看看各个服务器实例。

例如，我们可以在 [`http://127.0.0.1:8080/`](https://web.archive.org/web/20221126223525/http://127.0.0.1:8080/) 访问第一个服务器的同一个单实例 URL

```java
Server is up and running!
```

同样，我们可以在`[http://127.0.0.1:8230](https://web.archive.org/web/20221126223525/http://127.0.0.1:8230/)`访问服务器二

```java
Server is up and running!
```

稍后我们将看一下服务器 2 的奇怪端口配置。

## 3.部署

为了部署我们的第一个应用程序，我们需要一个好的示例应用程序。

让我们用 [Spring Boot 你好世界](https://web.archive.org/web/20221126223525/https://spring.io/guides/gs/spring-boot/)。稍加修改，它就可以用在野花上。

先来更新一下`pom.xml`建战。我们将更改`packaging `元素并添加 [`maven-war-plugin`](https://web.archive.org/web/20221126223525/https://search.maven.org/search?q=g:org.apache.maven.plugins%20AND%20a:maven-war-plugin) :

```java
<packaging>war</packaging>
...
<build>
  <plugins>
	<plugin>
	  <groupId>org.apache.maven.plugins</groupId>
	  <artifactId>maven-war-plugin</artifactId>
	  <configuration>
		<archive>
		  <manifestEntries>
			<Dependencies>jdk.unsupported</Dependencies>
		  </manifestEntries>
		</archive>
	  </configuration>
	</plugin>
  </plugins>
</build>
```

接下来，我们需要将`Application`类改为[扩展`SpringBootServletInitializer`](/web/20221126223525/https://www.baeldung.com/spring-boot-servlet-initializer) :

```java
public class Application extends SpringBootServletInitializer
```

并覆盖`configure`方法:

```java
 @Override
    protected SpringApplicationBuilder configure(SpringApplicationBuilder application) {
        return application.sources(Application.class);
    }
```

最后，让我们构建项目:

```java
./mvnw package
```

现在，我们可以部署它了。

### 3.1.部署应用程序

让我们看看通过控制台部署我们新的 WildFly 应用程序有多容易。

首先，我们使用前面设置的管理员用户和密码登录到控制台。

点击`Add -> Upload Deployment -> Choose a file…`

其次，我们浏览到 Spring Boot 应用程序的目标目录，并上传 WAR 文件。点击`Next`继续，并编辑新应用的名称。

最后，我们通过同一个 URL-[`http://127.0.0.1:8080/`](https://web.archive.org/web/20221126223525/http://127.0.0.1:8080/)访问应用程序。我们现在看到应用程序输出如下:

```java
Greetings from Spring Boot!
```

作为成功部署应用程序的结果，让我们回顾一些常见的配置属性。

## 4.服务器配置

对于这些常见属性，在管理控制台中查看它们是最方便的。

### 4.1.通用配置属性

首先，让我们看一下`Subsystems`——每个都有一个启用统计的值，可以设置为`true`进行运行时统计:

[![subsystems configuration](img/916baeaf775c34a6ceb95893b811d46a.png)](https://web.archive.org/web/20221126223525/https://baeldung.com/wp-content/uploads/2019/10/subsystems_configuration.png)

在`Paths` 部分，我们看到了服务器使用的几个重要文件路径:

[![paths configuration](img/4fc28dc237d2d1debedf3c01112f173a.png)](https://web.archive.org/web/20221126223525/https://baeldung.com/wp-content/uploads/2019/10/paths_configuration-1024x362.png)

例如，为配置目录和日志目录使用特定的值很有帮助:

```java
jboss.server.config.dir
jboss.server.log.dirw
```

此外，`Datasources and Drivers` 部分提供了管理外部数据源和数据持久层的不同驱动程序:

[![datasources configuration](img/5b08b9cc071176a05757799a62530c2b.png)](https://web.archive.org/web/20221126223525/https://baeldung.com/wp-content/uploads/2019/10/datasources_configuration-1024x522.png)

在`Logging` 子系统中，我们将日志级别从 INFO 更新为 DEBUG，以查看更多的应用程序日志:

[![logging configuration](img/6bb87b2f91681555c9a8bebf742c6902.png)](https://web.archive.org/web/20221126223525/https://baeldung.com/wp-content/uploads/2019/10/logging_configuration-1024x520.png)

### 4.2.服务器配置文件

还记得服务器在域模式下的两个 URL，`[http://127.0.0.1:8230](https://web.archive.org/web/20221126223525/http://127.0.0.1:8230/)`？奇数端口`8230 `是由于`host-slave.xml`配置文件中的默认偏移值`150`造成的:

```java
<server name="server-one" group="main-server-group"/>
<server name="server-two" group="main-server-group" auto-start="true">
  <jvm name="default"/>
  <socket-bindings port-offset="150"/>
</server> 
```

然而，我们可以对`port-offset` 值进行简单的更新:

```java
<socket-bindings port-offset="10"/>
```

因此，我们现在通过 URL `[http://127.0.0.1:8090](https://web.archive.org/web/20221126223525/http://127.0.0.1:8090/)`访问服务器 2。

在方便的管理控制台中使用和配置应用程序的另一个重要特性是，这些设置可以导出为 XML 文件。通过 XML 文件使用一致的配置简化了多种环境的管理。

## 5.监视

WildFly 的命令行界面允许像管理控制台一样查看和更新配置值。

要使用 CLI，我们只需执行:

```java
~\bin\jboss-cli.bat
```

之后，我们键入:

```java
[disconnected /] connect
[[[email protected]](/web/20221126223525/https://www.baeldung.com/cdn-cgi/l/email-protection):9990 /]
```

因此，我们可以按照提示连接到我们的服务器实例。连接后，我们可以实时访问配置。

例如，要在连接到服务器实例时以 XML 形式查看所有当前配置，我们可以运行`read-config-as-xml`命令:

```java
[[[email protected]](/web/20221126223525/https://www.baeldung.com/cdn-cgi/l/email-protection):9990 /] :read-config-as-xml
```

CLI 还可用于任何服务器子系统的运行时统计。最重要的是，CLI 非常适合通过服务器指标识别不同的问题。

## 6.结论

在本教程中，我们介绍了在 WildFly 中设置和部署第一个应用程序，以及服务器的一些配置选项。

Spring Boot 示例项目和 XML 配置的导出可以在 GitHub 上查看[。](https://web.archive.org/web/20221126223525/https://github.com/eugenp/tutorials/tree/master/server-modules/wildfly)