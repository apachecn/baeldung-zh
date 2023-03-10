# Git 的 docker 文件策略

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/ops/dockerfile-git-strategies>

## 1.介绍

。Git 是软件开发的领先版本控制系统。另一方面，Dockerfile 包含自动构建应用程序映像的所有命令。这两款产品对于任何想要采用 DevOps 的人来说都是完美的组合。

在本教程中，我们将学习一些结合这两种技术的解决方案。我们将详细介绍每个解决方案，并讨论其优缺点。

## 2.Git 存储库中的 Dockerfile

始终能够访问 docker 文件中的 Git 存储库的最简单的解决方案是将 docker 文件直接保存在 Git 存储库中:

```java
ProjectFolder/
  .git/
  src/
  pom.xml
  Dockerfile
  ...
```

上面的设置让我们可以访问整个项目的源代码目录。接下来，我们可以用一个 [`ADD`命令](/web/20220727020703/https://www.baeldung.com/ops/docker-copy-add)将它包含在我们的容器中，例如:

```java
ADD . /project/
```

我们当然可以限制复制到构建目录的范围:

```java
ADD /build/ /project/
```

或者像这样的构建输出。jar 文件:

```java
ADD /output/project.jar /project/
```

这个解决方案最大的优点是我们可以测试任何代码变更，而不需要将它们提交给存储库。一切都将存在于同一个本地目录中。

这里要记住的一点是创建一个 [`.dockerignore`](https://web.archive.org/web/20220727020703/https://docs.docker.com/engine/reference/builder/#dockerignore-file) ` file`。它类似于`.gitignore`文件，但是在这种情况下，它从 Docker 上下文中排除了匹配模式的文件和目录。这有助于我们避免不必要地向 Docker 构建过程发送大的或敏感的文件和目录，以及潜在地将它们添加到映像中。

## 3.克隆 Git 存储库

另一个简单的解决方案是在映像构建过程中获取我们的 git 存储库。我们可以通过简单地将 [SSH 密钥](/web/20220727020703/https://www.baeldung.com/linux/generating-ssh-keys-in-linux)添加到本地存储并调用 `git clone` 命令:来实现

```java
ADD ssh-private-key /root/.ssh/id_rsa
RUN git clone [[email protected]](/web/20220727020703/https://www.baeldung.com/cdn-cgi/l/email-protection):eugenp/tutorials.git
```

上面的命令将获取整个存储库，并将其放在我们的容器中的`./tutorials`目录中。

不幸的是，这种解决方案也有一些缺点。

首先，我们将我们的私有 SSH 密钥存储在 Docker 映像中，这可能会带来潜在的安全问题。我们可以通过使用 git 存储库的用户名和密码来应用一个变通方法:

```java
ARG username=$GIT_USERNAME
ARG password=$GIT_PASSWORD
RUN git clone https://username:[[email protected]](/web/20220727020703/https://www.baeldung.com/cdn-cgi/l/email-protection):eugenp/tutorials.git
```

和[将它们作为环境变量](/web/20220727020703/https://www.baeldung.com/ops/docker-container-environment-variables)从我们的机器中传递。这样，我们将把 git 凭证放在我们的映像之外。

其次，这一步将在以后的构建中被缓存，即使我们的库发生了变化。这是因为带有 `a RUN ` 命令的行是不变的，除非你在前面的步骤中破坏了缓存。虽然，我们可以通过在 `docker build` 命令中添加 `–no-cache` 参数来解决。

另一个小缺点是我们必须在容器中安装 git 包。

## 4.体积映射

我们可以使用的第三个解决方案是[体积映射](/web/20220727020703/https://www.baeldung.com/ops/docker-volumes)。它使我们能够将目录从我们的机器挂载到 Docker 容器中。这是 Docker 容器存储数据的首选机制。

我们可以通过在 docker 文件中添加以下行来实现:

```java
VOLUME /build/ /project/
```

这将在容器上创建`/project`目录，并将其挂载到我们机器上的`/build`目录。

当我们的 Git 存储库仅用于构建过程时，卷映射将是最佳选择。通过将存储库放在容器之外，我们不会增加它的大小，也不会允许存储库内容超出给定容器的生命周期。

我们需要记住的一点是，卷映射赋予 Docker 容器对挂载目录的写访问权。不恰当地使用该特性可能会导致 git 存储库目录中一些不必要的更改。

## 5.Git 子模块

如果我们将 Docker 文件和源代码保存在不同的存储库中，或者我们的 Docker 构建需要多个源存储库，我们可以考虑使用 [Git 子模块](https://web.archive.org/web/20220727020703/https://git-scm.com/book/en/v2/Git-Tools-Submodules)。

首先，我们必须创建一个新的 Git 存储库，并将 Dockerfile 放在那里。接下来，我们可以通过将子模块添加到。gitmodules 文件:

```java
[submodule "project"]
  path = project
  url = https://github.com/eugenp/tutorials.git
  branch = master
```

现在，我们可以像使用标准目录一样使用子模块。例如，我们可以将其内容复制到我们的容器中:

`ADD /build/ /project/`

请记住，子模块不会自动更新。我们需要运行以下 git 命令来获取最新的更改:

```java
git submodule update --init --recursive
```

## 6.概观

在本教程中，我们学习了一些在 Dockerfile 文件中使用 Git 库的方法。和往常一样，所有的源代码都可以在 GitHub 上找到。