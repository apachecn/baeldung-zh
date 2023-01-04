# 用 Java 测试 LDAP 连接

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-test-ldap-connections>

## 1.概观

在本教程中，我们将创建一个 [CLI](https://web.archive.org/web/20221206230431/https://baeldung.com/java-run-jar-with-arguments) 应用程序来测试到任何 [LDAP 认证](/web/20221206230431/https://www.baeldung.com/java-ldap-auth)服务器的连接。**我们不会使用 LDAP 来保护我们的应用程序**，因为例如使用 [Spring Security LDAP](/web/20221206230431/https://www.baeldung.com/spring-security-ldap) 可以做得更好。

即使在开发使用 LDAP 连接的应用程序之前，拥有一个快速检查 LDAP 连接有效性的工具也是非常有用的。在开发应用程序之间的某种集成时，尤其是在设置阶段，它也很有用。**和** **我们将使用核心 Java 类来实现。因此不需要额外的依赖关系**。

## 2.LDAP Java 客户端

让我们从创建我们唯一的类`LdapConnectionTool`开始。我们将从`main`方法开始。为了简单起见，我们所有的逻辑都放在这里:

```java
public class LdapConnectionTool {
    public static void main(String[] args) {
        // ...
    }
} 
```

首先，我们将把我们的参数作为[系统属性](/web/20221206230431/https://www.baeldung.com/java-system-get-property-vs-system-getenv)来传递。我们将对`factory` ( `LdapCtxFactory)` )和`authType` ( `simple`)变量使用默认值。`LdapCtxFactory`是核心 Java 类，负责连接服务器和填充用户属性的整个过程。`simple`认证类型意味着我们的密码将以明文形式发送。类似地，我们将把我们的`query`变量默认为`user,` ，这样我们可以指定一个或两个。我们稍后会看到使用细节:

```java
String factory = System.getProperty("factory", "com.sun.jndi.ldap.LdapCtxFactory");
String authType = System.getProperty("authType", "simple");
String url = System.getProperty("url");
String user = System.getProperty("user");
String password = System.getProperty("password");
String query = System.getProperty("query", user);
```

接下来，我们将创建我们的环境地图，它包含使用`InitialDirContext`进行连接所需的所有属性:

```java
Hashtable<String, String> env = new Hashtable<String, String>();
env.put(Context.INITIAL_CONTEXT_FACTORY, factory);
env.put(Context.SECURITY_AUTHENTICATION, authType);
env.put(Context.PROVIDER_URL, url);
```

**我们不想要求用户和密码，因为有些服务器允许匿名访问**:

```java
if (user != null) {
    env.put(Context.SECURITY_PRINCIPAL, user);
    env.put(Context.SECURITY_CREDENTIALS, password);
}
```

在测试连接时，我们通常会传入一个不正确的 URL，或者服务器没有响应。**由于默认客户端行为无限期阻塞，直到收到响应，我们将定义超时参数**。等待时间以毫秒为单位定义:

```java
env.put("com.sun.jndi.ldap.read.timeout", "5000");
env.put("com.sun.jndi.ldap.connect.timeout", "5000");
```

之后，我们尝试建立与新实例`InitialDirContext`的连接，以及基本的异常处理。这很重要，因为我们将使用它来诊断常见问题。同样，由于我们正在开发一个 CLI 应用程序，我们将消息打印到标准输出中:

```java
DirContext context = null;
try {
    context = new InitialDirContext(env);
    System.out.println("success");
    // ...
} catch (NamingException e) {
    System.out.println(e.getMessage());
} finally {
    context.close();
}
```

**最后，我们使用我们的`context`变量来查询由可选的`query`** 产生的所有属性:

```java
if (query != null) {
    Attributes attributes = context.getAttributes(query);
    NamingEnumeration<? extends Attribute> all = attributes.getAll();
    while (all.hasMoreElements()) {
        Attribute next = all.next();

        String key = next.getID();
        Object value = next.get();

        System.out.println(key + "=" + value);
    }
}
```

## 3.常见错误

在本节中，我们将回顾在尝试连接到服务器时遇到的一些常见错误和错误消息:

*   错误的基本 DN:如果我们没有正确设置基本 DN，我们将得到“错误代码 49-无效凭证”。由于每个服务器都有自己的结构，我们应该首先检查这一点，因为这个消息可能会误导。
*   没有匿名连接:如果我们不配置我们的服务器允许匿名访问，我们会得到错误“ERR_229 无法验证用户”。

## 4.使用

现在我们都设置好了，我们可以使用我们的应用程序。首先，让我们[将其构建为一个 jar](/web/20221206230431/https://www.baeldung.com/java-create-jar) ，将其重命名为`ldap-connection-tool.jar,`，然后尝试以下示例之一。**注意，这些值完全取决于我们的服务器配置**。

使用用户和密码连接:

```java
java -cp ldap-connection-tool.jar \
-Durl=ldap://localhost:389 \
-Duser=uid=gauss,dc=baeldung,dc=com \
-Dpassword=password \
com.baeldung.jndi.ldap.connectionTool.LdapConnectionTool
```

仅为快速连接测试指定服务器 URL:

```java
java -cp ldap-connection-tool.jar \
-Durl=ldap://localhost:389 \
com.baeldung.jndi.ldap.connectionTool.LdapConnectionTool
```

此外，指定一个`query`以及一个`user`和`password`，我们可以连接一个特定的用户，但查询另一个用户。这在我们需要以管理员身份连接时非常有用，例如，在执行查询之前。**同样，如果我们与拥有足够权限的用户连接，我们可以看到受保护的属性，如密码**。

最后，在处理简单的参数时，将系统属性作为输入传递是很好的。但是，有更好的方式来开发 CLI 应用程序，比如 [Spring Shell](/web/20221206230431/https://www.baeldung.com/spring-shell-cli) 。对于更复杂的事情，我们应该使用类似的东西。

## 5.结论

在本文中，我们创建了一个 CLI 应用程序，它可以连接到 LDAP 服务器并运行连接测试。**同样，在单元测试**中有更多的应用使用实例。和往常一样，源代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20221206230431/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-jndi)