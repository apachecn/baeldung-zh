# Java 会话超时

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/servlet-session-timeout>

## 1。概述

本教程将展示如何在基于 Servlet 的 web 应用程序中设置**会话超时。**

## 2。`web.xml` 中的全局会话超时

所有 Http 会话的超时可以在 web 应用程序的`web.xml`中配置:

```java
<?xml version="1.0" encoding="UTF-8"?>
<web-app ...>

    ...
    <session-config>
        <session-timeout>10</session-timeout>
    </session-config>

</web-app>
```

请注意，超时值是以分钟设置的**，而不是以秒设置的。**

一个有趣的侧面是，在 Servlet 3.0 环境中，可以使用注释而不是 XML 部署描述符，没有办法以编程方式设置全局会话超时。会话超时的编程配置在 Servlet 规范 JIRA 上确实有一个未解决的问题，但该问题尚未安排。

## 3。每个单独会话的编程超时

当前会话**的超时只有**可以通过`javax.servlet.http.HttpSession`的 API 以编程方式指定:

```java
HttpSession session = request.getSession();
session.setMaxInactiveInterval(10*60);
```

与具有分钟值的`<session-timeout>`元素相反，`setMaxInactiveInterval`方法接受秒值**。**

 **## 4。Tomcat 会话超时

所有的 Tomcat 服务器都为[提供了一个默认的`web.xml`文件](https://web.archive.org/web/20220812054347/https://tomcat.apache.org/tomcat-7.0-doc/default-servlet.html "Default Servlet Configuration for Tomcat 7")，可以为整个 web 服务器进行全局配置——这个文件位于:

```java
$tomcat_home/conf/web.xml
```

这个默认部署描述符将`<session-timeout>`的值配置为 30 分钟。

在它们自己的`web.xml`描述符中提供它们自己的超时值的单个部署的应用将优先于并且**将覆盖这个全局`web.xml`** 配置。

请注意，Jetty 中的[也可能是相同的](https://web.archive.org/web/20220812054347/https://www.eclipse.org/jetty/documentation/current/webdefault-xml.html "The webdefault-xml defaults for web.xml"):该文件位于:

```java
$jetty_home/etc/webdefault.xml
```

## 5。结论

本教程讨论了如何在 Servlet Java 应用程序中配置 HTTP 会话的超时。我们还展示了如何在 web 服务器级别进行设置，包括在 Tomcat 和 Jetty 中。

这些例子的实现可以在 github 项目中找到——这是一个基于 Eclipse 的项目，所以它应该很容易导入和运行。

当项目在本地运行时，可以在以下位置访问主页 html:**