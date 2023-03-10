# 将 Docker 映像推送到私有存储库

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/ops/docker-push-image-to-private-repository>

## 1.概观

本教程演示了如何将 Docker 映像推送到私有存储库。我们将从创建一个样例应用程序开始，它将是我们 Docker 图像的基础。然后我们将看到如何登录我们的私有 Docker 存储库，最后学习如何标记图像并将其推送到存储库。

## 2.私有 Docker 存储库

**私有 Docker 存储库提供对其包含的图像的受限访问**。与公共存储库不同，只有授权用户才能访问图像。这样，就有可能只允许特定的用户组访问，比如组织、团队，甚至是个人。对于不想公开 Docker 图像的项目来说，这是一个完美的选择。

将 Docker 存储库设为私有是通过存储库设置完成的。对于不同的提供者来说，关于如何做的细节可能会有所不同，但是通常情况下，只需要选中一个复选框。

现在我们知道了私有 Docker 存储库的好处，让我们来研究如何将图像推送到这样的存储库。步骤与将图像推送到公共存储库相同。唯一的区别是存储库被标记为私有。

## 3.准备图像

首先，我们需要通过可选地提供正确的别名和标记来准备图像。这可以在构建映像时完成，也可以通过使用现有映像并对其进行重命名来完成。

### 3.1.建立新形象

首先，我们从一个简单的 Spring Boot 应用程序创建一个 Docker 图像，该应用程序由一个向用户返回友好消息的`RestController `组成:

```java
@RestController
public class HelloWorldController {
    @GetMapping("/helloworld")
    String helloWorld() {
        return "Hello World!";
    }
}
```

我们使用以下 Dockerfile 文件:

```java
FROM openjdk:11
ARG JAR_FILE=target/*.jar
COPY ${JAR_FILE} app.jar
ENTRYPOINT ["java","-jar","/app.jar"]
```

最后，我们运行构建图像的命令。看起来是这样的:

```java
docker build [OPTIONS] PATH | URL | -
```

在我们的例子中，我们使用-t 选项，它告诉我们想要标记图像。我们还提供了点“.”作为 jar 文件的`PATH`。选择由你的用户名和库名组成的正确名称是很重要的**。**版本标签是可选的。如果避免，图像将被标记为`latest`:

```java
docker build -t username/fancy-repository:v1.0.0 .
```

现在，我们可以使用以下命令列出现有的图像:

```java
docker images
```

因此，我们将看到新创建的图像:

```java
REPOSITORY                  TAG       IMAGE ID        CREATED              SIZE
username/fancy-repository   v1.0.0    e20b5a89a0f2   About a minute ago   665MB
```

### 3.2.准备现有图像

在某些情况下，我们不想从头创建一个映像，而是推送一个现有的映像。这需要一些准备步骤，我们将在本节中探讨。假设我们的机器上有以下图像:

```java
REPOSITORY       TAG         IMAGE ID       CREATED        SIZE
existing-image   some-tag    e20b5a89a0f2   2 weeks ago   665MB
```

为了将它推送到我们的`fancy-repository`，我们首先需要使用以下命令用正确的名称/别名标记图像:

```java
docker tag SOURCE_IMAGE[:TAG] TARGET_IMAGE[:TAG]
```

在我们的例子中，`SOURCE_IMAGE[:TAG]`是现有图像的名称和标签。`TARGET_IMAGE[:TAG]` **是由我们的用户名和我们想要将图像推送到**的存储库的名称组成的别名。在我们的示例中，该命令如下所示:

```java
docker tag existing-image:some-tag username/fancy-repository:v1.0.0
```

我们可以使用以下命令检查结果:

```java
docker images
```

现在我们可以看到一个额外的条目，它显示了存储库的名称和新应用的版本标签。图像 id、时间戳和大小是相同的，因为它仍然是相同的图像，只是具有另一个别名:

```java
REPOSITORY                      TAG         IMAGE ID        CREATED               SIZE
existing-image                  some-tag    e20b5a89a0f2    2 weeks ago          665MB
username/fancy-repository       v1.0.0      e20b5a89a0f2    2 weeks ago          665MB
```

这样，我们可以继续将图像推送到我们的私有存储库。

## 4.推送图像

既然我们已经准备好了 Docker 映像，我们可以将它推送到我们的私有存储库中。第一步是使用以下命令登录 DockerHub 注册表:

```java
docker login
```

最后一步是使用以下命令推送映像:

```java
docker push [OPTIONS] NAME[:TAG]
```

在我们的例子中，我们不需要指定任何选项，只需要提供图像名称和标签。该命令将如下所示:

```java
docker push username/fancy-repository:v1.0.0
```

这样，图像将被上传到 DockerHub 上我们的私有 Docker 库。我们可以通过运行以下命令来验证映像是否在 DockerHub 上:

```java
docker search username/fancy-repository
```

因此，我们将获得以下输出，显示我们的图像细节，并证明它实际上在 DockerHub 上可用:

```java
NAME                        DESCRIPTION   STARS     OFFICIAL   AUTOMATED
username/fancy-repository                 0 
```

## 5.结论

在本文中，我们探讨了如何将 Docker 映像推送到私有存储库。我们已经了解了什么是私有存储库以及它的用途。然后，我们展示了如何准备一个图像并将其推送到私有存储库。

本文中提到的所有代码示例都可以在 GitHub 上的[处获得。](https://web.archive.org/web/20220524070336/https://github.com/eugenp/tutorials/tree/master/docker/docker-push-to-private-repo)