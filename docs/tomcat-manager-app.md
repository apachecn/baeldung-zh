# Tomcat 管理器应用程序指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/tomcat-manager-app>

## 1。简介

在本教程中，我们将深入了解 Tomcat 管理器应用程序。

简而言之，Tomcat Manager 应用程序**是一个 web 应用程序，它与 Tomcat 服务器**打包在一起，为我们提供管理部署的 web 应用程序所需的基本功能。

正如我们将要看到的，该应用程序有许多特性和服务。除了允许我们管理部署的应用程序，我们还可以看到服务器及其应用程序的状态和配置。

## 2。安装 Tomcat

在我们深入研究 Tomcat 管理器应用程序之前，我们首先需要安装一个 Tomcat 服务器。

幸运的是，安装 Tomcat 是一个简单的过程。请参考我们的[Apache Tomcat 简介](/web/20220922142729/https://www.baeldung.com/tomcat)指南来获得安装 Tomcat 的帮助。在本教程中，我们将使用最新的 [Tomcat 9 版本](https://web.archive.org/web/20220922142729/https://tomcat.apache.org/download-90.cgi)。

## 3。访问 Tomcat 管理器应用程序

现在，让我们来看看如何使用 Tomcat 管理器应用程序。我们有两个选择——我们可以选择使用基于 web(HTML)的应用程序或基于文本的 web 服务。

基于文本的服务是脚本的理想选择，而 HTML 应用程序是为人类设计的。

**基于网络的应用程序可在:**获得

*   `http[s]://<server>:<port>/manager/html/`

而相应的**文本服务在:**可用

*   `http[s]://<server>:<port>/manager/text/`

然而，在我们能够访问这些服务之前，我们需要配置 Tomcat。默认情况下，只有拥有正确权限的用户才能访问它。

让我们通过编辑`conf/tomcat-users`文件来添加这样的用户:

```java
<tomcat-users>
  <role rolename="manager-gui"/>
  <role rolename="manager-script"/>
  <user username="tomcatgui" password="s3cret" roles="manager-gui"/>
  <user username="tomcattext" password="baeldung" roles="manager-script"/>
</tomcat-users>
```

如我们所见，我们添加了两个新用户:

*   `tomcatgui`–具有`**manager-gui**`角色，可以使用基于网络的应用程序
*   `tomcattext`–具有`**manager-script**`角色，可以使用基于文本的 web 服务

在下一节中，我们将看到如何使用这两个用户来演示 Tomcat Manager 应用程序的功能。

## 4.列出当前部署的应用程序

在本节中，我们将学习如何查看当前部署的应用程序列表。

### 4.1.使用网络

让我们打开[http://localhost:8080/Manager/html/](https://web.archive.org/web/20220922142729/http://localhost:8080/manager/html/)查看 Tomcat Manager App 网页。为此，我们需要认证为`tomcatgui`用户。

登录后，网页会在页面顶部列出所有已部署的应用程序。对于每个应用程序，我们可以看到它是否正在运行、上下文路径以及活动会话的数量。我们还可以使用几个按钮来管理应用程序:

[![Tomcat Manager app list applications](img/49865a59f701a87df326c3eac3a40041.png)](/web/20220922142729/https://www.baeldung.com/wp-content/uploads/2019/10/tomcat-manager-app-e1570014354196.png)

### 4.2.使用文字服务

或者，我们可以使用文本 web 服务列出所有已部署的应用程序。这一次，我们使用`tomcattext`用户发出一个`curl`请求来进行身份验证:

```java
curl -u tomcattext:baeldung http://localhost:8080/manager/text/list
```

就像网页一样，响应显示了所有已部署的应用程序及其当前状态和活动会话的数量。例如，我们可以看到`manager`应用程序正在运行，并且有一个活动会话:

```java
OK - Listed applications for virtual host [localhost]
/:running:0:ROOT
/examples:running:0:examples
/host-manager:running:0:host-manager
/manager:running:1:manager
/docs:running:0:docs
```

## 5.管理应用程序

Tomcat 管理器应用程序允许我们做的一个关键功能是停止、启动和重新加载应用程序。

### 5.1.使用网络

对于 web 应用程序，停止和启动应用程序只需单击网页上的按钮。结果和任何问题都会在页面顶部的消息字段中报告。

### 5.2.使用文字服务

同样，我们可以使用文本服务来停止和启动应用程序。让我们停止然后使用一个`curl`请求启动`examples`应用程序:

```java
curl -u tomcattext:baeldung http://localhost:8080/manager/text/stop?path=/examples
OK - Stopped application at context path [/examples]
```

```java
curl -u tomcattext:baeldung http://localhost:8080/manager/text/start?path=/examples
OK - Started application at context path [/examples]
```

`path`查询参数指示管理哪个应用程序，并且必须匹配应用程序的上下文路径。

我们还可以重新加载应用程序来获取对类或资源的更改。然而，这只适用于解压到一个目录中并且不作为 WAR 文件部署的应用程序。

下面是一个我们如何使用文本服务重新加载`docs`应用程序的例子:

```java
curl -u tomcattext:baeldung http://localhost:8080/manager/text/reload?path=/docs
OK - Reloaded application at context path [/docs]
```

但是请记住，我们只需要单击 reload 按钮就可以在 web 应用程序中实现同样的功能。

## 6.过期会话

除了管理应用程序，我们还可以**管理用户会话**。Tomcat Manager 应用程序显示当前用户会话的详细信息，并允许我们手动终止会话。

### 6.1.通过网络界面

我们可以通过点击所有列出的应用程序的`Sessions`列中的链接来查看当前的用户会话。

在下面的例子中，我们可以看到`manager`应用程序有两个用户会话。它显示会话的持续时间、处于非活动状态的时间以及到期时间(默认为 30 分钟)。

我们也可以通过选择会话并选择`Invalidate selected sessions`来手动销毁会话:

[![Tomcat Manager app user sessions](img/1e542f6c1ce90c3d309ed51cd8888ee1.png)](/web/20220922142729/https://www.baeldung.com/wp-content/uploads/2019/10/tomcat-mgr-app-user-sessions-e1570576465312.png)

在主页上，有一个`Expire sessions`按钮。这还会销毁在指定时间内处于空闲状态的会话。

### 6.2.通过文本网络服务

同样，文本服务的等价物很简单。

为了查看当前用户会话的细节，我们使用我们感兴趣的应用程序的上下文路径调用`session`端点。在本例中，我们可以看到当前有两个用于`manager`应用程序的会话:

```java
curl -u tomcattext:baeldung "http://localhost:8080/manager/text/sessions?path=/manager"
OK - Session information for application at context path [/manager]
Default maximum session inactive interval is [30] minutes
Inactive for [2 - <3] minutes: [1] sessions
Inactive for [13 - <14] minutes: [1] sessions
```

如果我们想要销毁不活动的用户会话，那么我们使用`expire`端点。在本例中，我们终止了`manager`应用程序的非活动时间超过 10 分钟的会话:

```java
curl -u tomcattext:baeldung "http://localhost:8080/manager/text/expire?path=/manager&idle;=10"
OK - Session information for application at context path [/manager]
Default maximum session inactive interval is [30] minutes
Inactive for [5 - <6] minutes: [1] sessions
Inactive for [15 - <16] minutes: [1] sessions
Inactive for [>10] minutes: [1] sessions were expired
```

## 7.部署应用程序

现在我们已经了解了如何管理我们的应用程序，让我们看看如何部署新的应用程序。

首先，下载 Tomcat [示例](https://web.archive.org/web/20220922142729/https://tomcat.apache.org/tomcat-9.0-doc/appdev/sample/) WAR，这样我们就有一个新的应用程序可以部署了。

### 7.1.使用网络

现在，我们有几个选项来使用 web 页面部署我们的新示例 WAR。最简单的方法是**上传示例 WAR 文件并部署它**:

[![Tomcat Manager app war deploy](img/ddf0785dd88c13df48b233e32d5dd942.png)](/web/20220922142729/https://www.baeldung.com/wp-content/uploads/2019/10/tomcat-mgr-app-deploy-uploaded-war-e1570403895563.png)

该战争是用与战争名称匹配的**上下文路径部署的。如果成功，示例应用程序将被部署、启动并显示在应用程序列表中。如果我们跟随上下文路径中的`/sample`链接，我们可以查看我们正在运行的示例应用程序:**

[![Tomcat Manager app sample application](img/c64b699ed143a64c374bab4c2dffad06.png)](/web/20220922142729/https://www.baeldung.com/wp-content/uploads/2019/10/tomcat-mgr-app-sample-app-e1570404145384.png)

为了能够再次部署相同的应用程序，让我们单击`Undeploy`按钮。顾名思义，这将**取消部署应用程序**。请注意，这还会删除已部署应用程序的所有文件和目录。

接下来，我们可以通过指定文件路径来**部署示例 WAR 文件。我们指定 WAR 文件或解压缩目录的文件路径 URI 加上上下文路径。在我们的例子中，示例 WAR 在`/tmp`目录中，我们将上下文路径设置为`/sample`:**

[![Tomcat Manager app path to war](img/f36c4f11c2e39aa30ba97422dfc4c9f6.png)](/web/20220922142729/https://www.baeldung.com/wp-content/uploads/2019/10/tomcat-mgr-app-war-deploy.png)

或者，我们可以指定 XML 部署描述符的**文件路径。这种方法允许我们指定影响应用程序如何部署和运行的附加属性。在下面的例子中，我们正在部署示例 WAR 应用程序，并使其可重新加载。**

注意**部署描述符中指定的任何路径都被忽略**。上下文路径取自部署描述符的文件名。看一看[的共同属性](https://web.archive.org/web/20220922142729/https://tomcat.apache.org/tomcat-9.0-doc/config/context.html#Common_Attributes)来理解为什么，以及所有其他可能属性的描述:

```java
<Context docBase="/tmp/sample.war" reloadable="true" />
```

[![Tomcat Manager app xml deployment descriptor](img/5c2e7bf49c291f59cbebbf74c07f30f0.png)](/web/20220922142729/https://www.baeldung.com/wp-content/uploads/2019/10/tomcat-mgr-app-xml-deploy.png)

### 7.2.使用文字服务

现在让我们看看如何使用文本服务部署应用程序。

首先，让我们**取消部署我们的示例应用程序**:

```java
curl -u tomcattext:baeldung "http://localhost:8080/manager/text/undeploy?path=/sample"
OK - Undeployed application at context path [/sample]
```

为了**再次部署它**，我们指定示例 WAR 文件的上下文路径和位置 URI:

```java
curl -u tomcattext:baeldung "http://localhost:8080/manager/text/deploy?path=/sample&war;=file:/tmp/sample.war"
OK - Deployed application at context path [/sample]
```

此外，我们还可以使用 XML 部署描述符来部署应用程序:

```java
curl -u tomcattext:baeldung "http://localhost:8080/manager/text/deploy?config=file:/tmp/sample.xml"
OK - Deployed application at context path [/sample]
```

## 8.查看 SSL 配置

在我们可以看到任何 SSL 配置之前，我们需要在 Tomcat 中**启用 SSL。首先，让我们在 Tomcat 的`conf`目录中创建一个新的带有自签名证书的证书密钥库:**

```java
keytool -genkey -alias tomcat -keyalg RSA -keystore conf/localhost-rsa.jks 
```

接下来，我们更改`conf/tomcat-server.xml`文件以在 Tomcat 中启用 SSL 连接器:

```java
<Connector port="8443" protocol="org.apache.coyote.http11.Http11NioProtocol"
           maxThreads="150" SSLEnabled="true">
    <SSLHostConfig>
        <Certificate certificateKeystoreFile="conf/localhost-rsa.jks" type="RSA" />
    </SSLHostConfig>
</Connector>
```

一旦我们重新启动 Tomcat，我们发现它可以安全地运行在端口 8443 上！

### 8.1.使用网络

我们再打开[https://localhost:8443/Manager/html](https://web.archive.org/web/20220922142729/https://localhost:8443/manager/html)看看 Tomcat Manager App。看起来应该完全一样。

我们现在可以使用`Diagnostics`下的按钮查看我们的 SSL 配置:

[![Tomcat Manager app TLS buttons](img/414c60c49e88d2c876c3c16ed27f84cd.png)](/web/20220922142729/https://www.baeldung.com/wp-content/uploads/2019/10/tomcat-mgr-app-tls-e1570522151519.png)

*   `Ciphers`按钮显示了 Tomcat 理解的所有 SSL 密码
*   接下来，`Certificates`按钮显示了我们的自签名证书的详细信息
*   最后，`Trusted Certificates`按钮显示可信 CA 证书详情；在我们的示例中，它没有显示任何感兴趣的内容，因为我们没有添加任何可信的 CA 证书

此外，SSL 配置文件可以随时动态重新加载。我们可以通过输入主机名来重新加载每个虚拟主机。否则，将重新读取所有配置:

[![Tomcat manager app re-load SSL](img/d4d3d731fc0da34a6b592982db0b5133.png)](/web/20220922142729/https://www.baeldung.com/wp-content/uploads/2019/10/tomcat-mgr-app-ssl-reload-e1570583317405.png)

### 8.2.使用文字服务

同样，我们可以使用文本服务获得相同的信息。我们可以查看所有:

*   使用`sslConnectorCiphers`资源的 SSL 密码:

```java
curl -ku tomcattext:baeldung "https://localhost:8443/manager/text/sslConnectorCiphers"
```

*   使用`sslConnectorCerts`资源的证书:

```java
curl -ku tomcattext:baeldung "https://localhost:8443/manager/text/sslConnectorCerts"
```

*   使用`sslConnectorTrustedCerts`资源的可信证书:

```java
curl -ku tomcattext:baeldung "https://localhost:8443/manager/text/sslConnectorTrustedCerts"
```

SSL **配置可以通过以下方式重新加载**:

```java
curl -ku tomcattext:baeldung "https://localhost:8443/manager/text/sslReload"
OK - Reloaded TLS configuration for all TLS virtual hosts 
```

注意`curl`命令中的`-k`选项，因为我们使用的是自签名证书。

## 9.查看服务器状态

Tomcat Manager 应用程序还向我们显示了服务器的**状态和部署的应用程序**。当我们想要查看总体使用统计时，这些页面特别方便。

如果我们点击右上角显示的`Server Status`链接，我们可以看到服务器的详细信息。`Complete Server Status`链接显示了应用程序的更多详细信息:

[![Tomcat Manager app server status](img/a09d3d05bcdf3eba4ae941c28d9e92e0.png)](/web/20220922142729/https://www.baeldung.com/wp-content/uploads/2019/10/tomcat-mgr-app-server-status-e1570573074460.png)

没有相应的文字服务。但是，我们可以修改 [`Server Status`链接](https://web.archive.org/web/20220922142729/https://localhost:8443/manager/status?XML=true)来查看 XML 中的服务器状态。不幸的是，对 [`Complete Server Status`链接](https://web.archive.org/web/20220922142729/https://localhost:8443/manager/status/all?XML=true)做同样的事情可能行得通，也可能行不通，这取决于我们使用的 Tomcat 版本。

## 10.保存配置

文本服务允许我们将当前配置保存到 Tomcat `conf/server.xml`中。如果我们已经更改了配置并希望保存它以备后用，这将非常有用。

令人欣慰的是，这也备份了以前的`conf/server.xml`，尽管任何以前的注释都可能在新的 `conf/server.xml`配置文件中被删除。

然而，在我们这样做之前，我们需要添加一个新的侦听器。编辑`conf/server.xml`并将以下内容添加到现有监听器列表的末尾:

```java
<Listener className="org.apache.catalina.storeconfig.StoreConfigLifecycleListener" />
```

重新启动 Tomcat 后，我们可以使用以下命令保存我们的配置:

```java
curl -u tomcattext:baeldung "http://localhost:8080/manager/text/save"
OK - Server configuration saved
```

## 11.诊断学

最后，让我们看看 Tomcat 管理器应用程序提供的其他诊断特性。

### 11.1.线程转储

我们可以使用文本服务来获取正在运行的 Tomcat 服务器的**线程转储:**

```java
curl -u tomcattext:baeldung "http://localhost:8080/manager/text/threaddump"
OK - JVM thread dump
2019-10-06 23:19:10.066
Full thread dump Java HotSpot(TM) 64-Bit Server VM (11.0.3+12-LTS mixed mode):
...
```

当我们需要分析或查找导致性能问题的线程(例如长时间运行或死锁的线程)时，这尤其有用。

### 11.2.查找内存泄漏

Tomcat 通常在防止内存泄漏方面做得很好。但是当我们怀疑内存泄漏时，Tomcat 管理器应用程序有一个内存泄漏检测服务来帮助我们。它执行完整的垃圾收集，并检测自上次应用程序重新加载以来仍然驻留在内存中的任何类。

我们只需要运行网页上的`Find Leaks`按钮来检测泄漏。

类似地，文本服务可以运行内存泄漏检测:

```java
curl -u  tomcattext:baeldung "http://localhost:8080/manager/text/findleaks?statusLine=true"
OK - No memory leaks found
```

### 11.3.显示可用资源

文本服务提供了可用资源的列表。在本例中，我们看到有一个可用的内存数据库:

```java
curl -u tomcattext:baeldung "http://localhost:8080/manager/text/resources"
OK - Listed global resources of all types
UserDatabase:org.apache.catalina.users.MemoryUserDatabase
```

## 12。结论

在本文中，我们详细介绍了 Tomcat 管理器应用程序。我们从安装应用程序开始，看看如何通过为两个不同的用户配置权限来授予访问权。

然后我们探索了几个使用基于 web 的应用程序和基于文本的 web 服务的例子。我们看到了如何使用各种方法来查看、管理和部署应用程序。然后我们看了一下如何查看服务器的配置和状态。

要了解关于 Tomcat 管理器应用程序的更多信息，请查看在线文档。