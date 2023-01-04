# Eclipse 中的 Tomcat 配置

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/eclipse-tomcat>

## 1.概观

web 开发的关键挑战之一是在 web 服务器上运行时能够有效地进行调试。由于构建、打包和部署会耗费大量时间，这可能很难实现。

幸运的是，Eclipse 允许我们在 IDE 本身中启动服务器，节省了构建和打包时间。此外，通过在调试模式下启动服务器来帮助我们调查问题。

在这个快速教程中，**我们将看到如何在 Eclipse** 中配置 Tomcat 服务器来实现这一点。

## 2.在 Eclipse 中定义服务器

在 Eclipse 中配置 Tomcat 之前，我们必须先[安装](https://web.archive.org/web/20221129020407/https://tomcat.apache.org/)它。

现在，让我们从使用`File > New > Other:`调用 Eclipse 中的`New Server`向导开始

![Define Server Image 1](img/a8e81915c458e9f3411fb9c52010fe9e.png)

点击`Next `将把我们带到可以选择 Tomcat 版本的窗口。在这里，我们选择了`version 9.0:`

![Define Server Image 2](img/fec1f32ea5eb8cdcb8a172dba6a3caf7.png)

向导将默认服务器名称为`localhost`，服务器名称为`Tomcat v9.0 Server at localhost. `

我们将看到，当我们第一次在 Eclipse 中添加 Tomcat 服务器时，向导会要求我们配置服务器运行时环境:

![Define Server Image 3](img/704b2fa25a7bade4aa19556b9b8f486b.png)

这里，我们将指定 Tomcat 安装目录的位置。此外，我们将为 Tomcat 服务器指定 JRE。

如果我们单击`Next`，Eclipse 将允许我们添加要在服务器上部署的 web 应用程序。但是，让我们在后面的章节中讨论这个问题，然后点击`Finish `。

现在我们可以在`Project Explorer `和`Server `视图中看到新的服务器。

## 3.配置服务器

在`Project Explorer`中，我们将看到通常的 tomcat 服务器配置文件，例如`server.xml, tomcat-users.xml etc.`

此外，如果我们双击`Tomcat v9.0 Server at localhost, `，我们可以使用提供的 UI 配置服务器:

![config](img/aec6af05dda5e53ede3eb96609173115.png)

在此屏幕上，我们可以配置:

*   `server name`–这是将出现在服务器视图中的名称
*   `configuration path`–这是我们在`Project Explorer`中看到的文件所在的位置
*   `server location`–这是我们配置服务器安装位置的地方。此外，我们可以在这里设置应用程序部署位置
*   `module publishing`–这是我们配置 web 模块发布方式的地方
*   `timeouts` –这些是启动/停止服务器的超时时间
*   `ports` –在这里我们可以设置各种服务器端口
*   `MIME mappings`–这些是各种 MIME 类型映射
*   `server launch configuration `–在这里我们可以配置虚拟机参数、类路径等。
*   `server options `–在这里，我们可以启用/禁用安全、默认自动重新加载模块等功能。

## 4.向服务器添加应用程序

我们现在可以在这个服务器上部署我们的 web 应用程序了。因此，在添加它们之前，我们必须确保为项目启用了`Dynamic Web Module` facet。

因此，让我们在`Servers `视图中右键单击 tomcat 服务器，并选择`Add and Remove… `菜单项。然后，在接下来的屏幕上，我们将选择`spring-rest `网络模块:

![6](img/c7c0276a5ddd5166d7baa45748686c12.png)

最后，如果我们现在点击`Finish`，我们将在`Servers `视图中看到`spring-rest `。

## 5.运行服务器

现在剩下要做的就是启动 tomcat 服务器。然后，当服务器启动时，我们将在`Console `视图中看到服务器日志。

请记住，如果服务器超时非常低，服务器可能无法启动。因此，我们可以通过在上面看到的配置屏幕上增加服务器启动超时来解决这个问题。

**需要注意的是，eclipse 不会将应用程序发布到服务器的``webapps``文件夹中。**它会将这个 web 应用程序部署到一个临时文件夹中。因此，不修改 Tomcat 安装。如果我们不更改配置，Eclipse 会将应用程序发布到工作区文件夹:

```
<workspace>/.metadata/.plugins/org.eclipse.wst.server.core/tmp0/wtpwebapps
```

现在，Eclipse 将继续监控我们的源代码并寻找代码变更。然后，我们可以将这些更改与服务器同步，以便在服务器上部署最新的代码。

## 6.结论

在本教程中，我们看到了如何在 Eclipse IDE 中部署 web 应用程序。

这有助于我们避免显式地构建、打包和部署应用程序，从而节省我们宝贵的开发时间，可以更有效地利用这些时间。