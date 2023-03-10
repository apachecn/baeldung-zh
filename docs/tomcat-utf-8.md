# 准备好雄猫 UTF 8 号

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/tomcat-utf-8>

## 1。简介

UTF-8 是 web 应用程序中最常用的字符编码。它支持目前世界上使用的所有语言，包括中文、朝鲜语和日语。

在本文中，我们演示了确保 Tomcat 中 UTF-8 所需的所有配置。

## 2。连接器配置

连接器侦听特定端口上的连接。我们需要确保我们所有的连接器都使用 UTF-8 编码请求。

让我们将参数`URIEncoding=”UTF-8″`添加到`TOMCAT_ROOT/conf/server.xml`中的所有连接器:

```java
<Connector 
  URIEncoding="UTF-8" 
  port="8080" 
  redirectPort="8443" 
  connectionTimeout="20000" 
  protocol="HTTP/1.1"/>

<Connector 
  URIEncoding="UTF-8" 
  port="8009" 
  redirectPort="8443" 
  protocol="AJP/1.3"/>
```

## 3。字符集过滤器

配置完连接器后，就该强制 web 应用程序处理 UTF-8 中的所有请求和响应了。

让我们定义一个名为`CharacterSetFilter`的类:

```java
public class CharacterSetFilter implements Filter {

    // ...

    public void doFilter(
      ServletRequest request, 
      ServletResponse response, 
      FilterChain next) throws IOException, ServletException {
        request.setCharacterEncoding("UTF-8");
        response.setContentType("text/html; charset=UTF-8");
        response.setCharacterEncoding("UTF-8");
        next.doFilter(request, response);
    }

    // ...
} 
```

我们需要将过滤器添加到应用程序的`web.xml`中，这样它就可以应用于所有的请求和响应:

```java
<filter>
    <filter-name>CharacterSetFilter</filter-name>
    <filter-class>com.baeldung.CharacterSetFilter</filter-class>
</filter>

<filter-mapping>
    <filter-name>CharacterSetFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

## 4。服务器页面编码

我们需要配置的 web 应用程序的另一部分是 Java 服务器页面。

确保服务器页面中 UTF-8 的最好方法是在每个 JSP 页面的顶部添加这个标签:

```java
<%@page pageEncoding="UTF-8" contentType="text/html; charset=UTF-8"%>
```

## 5。HTML 页面编码

服务器页面编码告诉 JVM 如何处理页面字符，而 HTML 页面编码告诉浏览器如何处理页面字符。

我们应该在所有 HTML 页面的`head`部分添加这个`<meta>`标签:

```java
<meta http-equiv='Content-Type' content='text/html; charset=UTF-8' />
```

## 6。MySQL 服务器配置

现在，我们的 Tomcat 已经配置好了，是时候配置数据库了。

我们假设使用了 MySQL 服务器。配置文件在 Windows 上被命名为`my.ini`，在 Linux 上被命名为`my.cnf`。

我们需要找到配置文件，搜索这些参数，并相应地编辑它们:

```java
[client]
default-character-set = utf8mb4

[mysql]
default-character-set = utf8mb4

[mysqld]
character-set-client-handshake = FALSE
character-set-server = utf8mb4
collation-server = utf8mb4_unicode_ci
```

我们需要重启 MySQL 服务器以使更改生效。

## 7。MySQL 数据库配置

MySQL 服务器字符集配置只适用于新数据库。我们需要手动迁移旧的。这可以通过几个命令轻松实现。

对于每个数据库:

```java
ALTER DATABASE database_name CHARACTER SET = utf8mb4 
    COLLATE = utf8mb4_unicode_ci;
```

对于每个表:

```java
ALTER TABLE table_name CONVERT TO 
    CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

对于每个`VARCHAR`或`TEXT`列:

```java
ALTER TABLE table_name CHANGE column_name column_name 
    VARCHAR(69) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

如果我们在数据库查询中传递带有 UTF-8 字符的数据，我们需要确保所建立的任何数据库连接都符合 UTF-8 编码。

对于基于 JDBC 的连接，这可以通过以下连接 URL 实现:

```java
jdbc:mysql://localhost:3306/?useUnicode=yes;characterEncoding=UTF-8
```

## 8。结论

在本文中，我们演示了如何确保 Tomcat 使用 UTF-8 编码。