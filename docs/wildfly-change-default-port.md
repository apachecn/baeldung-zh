# 更改 WildFly 中的默认端口

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/wildfly-change-default-port>

## 1.概观

在这个快速教程中，我们将看看如何在 WildFly 中更改默认端口；一般这个端口是 8080。

当然， [WildFly](https://web.archive.org/web/20220627093154/http://wildfly.org/) 是 JBoss 社区维护的一个流行的开源应用服务器。

## 2.使用配置 XML

在独立模式下，我们可以更新配置 XML 文件来更改默认端口。

在 WildFly 安装主目录中，`standalone.xml`可以在`standalone/configuration`文件夹中找到。我们可以在任何文本编辑器中打开该文件，并在下面一行中用我们选择的任何端口替换默认端口 8080:

```java
<socket-binding name="http" port="${jboss.http.port:8080}"/>
```

还有一种方法可以通过调整 XML 来更改默认端口。在同一个`standalone.xml`中，我们可以**为端口**设置一个偏移值:

```java
<socket-binding-group name="standard-sockets" default-interface="public" port-offset="${jboss.socket.binding.port-offset:0}">
```

默认值为 0，表示 HTTP 端口为 8080。如果我们将偏移值更改为 10，那么 HTTP 端口将是 8090 (8080+10)。**我们必须记住，偏移也会影响其他端口**。

例如，https 端口将更改为 8453 (8443+10)。

## 3.使用系统属性

启动服务器时，可以通过设置系统属性`jboss.http.port – `来更改默认的 WildFly 端口。

**对于 Windows:**

```java
standalone.bat -Djboss.http.port=<Desired_Port_Number>
```

**对于 Unix/Linux:**

```java
standalone.sh -Djboss.http.port=<Desired_Port_Number>
```

## 4.结论

在这篇快速的文章中，我们发现了如何在 WildFly 中很容易地更改默认端口。

您可以在我们的[上一篇文章](/web/20220627093154/https://www.baeldung.com/java-servers)中探索可用于 Java 开发的不同流行服务器。