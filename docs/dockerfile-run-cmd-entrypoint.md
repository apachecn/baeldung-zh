# Dockerfile 文件中 run、cmd 和 entrypoint 之间的区别

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/ops/dockerfile-run-cmd-entrypoint>

## 1.概观

在 docker 文件中，我们经常会遇到像`run`、`cmd,`或者`entrypoint`这样的指令。乍一看，它们都用于指定和运行命令。但是它们之间有什么区别呢？它们是如何相互作用的？

在本教程中，我们将回答这些问题。我们将介绍这些指令的作用以及它们是如何工作的。我们还将看看它们在构建映像和运行 [Docker 容器](/web/20221111103508/https://www.baeldung.com/docker-images-vs-containers)中扮演什么角色。

## 2.设置

首先，让我们创建一个脚本，`log-event.sh.`它只是在文件中添加一行，然后打印出来:

```java
#!/bin/sh

echo `date` [[email protected]](/web/20221111103508/https://www.baeldung.com/cdn-cgi/l/email-protection) >> log.txt;
cat log.txt;
```

现在，让我们创建一个简单的 Dockerfile 文件:

```java
FROM alpine
ADD log-event.sh /
```

它将通过在不同的场景中向`log.txt`追加行来利用我们的脚本。

## 3.`run`命令

当我们构建映像时，`run`指令执行。这意味着传递给`run`的命令在新图层中的当前图像之上执行。然后将结果提交给图像。让我们看看这是如何工作的。

首先，我们将在 docker 文件中添加一条`run`指令:

```java
FROM alpine
ADD log-event.sh /
RUN ["/log-event.sh", "image created"]
```

其次，让我们用以下方式建立我们的形象:

```java
docker build -t myimage .
```

现在我们希望有一个 Docker 图像，其中包含一个带有一行`image created`的`log.txt`文件。让我们通过运行一个基于图像的容器来检查这一点:

```java
docker run myimage cat log.txt
```

当列出文件的内容时，我们将看到如下输出:

```java
Fri Sep 18 20:31:12 UTC 2020 image created
```

如果我们多次运行该容器，我们将会看到日志文件中的日期没有改变。这是有意义的，因为 **`run`步骤在映像构建时**执行，而不是在容器运行时执行。

现在让我们再次建立我们的形象。我们注意到日志中的创建时间没有改变。这是因为 **[Docker 缓存了运行指令](/web/20221111103508/https://www.baeldung.com/linux/docker-build-cache)**的结果，如果 Docker 文件没有改变的话。如果我们想使缓存无效，我们需要将`–no-cache`选项传递给 build 命令。

## 4.`cmd`命令

使用 **`cmd`指令，我们可以指定容器启动时执行的默认命令。**让我们在 docker 文件中添加一个`cmd`条目，看看它是如何工作的:

```java
...
RUN ["/log-event.sh", "image created"]
CMD ["/log-event.sh", "container started"]
```

构建完映像后，现在让我们运行它并检查输出:

```java
$ docker run myimage
Fri Sep 18 18:27:49 UTC 2020 image created
Fri Sep 18 18:34:06 UTC 2020 container started
```

如果我们运行多次，我们会看到`image created`条目保持不变。但是`container started`条目会随着每次运行而更新。这显示了每次容器启动时`cmd`是如何执行的。

注意，我们这次使用了一个稍微不同的`docker run`命令来启动我们的容器。让我们看看，如果运行与之前相同的命令，会发生什么情况:

```java
$ docker run myimage cat log.txt
Fri Sep 18 18:27:49 UTC 2020 image created
```

这一次，Dockerfile 文件中指定的`cmd`被忽略。这是因为我们已经为`docker run`命令指定了参数。

现在让我们继续，看看如果 docker 文件中有不止一个`cmd`条目会发生什么。让我们添加一个新条目来显示另一条消息:

```java
...
RUN ["/log-event.sh", "image created"]
CMD ["/log-event.sh", "container started"]
CMD ["/log-event.sh", "container running"]
```

构建映像并再次运行容器后，我们会发现以下输出:

```java
$ docker run myimage
Fri Sep 18 18:49:44 UTC 2020 image created
Fri Sep 18 18:49:58 UTC 2020 container running
```

正如我们所见，`container started`条目不存在，只有`container running` 是`.` ，这是因为如果指定了多个命令，那么**只调用最后一个命令。**

## 5.`entrypoint`命令

正如我们在上面看到的，如果在启动容器时传递任何参数，`cmd`将被忽略。如果我们想要更大的灵活性呢？假设我们想要定制附加的文本，并将其作为参数传递给`docker run`命令。为此，让我们使用`entrypoint.` 来指定容器启动时运行的默认命令。此外，我们现在能够提供额外的参数。

让我们用`entrypoint:`替换 docker 文件中的`cmd`条目

```java
...
RUN ["/log-event.sh", "image created"]
ENTRYPOINT ["/log-event.sh"]
```

现在让我们通过提供一个自定义文本条目来运行容器:

```java
$ docker run myimage container running now
Fri Sep 18 20:57:20 UTC 2020 image created
Fri Sep 18 20:59:51 UTC 2020 container running now
```

我们可以看到 **`entrypoint`的行为与** `**cmd**.` **相似，此外，它允许我们定制启动时执行的命令。**

与`cmd`一样，如果有多个`entrypoint`条目，则只考虑最后一个。

## 6.`cmd`和`entrypoint`之间的相互作用

我们使用了`cmd`和`entrypoint`来定义运行容器时执行的命令。现在让我们继续，看看如何结合使用`cmd`和`entrypoint`。

一个这样的用例是为`entrypoint.` 定义默认参数，让我们在 docker 文件中的`entrypoint`后添加一个`cmd`条目:

```java
...
RUN ["/log-event.sh", "image created"]
ENTRYPOINT ["/log-event.sh"]
CMD ["container started"]
```

现在，让我们在不提供任何参数的情况下运行我们的容器，并使用在`cmd`中指定的默认值:

```java
$ docker run myimage
Fri Sep 18 21:26:12 UTC 2020 image created
Fri Sep 18 21:26:18 UTC 2020 container started
```

如果我们愿意，也可以覆盖它们:

```java
$ docker run myimage custom event
Fri Sep 18 21:26:12 UTC 2020 image created
Fri Sep 18 21:27:25 UTC 2020 custom event
```

需要注意的是,`entrypoint`以 shell 形式使用时的不同行为。让我们更新 docker 文件中的`entrypoint`:

```java
...
RUN ["/log-event.sh", "image created"]
ENTRYPOINT /log-event.sh
CMD ["container started"]
```

在这种情况下，当运行容器时，我们将看到 Docker 如何忽略传递给`docker run` 或`cmd`的任何参数。

## 7.结论

在本文中，我们已经看到了 Docker 指令之间的异同:`run`、`cmd,`和`entrypoint`。我们已经观察到它们在什么时候被调用。此外，我们还了解了它们的用途以及它们是如何协同工作的。