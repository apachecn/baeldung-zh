# 如何将环境变量值传入 Dockerfile

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/ops/dockerfile-env-variable>

## 1.概观

环境变量是外部化应用程序配置的一种便捷方式。因此，它们也可用于建造码头集装箱。然而，在`Dockerfile`中传递和使用它们并不像想象中那么容易。

在这个简短的教程中，我们将仔细看看如何将环境变量值传递到 Dockerfile 中。

首先，我们将展示将环境变量传递给构建过程可能有用的可能用例 `. ` 然后，我们将解释做这件事的`ARG`命令。最后，我们将看到一个工作示例。

## 2.`Dockerfile`中的环境变量与容器

`Dockerfile`是一个**脚本，包含如何构建 Docker 映像**的指令。另一方面，**Docker 容器是一个图像**的可运行实例。根据我们的需要，我们可能需要构建时或运行时环境变量[。](/web/20220816103559/https://www.baeldung.com/ops/docker-container-environment-variables)

我们只关注构建时定制，将环境变量传递到我们的`Dockerfile`中，供`docker build`使用。

## 3.在`Dockerfile`中使用环境变量的优势

使用环境变量的最大优点是灵活性。我们可以只创建一个`Dockerfile`，根据构建容器的环境，对**进行不同的配置。举例来说，让我们想象一个在开发环境中启用了调试选项而在生产环境中禁用了相同选项的应用程序。对于环境变量，我们只需要创建一个`Dockerfile`，它将把包含调试标志的环境变量传递给容器和其中的应用程序。**

另一个重要的优势是安全问题。将密码或其他敏感信息直接存储在`Dockerfile`中可能不是最好的主意。环境变量有助于克服这个问题。

## 4.示例配置

在我们看到如何将环境变量值传递到一个`Dockerfile`之前，让我们构建一个例子来测试它。

我们将创建一个名为 `greetings.sh`的简单 bash 脚本，它使用一个环境变量将问候打印到控制台:

```java
#!/bin/sh

echo Hello $env_name
```

现在，让我们在同一个目录中创建一个`Dockerfile`:

```java
FROM alpine:latest

COPY greetings.sh .

RUN chmod +x /greetings.sh

CMD ["/greetings.sh"]
```

它复制我们的脚本，使其可执行，并运行它。让我们构建图像:

```java
docker build -t baeldung_greetings . 
```

那么，让我们运行它:

```java
docker run baeldung_greetings
```

我们应该在控制台中只看到一行:

```java
Hello
```

## 5.将环境变量传递到一个`Dockerfile`

`Dockerfile`提供了一个专用的变量类型`ENV`来创建一个环境变量。**我们可以在构建期间访问 ENV 值，也可以在容器运行后访问 ENV 值**。

让我们看看如何使用它将值传递给我们的问候脚本。有两种不同的方法做这件事。

### 5.1.硬编码环境值

传递环境值最简单的方法是在`Dockerfile`中硬编码。在某些情况下，这已经足够好了。让我们将`John `硬编码为 docker 文件中的默认名称:

```java
FROM alpine:latest

ENV env_name John

COPY greetings.sh .

RUN chmod +x /greetings.sh

CMD ["/greetings.sh"]
```

现在，让我们构建并运行我们的映像。以下是所需的控制台输出:

`Hello John`

### 5.2.设置动态环境值

`Dockerfile`不提供动态工具来在构建过程中设置 ENV 值。但是，这个问题是有解决办法的。我们得用`ARG`。 **ARG 值的工作方式与 ENV 不同，因为一旦映像被构建**，w **e 就不能再访问它们了。让我们看看如何解决这个问题:**

```java
ARG name
ENV env_name $name
```

我们现在介绍的是 `name` `ARG`变量。之后，我们用它来给使用`ENV`的 `env_name` 环境变量赋值。

当我们要设置这个参数时，我们用 `–build-arg` 标志传递它:

```java
docker build -t baeldung_greetings --build-arg name=Baeldung .
```

现在，让我们运行我们的容器。我们应该看到:

```java
Hello Baeldung
```

如果我们想改名字呢？我们所要做的就是用不同的`build-arg`值重建图像。

## 6.结论

在本文中，我们学习了如何在构建`Dockerfile`的过程中设置环境变量。

首先，我们看到了参数化我们的`Dockerfile`的优势。然后，我们看到了如何使用`ENV`命令来设置环境变量，以及如何使用`ARG`来允许在构建时修改这个值。

和往常一样，GitHub 上的[提供了这篇文章的完整源代码。](https://web.archive.org/web/20220816103559/https://github.com/eugenp/tutorials/tree/master/docker-modules/docker-environment-variables)