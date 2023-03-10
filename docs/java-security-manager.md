# Java SecurityManager 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-security-manager>

## 1。概述

在本教程中，我们将了解 Java 的内置安全基础设施，它在默认情况下是禁用的。具体来说，我们将研究它的主要组件、扩展点和配置。

## 2。`SecurityManager`在行动中

这可能是一个惊喜，但是**默认`SecurityManager`设置不允许** **许多标准操作**:

```java
System.setSecurityManager(new SecurityManager());
new URL("http://www.google.com").openConnection().connect();
```

在这里，我们以编程方式使用默认设置启用安全监管，并尝试连接到 google.com。

然后我们得到以下异常:

```java
java.security.AccessControlException: access denied ("java.net.SocketPermission"
  "www.google.com:80" "connect,resolve")
```

在标准库中还有许多其他的用例——例如，读取系统属性、读取环境变量、打开文件、反射和更改语言环境等等。

## 3。用例

这个安全基础设施从 Java 1.0 开始就有了。这是一个小程序——嵌入浏览器的 Java 应用程序——非常普遍的时代。自然，有必要限制他们对系统资源的访问。

如今，小程序已经过时了。然而，当存在第三方代码在受保护环境中执行的情况时，**安全实施仍然是一个实际的概念。**

例如，假设我们有一个 Tomcat 实例，其中第三方客户端可以托管它们的 web 应用程序。我们不希望允许他们执行像`System.exit()`这样的操作，因为那会影响其他应用程序，甚至整个环境。

## 4。设计

### 4.1。安全经理

内置安全基础设施中的一个主要组件是 [`java.lang SecurityManager`](https://web.archive.org/web/20221205174240/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/SecurityManager.html) 。它有几个像`checkConnect`一样的`checkXxx` 方法，在上面的测试中授权我们尝试连接到 Google。所有这些都委托给了`checkPermission(java.security.Permission)`方法。

### 4.2。许可

实例代表授权请求。标准的 JDK 类为所有潜在的危险操作(如读/写文件、打开套接字等)创建它们。)并将它们交给`SecurityManager`进行适当的授权。

### 4.3。配置

我们以特殊的策略格式定义权限。这些权限采用`grant`条目的形式:

```java
grant codeBase "file:${{java.ext.dirs}}/*" {
    permission java.security.AllPermission;
};
```

上面的`codeBase`规则是可选的。我们可以不指定任何字段，或者使用`signedBy`(与密钥库中相应的证书集成在一起)或`principal` (通过`javax.security.auth.Subject`附加到当前线程的`java.security.Principal`)。我们可以使用这些规则的任意组合。

默认情况下，JVM 加载位于<`java.home>/lib/security/java.policy`的通用系统策略文件。如果我们在`<user.home>/.java.policy`中定义了任何用户本地策略，JVM 会将它附加到系统策略中。

也可以通过命令行指定策略文件:–`Djava.security.policy=/my/policy-file`。这样，我们可以将策略附加到先前加载的系统和用户策略中。

替换所有系统和用户策略(如果有)有一个特殊的语法–双等号:–`Djava.security.policy==/my/policy-file`

## 5。示例

让我们定义一个自定义权限:

```java
public class CustomPermission extends BasicPermission {
    public CustomPermission(String name) {
        super(name);
    }

    public CustomPermission(String name, String actions) {
        super(name, actions);
    }
}
```

以及应该受到保护的共享服务:

```java
public class Service {

    public static final String OPERATION = "my-operation";

    public void operation() {
        SecurityManager securityManager = System.getSecurityManager();
        if (securityManager != null) {
            securityManager.checkPermission(new CustomPermission(OPERATION));
        }
        System.out.println("Operation is executed");
    }
}
```

如果我们试图在启用安全管理器的情况下运行它，就会抛出一个异常:

```java
java.security.AccessControlException: access denied
  ("com.baeldung.security.manager.CustomPermission" "my-operation")

    at java.security.AccessControlContext.checkPermission(AccessControlContext.java:472)
    at java.security.AccessController.checkPermission(AccessController.java:884)
    at java.lang.SecurityManager.checkPermission(SecurityManager.java:549)
    at com.baeldung.security.manager.Service.operation(Service.java:10)
```

我们可以用以下内容创建我们的`<user.home>/.java.policy`文件，并尝试重新运行应用程序:

```java
grant codeBase "file:<our-code-source>" {
    permission com.baeldung.security.manager.CustomPermission "my-operation";
};
```

它现在工作得很好。

## 6。结论

在本文中，我们检查了内置的 JDK 安全系统是如何组织的，以及我们如何扩展它。即使目标用例相对较少，了解它也是有好处的。

像往常一样，本文的完整源代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20221205174240/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-security)