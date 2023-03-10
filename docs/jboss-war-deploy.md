# 在 JBoss 中部署 WAR 文件

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jboss-war-deploy>

## 1.介绍

在本教程中，我们将了解如何在 JBoss 服务器上部署 war 文件。

我们可以通过手动将文件放在合适的目录中或者直接从 Eclipse 部署 war 文件。

## 2.手动部署 WAR 文件

如果我们已经有了 war 文件，并且想要在 JBoss 上部署它，**我们可以转到位于`standalone/deployments`的 JBoss 安装目录，并将文件粘贴到那里**。

部署有两种工作模式:

*   **手动:**部署扫描程序不会尝试直接监控部署文件夹。相反，扫描仪依赖于标记文件。用户添加的标记文件充当一种命令，告诉扫描器部署内容。
*   **auto:** 扫描器将直接监控部署文件夹，自动部署新内容并重新部署时间戳已更改的内容。

我们可以通过将`auto-deploy-zipped `属性的值设置为`true`或`false:`来指定配置文件`standalone.xml `中的模式

```java
<deployment-scanner 
  name="default" 
  path="deployments" 
  scan-enabled="true" 
  scan-interval="5000" 
  relative-to="jboss.server.base.dir" 
  auto-deploy-zipped="true" 
  deployment-timeout="60"/>
```

默认情况下，该值为`true`。因此，每当我们将一个 war 文件放在部署文件夹中时，它就会被自动部署。JBoss 会自动创建`.deployed`标记文件，表示内容已经部署。

但是，如果我们在将新的 war 文件复制到部署文件夹之前删除了以前的部署，JBoss 将创建一个`.undeployed`标记文件，表明部署已经被删除。在这种情况下，我们需要手动删除标记文件，以便开始部署。

如果`auto-deploy-zipped` 的值被设置为`false`，我们将需要手动创建`.deployed`标记文件来启动部署。

## 3.使用 Eclipse 进行部署

我们可以**在 Eclipse 中创建一个动态 web 项目，添加一个 JBoss 服务器，然后配置应用程序在服务器**上运行。在内部，Eclipse 将创建应用程序的 war 文件，并将其放在 JBoss 目录中。我们可以创建一个`index.html`文件，并设置 `web.xml`中的`welcome-file `指向它。

为了测试应用程序是否部署成功，我们可以启动 web 浏览器并尝试访问以下格式的 URL:`http://localhost:`<端口号> / <项目名称>

如果我们看到索引页面，说明应用程序部署成功。

## 4.结论

在本文中，我们研究了如何通过使用部署文件夹和 Eclipse 在 JBoss 服务器上部署 war 文件。

我们还讨论了自动和手动部署模式，以及它们如何处理 JBoss 的标记文件。