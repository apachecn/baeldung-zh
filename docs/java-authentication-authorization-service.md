# Java 认证和授权服务指南(JAAS)

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-authentication-authorization-service>

## 1。概述

[Java 认证和授权服务](https://web.archive.org/web/20220628124807/https://docs.oracle.com/en/java/javase/11/security/java-authentication-and-authorization-service-jaas-reference-guide.html) (JAAS)是一个 Java SE 底层安全框架，**将安全模型从基于代码的安全扩展到基于用户的安全**。我们可以将 JAAS 用于两个目的:

*   身份验证:识别当前运行代码的实体
*   授权:一旦通过身份验证，确保该实体拥有执行敏感代码所需的访问控制权限或许可

在本教程中，我们将介绍如何通过实现和配置各种 API，尤其是`LoginModule`，在示例应用程序中设置 JAAS。

## 2。JAAS 如何运作

在应用程序中使用 JAAS 时，会涉及到几个 API:

*   `CallbackHandler`:用于收集用户凭证，创建`LoginContext`时可选提供
*   `Configuration`:负责加载`LoginModule`实现，可以在`LoginContext`创建时可选提供
*   `LoginModule`:有效用于认证用户

我们将使用默认的`Configuration` API 实现，并为`CallbackHandler` 和`LoginModule`API 提供我们自己的实现。

## 3。提供`CallbackHandler`实施

在深入研究`LoginModule`实现之前，我们首先需要**为`CallbackHandler`接口提供一个实现，该接口用于收集用户凭证**。

它有一个方法`handle()`，接受一个`Callback`数组。此外，JAAS 已经提供了许多`Callback`实现，我们将使用`NameCallback`和`PasswordCallback`分别收集用户名和密码。

让我们看看我们的`CallbackHandler`接口的实现:

```java
public class ConsoleCallbackHandler implements CallbackHandler {

    @Override
    public void handle(Callback[] callbacks) throws UnsupportedCallbackException {
        Console console = System.console();
        for (Callback callback : callbacks) {
            if (callback instanceof NameCallback) {
                NameCallback nameCallback = (NameCallback) callback;
                nameCallback.setName(console.readLine(nameCallback.getPrompt()));
            } else if (callback instanceof PasswordCallback) {
                PasswordCallback passwordCallback = (PasswordCallback) callback;
                passwordCallback.setPassword(console.readPassword(passwordCallback.getPrompt()));
            } else {
                throw new UnsupportedCallbackException(callback);
            }
        }
    }
}
```

因此，为了提示和读取用户名，我们使用了:

```java
NameCallback nameCallback = (NameCallback) callback;
nameCallback.setName(console.readLine(nameCallback.getPrompt()));
```

同样，要提示并读取密码:

```java
PasswordCallback passwordCallback = (PasswordCallback) callback;
passwordCallback.setPassword(console.readPassword(passwordCallback.getPrompt()));
```

稍后，我们将看到在实现`LoginModule`时如何调用`CallbackHandler`。

## 4。提供`LoginModule`实施

为了简单起见，我们将提供一个存储硬编码用户的实现。所以，姑且称之为`InMemoryLoginModule`:

```java
public class InMemoryLoginModule implements LoginModule {

    private static final String USERNAME = "testuser";
    private static final String PASSWORD = "testpassword";

    private Subject subject;
    private CallbackHandler callbackHandler;
    private Map<String, ?> sharedState;
    private Map<String, ?> options;

    private boolean loginSucceeded = false;
    private Principal userPrincipal;
    //...
}
```

在接下来的小节中，我们将给出更重要的方法的实现:`initialize()`、`login()`和`commit()`。

### 4.1。`initialize()`

**首先加载`LoginModule`，然后用`Subject`和`CallbackHandler`** 初始化。此外，`LoginModule`可以使用一个`Map`在它们之间共享数据，另一个`Map`用于存储私有配置数据:

```java
public void initialize(
  Subject subject, CallbackHandler callbackHandler, Map<String, ?> sharedState, Map<String, ?> options) {
    this.subject = subject;
    this.callbackHandler = callbackHandler;
    this.sharedState = sharedState;
    this.options = options;
}
```

### 4.2。`login()`

在`login()`方法中，我们用一个`NameCallback`和一个`PasswordCallback`调用`CallbackHandler.handle()`方法来提示并获取用户名和密码。然后，我们将这些提供的凭证与硬编码的凭证进行比较:

```java
@Override
public boolean login() throws LoginException {
    NameCallback nameCallback = new NameCallback("username: ");
    PasswordCallback passwordCallback = new PasswordCallback("password: ", false);
    try {
        callbackHandler.handle(new Callback[]{nameCallback, passwordCallback});
        String username = nameCallback.getName();
        String password = new String(passwordCallback.getPassword());
        if (USERNAME.equals(username) && PASSWORD.equals(password)) {
            loginSucceeded = true;
        }
    } catch (IOException | UnsupportedCallbackException e) {
        //...
    }
    return loginSucceeded;
}
```

**`login()`方法应该为成功操作返回`true`，为失败登录返回`false`**。

### 4.3。`commit()`

如果对`LoginModule#login`的所有调用成功，我们**用一个附加的`Principal`** 更新`Subject`:

```java
@Override
public boolean commit() throws LoginException {
    if (!loginSucceeded) {
        return false;
    }
    userPrincipal = new UserPrincipal(username);
    subject.getPrincipals().add(userPrincipal);
    return true;
}
```

否则，调用`abort()`方法。

此时，我们的`LoginModule`实现已经准备好，需要进行配置，以便可以使用`Configuration`服务提供者动态加载它。

## 5。`LoginModule`配置

JAAS 使用`Configuration`服务提供者在运行时加载`LoginModule`。默认情况下，它提供并使用`ConfigFile`实现，其中`LoginModule`是通过登录文件配置的。例如，下面是我们的`LoginModule`使用的文件的内容:

```java
jaasApplication {
   com.baeldung.jaas.loginmodule.InMemoryLoginModule required debug=true;
};
```

正如我们所见，**我们已经提供了`LoginModule`实现**的完全限定类名、一个`required`标志和一个调试选项。

最后，注意我们也可以通过`java.security.auth.login.config`系统属性指定登录文件:

```java
$ java -Djava.security.auth.login.config=src/main/resources/jaas/jaas.login.config
```

我们还可以通过 Java 安全文件`${java.home}/jre/lib/security/java.security`中的属性`login.config.url`指定一个或多个登录文件:

```java
login.config.url.1=file:${user.home}/.java.login.config
```

## 6。认证

首先，**应用程序通过创建一个 [`LoginContext`](https://web.archive.org/web/20220628124807/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/javax/security/auth/login/LoginContext.html) 实例**来初始化认证过程。为此，我们可以看一下完整的构造函数，以了解我们需要什么参数:

```java
LoginContext(String name, Subject subject, CallbackHandler callbackHandler, Configuration config)
```

*   `name`:用作仅加载相应`LoginModule` s 的索引
*   `subject`:表示想要登录的用户或服务
*   `callbackHandler`:负责将用户凭证从应用程序传递到`LoginModule`
*   `config`:负责加载 name 参数对应的`LoginModule`

这里，我们将使用重载的构造函数，在这里我们将提供我们的`CallbackHandler`实现:

```java
LoginContext(String name, CallbackHandler callbackHandler)
```

现在我们有了一个`CallbackHandler`和一个已配置的`LoginModule`、**，我们可以通过初始化一个`LoginContext`对象**来开始认证过程:

```java
LoginContext loginContext = new LoginContext("jaasApplication", new ConsoleCallbackHandler());
```

此时，**我们可以调用`login()`方法来认证用户**:

```java
loginContext.login();
```

反过来，`login()`方法创建我们的`LoginModule`的一个新实例，并调用它的`login()`方法。并且，**认证成功后，我们可以检索已认证的`Subject`** :

```java
Subject subject = loginContext.getSubject();
```

现在，让我们运行一个连接了`LoginModule`的示例应用程序:

```java
$ mvn clean package
$ java -Djava.security.auth.login.config=src/main/resources/jaas/jaas.login.config \
    -classpath target/core-java-security-2-0.1.0-SNAPSHOT.jar com.baeldung.jaas.JaasAuthentication
```

当系统提示我们提供用户名和密码时，我们将使用`testuser`和`testpassword`作为凭证。

## 7。授权

当用户第一次连接并关联到`AccessControlContext`时，授权开始生效。**使用 Java 安全策略，我们可以向`Principal`授予一个或多个访问控制权限。然后我们可以通过调用`SecurityManager#checkPermission`方法:**来阻止对敏感代码的访问

```java
SecurityManager.checkPermission(Permission perm)
```

### 7.1.定义权限

访问控制权限或**许可是对资源**执行动作的能力。我们可以通过子类化`Permission`抽象类来实现权限。为此，我们需要提供一个资源名称和一组可能的操作。例如，我们可以使用`FilePermission`来配置文件的访问控制权限。可能的动作有`read`、`write`、`execute`等等。对于不需要采取行动的情况，我们可以简单地使用`BasicPermision`。

接下来，我们将通过`ResourcePermission`类提供一个权限实现，其中用户可能拥有访问资源的权限:

```java
public final class ResourcePermission extends BasicPermission {
    public ResourcePermission(String name) {
        super(name);
    }
}
```

稍后，我们将通过 Java 安全策略为这个权限配置一个条目。

### 7.2.授予权限

通常，我们不需要知道策略文件的语法，因为我们总是可以使用[策略工具](https://web.archive.org/web/20220628124807/https://docs.oracle.com/javase/8/docs/technotes/guides/security/PolicyGuide.html)来创建一个。让我们看一下我们的策略文件:

```java
grant principal com.sun.security.auth.UserPrincipal testuser {
    permission com.baeldung.jaas.ResourcePermission "test_resource"
};
```

在这个示例中，**我们将`test_resource`权限授予了`testuser`用户**。

### 7.3.检查权限

一旦`Subject`被认证并且权限被配置，**我们可以通过调用`Subject#doAs`或`Subject#doAsPrivilieged`静态方法**来检查访问。为此，我们将提供一个`PrivilegedAction`来保护对敏感代码的访问。在`run()`方法中，我们调用`SecurityManager#checkPermission`方法来确保被认证的用户拥有`test_resource`权限:

```java
public class ResourceAction implements PrivilegedAction {
    @Override
    public Object run() {
        SecurityManager sm = System.getSecurityManager();
        if (sm != null) {
            sm.checkPermission(new ResourcePermission("test_resource"));
        }
        System.out.println("I have access to test_resource !");
        return null;
    }
}
```

最后一件事是调用`Subject#doAsPrivileged`方法:

```java
Subject subject = loginContext.getSubject();
PrivilegedAction privilegedAction = new ResourceAction();
Subject.doAsPrivileged(subject, privilegedAction, null);
```

与身份验证一样，我们将运行一个简单的授权应用程序，其中除了`LoginModule`之外，我们还提供了一个权限配置文件:

```java
$ mvn clean package
$ java -Djava.security.manager -Djava.security.policy=src/main/resources/jaas/jaas.policy \
    -Djava.security.auth.login.config=src/main/resources/jaas/jaas.login.config \
    -classpath target/core-java-security-2-0.1.0-SNAPSHOT.jar com.baeldung.jaas.JaasAuthorization
```

## 8。结论

在本文中，我们通过探索主要的类和接口并展示如何配置它们，展示了如何实现 JAAS。特别是，我们实现了一个服务提供者`LoginModule`。

和往常一样，这篇文章中的代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220628124807/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-security-2)