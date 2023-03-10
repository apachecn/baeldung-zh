# WildFly 管理远程访问

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/wildfly-remote-access>

## 1.介绍

WildFly 为服务器管理提供了不同的方法。最常见的方法是使用它的 web 界面，但是我们也可以使用 CLI 或 XML 脚本。

在本教程中，我们将重点介绍访问管理 web 界面的 **。**

我们假设读者已经理解了标准的[野火设置](/web/20221002161029/https://www.baeldung.com/wildfly-server-setup)流程。

## 2.远程存取

web 界面或控制台是一个 GWT 应用程序，它使用 **WildFly 的 HTTP 管理 API 来配置独立或域管理的服务器**。这个 API 服务于两种不同的上下文:

*   `Web interface`:[`http://<host>:9990/console`](https://web.archive.org/web/20221002161029/http://%3Chost%3E:9990/management)
*   `Management operations` : `[http://<host>:9990/management](https://web.archive.org/web/20221002161029/http://%3Chost%3E:9990/management)`

默认情况下，**web 控制台只能从本地主机**访问。也就是说，我们的配置文件只包含本地的`interfaces`来与 web 控制台交互。

用 WildFly 的行话来说，一个接口由一个带有选择标准的网络接口组成。大多数情况下，选择标准是接口的绑定 IP 地址。本地接口声明如下:

```java
<interface name="management">
    <inet-address value="${jboss.bind.address.management:127.0.0.1}"/>
</interface>
<!--127.0.0.1 is the localhost IP address. -->
```

因此，这个`management` local 被附加到套接字监听器 **`management-http`** 从端口`9000` : 接收 web 控制台的连接

```java
<socket-binding-group name="standard-sockets" default-interface="public" 
  port-offset="${jboss.socket.binding.port-offset:0}">
    <socket-binding name="ajp" port="${jboss.ajp.port:8009}"/>
    <socket-binding name="http" port="${jboss.http.port:8080}"/>
    <socket-binding name="https" port="${jboss.https.port:8443}"/>
    <socket-binding name="management-http" interface="management" 
      port="${jboss.management.http.port:9990}"/>
    <socket-binding name="management-https" interface="management" 
      port="${jboss.management.https.port:9993}"/>
    <socket-binding name="txn-recovery-environment" port="4712"/>
    <socket-binding name="txn-status-manager" port="4713"/>
    <outbound-socket-binding name="mail-smtp">
       <remote-destination host="localhost" port="25"/>
    </outbound-socket-binding>
</socket-binding-group>
```

为了允许从远程机器访问，我们首先需要在适当的配置文件中创建远程 `management interface` 。如果我们正在配置一个独立的服务器，我们将更改 `standalone/configuration/standalone.xml`，对于域管理的，我们将更改`domain/configuration/host.xml` :

```java
<interface name="remoteManagement">
    <inet-address value="${jboss.bind.address.management:REMOTE_HOST_IP}"/> 
</interface> 
<!--REMOTE_HOST_IP is the remote host IP address. (e.g 192.168.1.2) -->
```

我们还必须修改 `management-http` 的套接字绑定，删除之前的本地接口，添加新的:

```java
<socket-binding-group name="standard-sockets" default-interface="public" 
  port-offset="${jboss.socket.binding.port-offset:0}">
    <!-- same as before -->
    <socket-binding name="management-http" interface="remoteManagement" 
      port="${jboss.management.http.port:9990}"/>
    <socket-binding name="management-https" interface="remoteManagement" 
      port="${jboss.management.https.port:9993}"/>
    <!-- same as before -->
</socket-binding-group>
```

在上面的配置中，我们将新的`remoteManagement `接口绑定到我们的 HTTP (9990)和 HTTPS (9993)端口。它将允许远程主机 IP 通过 HTTP/HTTPS 端口连接到 web 接口。

## 3.证明

默认情况下，WildFly 会保护所有远程连接。默认的安全机制是通过 HTTP 摘要认证的用户名/密码。

**但是，如果我们在向服务器添加用户之前尝试连接到管理控制台，** **，我们将不会得到登录弹出窗口** 的提示。

为了创建用户，WildFly 提供了一个交互式`add-user.sh` (在 Windows 平台上为`add-user.bat`)脚本，包含几个步骤:

1.  `Type of user` : 管理或应用用户
2.  `Realm` : 配置中使用的域名，默认为`ManagementRealm `
3.  `Username` : 新用户的用户名
4.  `Password` : 新用户的密码
5.  `Slave domain controller` : 指示用户是否将控制分布式域架构中的从域进程的标志；默认为`No`

还可以通过使用相同的脚本并将输入指定为参数，以非交互方式添加用户:

```java
$ ./add-user.sh -u 'adminuser1' -p 'password1!'
```

添加密码为“password1！“到默认境界。

## 4.结论

在这个简短的教程中，我们探索了如何设置 WildFly 以允许远程访问服务器的管理 web 控制台。此外，我们还看到了如何使用 WildFly 提供的脚本来创建用户。