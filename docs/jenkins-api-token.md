# 如何为 Jenkins 获取 API 令牌

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/ops/jenkins-api-token>

## 1.概观

[Jenkins](/web/20221229072807/https://www.baeldung.com/linux/jenkins-install-run) 是一个强大的自动化服务器，组织广泛使用它来自动化各种任务，例如[构建和部署软件](/web/20221229072807/https://www.baeldung.com/category/devops)。此外，Jenkins 还提供了一个 API 令牌来认证用户，并授予他们访问[Jenkins API](https://web.archive.org/web/20221229072807/https://www.jenkins.io/doc/book/using/remote-access-api/)的权限。此外，使用 API 令牌，我们可以创建、管理、查询和访问构建工件。

在本教程中，我们将学习在 Jenkins 中生成和使用 API 令牌。

## 2.生成 API 令牌

为了访问 Jenkins API，用户需要使用 API 令牌进行身份验证。API 令牌用作与 Jenkins 进行身份验证的标识符。它通常用于允许第三方应用程序或脚本与 Jenkins 进行交互，例如触发构建或访问构建工件。 **Jenkins 管理员可以创建和管理访问令牌，以授予对特定 API 资源的访问权限。**

为了在 Jenkins 中生成 API 令牌，我们需要遵循以下步骤:

*   以管理员身份登录 Jenkins 实例
*   点击詹金斯仪表盘中的 *管理詹金斯*
*   点击 *管理用户*
*   选择我们想要为其生成 API 令牌的用户，并点击其名称以访问其用户配置页面
*   使用用户配置页面的 *添加新令牌* 部分生成令牌
*   点击 *复制* 按钮将令牌复制到剪贴板
*   `Save`配置

整个过程可以在下面的 GUI 中看到:

[![](img/a0f0dd36784de4b78669a10af25bce04.png)](/web/20221229072807/https://www.baeldung.com/wp-content/uploads/2022/12/tokenKey.png)

一旦我们获得了 API 令牌，我们就可以用它来验证对 Jenkins 的请求。当访问 Jenkins remote API 时，我们可以使用令牌来触发构建或访问构建工件。此外，重要的是要保证 API 令牌的安全，不要与任何无权访问我们的 Jenkins 帐户的人共享它。**建议定期轮换我们的 API 令牌，以降低** **的安全风险。**

## 3.使用 API 令牌

到目前为止，我们已经成功地为一个 Jenkins 用户生成了 API 令牌。我们可以使用 Jenkins 远程访问 API 来触发作业或更新作业的配置。这对于自动化配置过程或者将 Jenkins 与其他系统集成非常有用。

为了演示，我们将通过向 Jenkins 服务器的`/build`端点发送 HTTP POST 请求来触发构建。该请求将包括 API 令牌以及其他参数。

让我们看看使用 API 令牌密钥触发 Jenkins 作业的请求:

```java
$ curl -X POST http://11.223.231.112:8080/job/testJob/build --user testuser:1100a338c975eb40189c3fe2cf580b2bdf
```

在上面运行远程作业的请求中，我们为`Jenkins URL`、`job name`和`user name`提供了一个访问令牌密钥。

## 4.结论

在本文中，我们演示了如何在 Jenkins 中生成和使用 API 令牌。

首先，我们学习了为 Jenkins 用户生成 API 令牌。之后，我们使用同一个令牌密钥远程触发了一个作业。