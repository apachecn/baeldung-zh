# 配置 git 凭据

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/ops/git-configure-credentials>

## 1.介绍

近年来， [git](/web/20221026020020/https://www.baeldung.com/cs/git-vs-svn) 比 subversion 等其他配置管理系统更受欢迎。随着 GitHub 和 GitLab 等免费平台的兴起，**安全地版本化和保存我们的应用程序代码比以往任何时候都容易**。

但是，不断输入凭据可能很麻烦，并且很难创建自动化的 CI/CD 管道。因此，在本教程中，我们将看看如何配置 git 凭证，以避免必须手动输入它们。

## 2.输入凭据

每当远程连接需要认证时， **git 有几种方法来寻找凭证以使用**。

让我们从基础开始，其中没有配置任何凭证。如果 git 需要用户名和密码来访问远程连接，它会采取以下步骤提示用户输入。

首先，它试图调用允许用户输入凭证的应用程序。检查以下值(按顺序)以确定要使用的应用程序:

*   `GIT_ASKPASS`环境变量
*   `core.askPass`配置变量
*   `SSH_ASKPASS`环境变量

如果设置了其中的任何一项，应用程序就会被调用，并且从其标准输出中读取用户的输入。

如果没有设置这些值，git 会回复到提示用户在命令行上输入。

## 3.存储凭据

输入用户名和密码可能很繁琐，尤其是在一天中频繁提交代码的时候。手动输入密码很容易出错，也很难创建自动化管道。

为此，git 提供了几种存储用户名和密码的方法。我们将在接下来的部分中研究每种方法。

### 3.1.URL 中的用户名和密码

一些 git 提供者允许在存储库 URL 中同时嵌入用户名和密码。这可以在我们克隆存储库时完成:

```
git clone https://<username>:<password>@gitlab.com/group/project.git
```

请记住**如果密码有特殊字符，需要对它们进行转义**以防止 shell 试图解释它们。

或者，我们可以编辑存储库中的 git 配置文件，以包含用户名和密码:

```
url = https://<username>:<password>@<code class="language-shell">gitlab.com/group/project.git
```

无论哪种方式，**记住用户名和密码是以纯文本形式存储的，**这样任何可以访问存储库的人都可以看到它们。

### 3.2.凭据上下文

Git 还允许根据上下文配置凭证。以下命令将配置特定的 git 上下文以使用特定的用户名:

```
git config --global credential.https://github.com.username <your_username>
```

或者，我们可以直接编辑我们的全局 git 配置文件。这通常可以在我们的主目录中的一个名为`.gitconfig`的文件中找到，我们将添加以下几行:

```
[credential "https://github.com"]
	username = <username>
```

**这种方法也不安全，因为用户名是以纯文本的形式存储的**。它也不允许存储密码，所以 git 会继续提示输入密码。

## 4.凭据助手

Git 提供了凭证助手来更安全地保存凭证。凭证助手可以以多种方式存储数据，甚至可以与第三方系统集成，如密码钥匙链。

**git 提供了 2 个现成的基本凭证助手**:

*   缓存:在内存中短期存储的凭据
*   存储:凭据无限期存储在磁盘上

接下来我们将逐一介绍。

### 4.1.缓存凭据帮助程序

缓存凭据帮助程序可以配置如下:

```
git config credential.helper cache
```

缓存凭据帮助程序从不将凭据写入磁盘，尽管可以使用 Unix 套接字访问凭据。这些套接字使用文件权限进行保护，这些文件权限仅限于存储它们的用户，所以一般来说，它们是安全的。

在配置缓存凭证助手时，我们还可以提供一个`timeout`参数。这使我们能够控制凭据在内存中保留的时间:

```
git config credential.helper 'cache --timeout=86400'
```

这将在输入凭据后在内存中保存 1 天。

### 4.2.存储凭据帮助程序

**存储凭证助手无限期地将凭证保存到文件**。我们可以按如下方式配置存储凭据帮助器:

```
git config credential.helper store
```

虽然文件内容没有加密，**它们受到创建文件**的用户的文件系统访问控制的保护。

默认情况下，该文件存储在用户的主目录中。我们可以通过向命令传递一个`file`参数来覆盖文件位置:

```
git config credential.helper 'store --file=/full/path/to/.git_credentials'
```

### 4.3.自定义凭据帮助程序

除了上面提到的两个默认凭证助手，还可以配置[自定义助手](https://web.archive.org/web/20221026020020/https://git-scm.com/docs/gitcredentials#_custom_helpers)。这使我们能够通过委托给第三方应用程序和服务来进行更复杂的凭据管理。

**创建自定义凭证助手不是大多数用户需要担心的事情**。然而，有几个原因让它们很有帮助:

*   与 macOS 上的钥匙串等操作系统工具集成
*   整合现有的企业认证方案，如 LDAP 或 Active Directory
*   提供额外的安全机制，如双因素身份验证

## 5.SSH 密钥

大多数现代 git 服务器提供了一种通过 HTTPS 使用 SSH 密钥而不是用户名和密码访问存储库的方法。SSH 密钥比密码更难猜测，一旦泄露就很容易被撤销。

使用 SSH 的主要缺点是它使用非标准端口。一些网络或代理可能会阻塞这些端口，从而无法与远程服务器通信。它们还需要额外的步骤来在服务器和客户机上设置 SSH 密钥，这在大型组织中可能很麻烦。

为 git 存储库启用 ssh 的最简单方法是在克隆它时对协议使用 SSH:

```
git clone [[email protected]](/web/20221026020020/https://www.baeldung.com/cdn-cgi/l/email-protection):group/project.git
```

对于现有的存储库，我们可以使用以下命令更新远程存储库:

```
git remote set-url origin [[email protected]](/web/20221026020020/https://www.baeldung.com/cdn-cgi/l/email-protection):group/project.git
```

对于每个 git 服务器，配置 SSH 密钥的过程略有不同。一般来说，这些步骤是:

*   在您的机器上生成兼容的公钥/私钥组合
*   将公钥上传到您的 git 服务器

大多数 Unix/Linux 用户已经在其主目录中创建和配置了 SSH 密钥对，并上传了现有的公钥。提醒一下，**我们不应该上传或以其他方式分享我们的私钥**。

## 6.结论

在本教程中，我们看到了配置 git 凭证的各种方法。最常见的方法是使用内置的凭据帮助器将凭据本地存储在内存或磁盘上的文件中。一种更复杂、更安全的存储凭证的方法是使用 SSH，尽管这可能更复杂，并且可能不适用于所有网络。