# 从命令行重置/禁用 Jenkins 安全设置

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/ops/jenkins-security-settings>

## 1.概观

[Jenkins](https://web.archive.org/web/20221208143832/https://www.jenkins.io/) 是一款开源自动化服务器，用于自动化部分和全部软件开发周期。便于[持续集成和持续交付](/web/20221208143832/https://www.baeldung.com/spring-boot-ci-cd)。

有了 Jenkins，我们能够为不同的用户提供不同级别的安全性。默认情况下，我们不需要向所有用户提供完全访问权限。
可以从 Jenkins 控制台(UI)和命令行查看、重置或完全禁用 Jenkins 安全性。使用命令行，我们需要更改 Jenkins 配置。

在我们进一步讨论之前，让我们先了解这个问题。有时，我们可能会忘记 Jenkins 登录凭证。因此，我们无法接近詹金斯。

在本教程中，我们将讨论使用命令行重新访问 Jenkins 控制台的不同方法。我们将学习重置丢失的密码，禁用安全，以及其他替代方法。

假设我们拥有对 Jenkins 机器的 SSH 访问权限。**我们现在要讨论的每个解决方案都需要重启 Jenkins 服务器。**所以要确保机器上没有正在运行的作业。

## 2.通过更新詹金斯主配置文件

由于我们无法访问 Jenkins 控制台，我们将使用命令行更新 Jenkins 配置。

### 2.1.查找主配置文件

一般来说，我们可以用两种方法在 Linux 机器上安装 Jenkins，使用[包管理器](/web/20221208143832/https://www.baeldung.com/linux/yum-and-apt)或者使用 WAR 文件。如果 Jenkins 是使用软件包管理器服务器安装的，那么`config.xml`文件的路径将是`/var/lib/jenkins/config.xml`。另一方面，如果 [Jenkins 安装是使用 WAR 文件](https://web.archive.org/web/20221208143832/https://www.jenkins.io/doc/book/installing/war-file/)完成的，那么`config.xml`文件将位于`~/.jenkins/config.xml`中

如果`config.xml`文件不在上述路径中，我们可以使用 [`find`](https://web.archive.org/web/20221208143832/https://man7.org/linux/man-pages/man1/find.1.html) 命令在整个机器中搜索:

```java
$ find / -name config.xml
```

### 2.2.禁用詹金斯安全系统

一旦我们找到了`config.xml`文件，让我们将下面的安全属性从`true`更新为`false`:

```java
<useSecurity>false</useSecurity>
```

如果无法访问[编辑器](/web/20221208143832/https://www.baeldung.com/linux/vi-editor)，让我们使用 [`sed`](https://web.archive.org/web/20221208143832/https://www.gnu.org/software/sed/manual/sed.html) 命令更新`config.xml` 文件:

```java
$ sed -i 's/<useSecurity>true<\/useSecurity>/<useSecurity>false<\/useSecurity>/g' /var/lib/jenkins/config.xml
```

### 2.3.重启詹金斯

最后，我们将重新启动 Jenkins 以使更改生效。如果 Jenkins 是使用软件包管理器安装的，请使用以下命令:

```java
$ systemctl restart jenkins
```

如果 Jenkins 是使用 WAR 安装的，首先我们需要停止 Java 进程，然后使用`java -jar`命令重启 Jenkins。

现在，当访问詹金斯控制台时，它不会要求输入密码。这种解决方案很简单，但不推荐使用，因为它完全绕过了安全性。

## 3.通过更新 Jenkins 用户配置文件

现在让我们来看看一个更好的解决方案，我们将在 Jenkins 用户的配置文件中重置密码。让我们确保我们有足够的权限来更新 Jenkins 工作目录中的文件。

### 3.1.查找用户配置文件

在进一步深入之前，让我们深入研究一下 Jenkins 目录结构。Jenkins 创建了一个`users`目录来存储所有用户帐户的详细信息。这个目录将存在于 Jenkins 工作目录中。我们将在以下文件路径中找到对应于每个 Jenkins 用户的`config.xml`文件:

```java
<Jenkins_Working_Directory>/users/<Jenkins_User_Folder>/config.xml
```

这里的`Jenkins_Working_Directory`是一个存储所有日志、配置和构建工件的目录。Jenkins 工作目录的默认路径是`/var/lib/jenkins.` 。 `Jenkins_User_Folder` 是 Jenkins 用户的文件夹名:

```java
$ cd /var/lib/jenkins/users/
$ ls
user1_4268539434599263174  user2_948489902389144094  user3_162302090988132370  users.xml
$ cd user1_4268539434599263174/
$ ls
config.xml
```

### 3.2.生成 BCrypt 哈希

我们刚刚发现的用户配置文件包括许多用户级配置，包括密码散列。Jenkins 使用 [bcrypt](/web/20221208143832/https://www.baeldung.com/spring-security-registration-password-encoding-bcrypt) 哈希算法生成密码的哈希。bcrypt 算法使用 salt round 来决定返回最终哈希之前的迭代次数。因此，它是多层安全的。

简单地说，我们将选择一个新密码，生成它的散列并替换`config.xml`文件中的散列。这样，我们的密码将重置成功。为了加密密码并生成它的散列，我们将使用这个[公开可用的工具](https://web.archive.org/web/20221208143832/https://www.javainuse.com/onlineBcrypt)。

让我们假设一种情况，我们丢失了根用户的密码。我们现在决定更新新密码为`secret.` ，使用在线工具生成的散列值为`$2a$10$a7XcruSVvyb0.6ckv97/hOqqTuVx.qzuf7oq9He6HG7puq8DzYwFq`

**请注意，对于相同的密码，我们每次加密密码时都会生成不同的哈希。**这一行为证明了 bcrypt 算法的强大。

### 3.3.更新配置文件

现在让我们替换用户`config.xml`文件中的`passwordHash`:

```java
<passwordHash>#jbcrypt:$2a$10$a7XcruSVvyb0.6ckv97/hOqqTuVx.qzuf7oq9He6HG7puq8DzYwFq</passwordHash>
```

这里 jBCrypt 表示 BCrypt 的 Java 实现。最后，我们需要重启 Jenkins 服务器以使更改生效。我们的密码现在重置为秘密。

这种方法优于前面的方法，因为它不影响其他 Jenkins 用户的安全。

## 4.使用另一个用户

因为我们已经丢失了 admin 用户的密码，所以让我们创建一个具有 root 权限的新用户。我们将使用这个新用户从 Jenkins 控制台重置旧用户的密码。

最后，我们将清理所有的配置和我们创建的新用户。

### 4.1.启用注册

默认情况下，Jenkins 会在初始安装时提供注册选项。让我们通过将主`config.xml` ( `/var/lib/jenkins/config.xml`或`~/.jenkins/config.xml`)文件中的`disableSignup`属性设置为`false`来启用它:

```java
<disableSignup>false</disableSignup>
```

### 4.2.创建新的根用户

现在让我们重新启动 Jenkins 服务器并访问 Jenkins 控制台。这一次，我们将在 Jenkins 登录页面上找到一个创建新帐户的链接。

让我们首先从 Jenkins 控制台注册一个新用户(`myuser`)。然后，通过更新主`config.xml`文件，将管理员权限赋予该用户:

```java
<roleMap type="globalRoles">
    <role name="admin" pattern=".*">
        <permissions>
            ...
        </permissions>
        <assignedSIDs>
            <sid>myuser</sid>
        </assignedSIDs>
    </role>
<roleMap/>
```

这里，我们在`assignedSIDs`标签内的`sid`标签周围添加了新创建的用户。现在重启 Jenkins 服务器。

### 4.3.更新密码

让我们使用新用户(`myuser`)登录，然后前往`Manage Jenkins > Manage Users`。现在选择我们希望更新密码的用户(root ),并更新密码。

现在让我们保存并应用更改。这将成功更新我们之前丢失的用户密码。

### 4.4.打扫

一旦我们恢复了密码，让我们清理一切。首先，我们将删除新创建的用户`myuser`。为此，请使用我们刚刚恢复密码的管理员用户登录。然后，转到`Manage Jenkins > Manage Users`并删除我们之前创建的用户。

其次，我们需要从`config.xml`文件的`assignedSIDs`标签中删除用户条目。最后，通过将`disableSignup`标志设置回`true.`来禁用注册功能

这个解决方案不会对 Jenkins 的安全造成任何伤害。当使用基于角色的机制管理用户时，使用[基于角色的授权策略插件](https://web.archive.org/web/20221208143832/https://plugins.jenkins.io/role-strategy/)会很有帮助。

## 5.删除配置

如果由于某种原因上述方法都不起作用，我们可以删除配置属性/文件。它将禁用所有詹金斯用户的安全。因此，这不是完成工作的推荐方式。

### 5.1.删除配置属性

我们可以从`config.xml`中删除`useSecurity`和`authorizationStrategy`安全属性，以禁用 Jenkins 中的安全设置:

```java
$ sudo ex +g/useSecurity/d +g/authorizationStrategy/d -scwq /var/lib/jenkins/config.xml
```

让我们重新启动詹金斯服务器。一旦我们能够访问 Jenkins，我们就可以从 Jenkins 控制台上的“配置全局安全性”页面重新启用安全性。

### 5.2.删除配置文件

我们还可以删除 Jenkins `config.xml`文件来禁用安全性:

```java
$ rm -f /var/lib/jenkins/config.xml
```

**请注意，之前所做的所有配置更改都将被丢弃，默认配置文件将被加载。**

同样，我们需要重启 Jenkins 服务器以使更改生效。

## 6.结论

在本文中，我们通过不同的方法在丢失密码后重新获得对 Jenkins 控制台的访问。

首先，我们研究了一种完全破坏安全的方法。这也会影响到其他用户。因此，不建议这样做。

此外，我们通过覆盖 Jenkins `config.xml`文件中的哈希并创建另一个 admin 用户来重置密码。这是一个完美的解决问题的方法，不会妨碍詹金斯的任何其他方面。

最后，我们删除了与 Jenkins 配置安全相关的属性和文件，以禁用安全性。