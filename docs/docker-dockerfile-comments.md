# 在 Dockerfile 文件中添加注释

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/ops/docker-dockerfile-comments>

## 1.概观

在本教程中，我们将学习如何在[docker 文件](/web/20221110103941/https://www.baeldung.com/ops/docker-compose#2-building-an-image)中添加注释。我们还将强调看起来像注释但不是注释的指令之间的区别。

## 2.向 Dockerfile 文件添加注释

我们将使用以下 Dockerfile 文件:

```java
FROM ubuntu:latest
RUN echo 'This is a Baeldung tutorial'
```

让我们快速了解一下:

*   第一行声明我们使用最新的 ubuntu 映像
*   第二行将 [`echo`命令](/web/20221110103941/https://www.baeldung.com/linux/echo-command)作为 shell 的参数传递

让我们建立我们的[形象](https://web.archive.org/web/20221110103941/https://baeldung-cn.com/ops/docker-images-vs-containers):

```java
$ docker build -t baeldungimage .
#4 [1/2] FROM docker.io/library/ubuntu:latest
#5 [2/2] RUN echo 'This is a Baeldung tutorial'
```

Docker 打印了(在我们没有列出的其他行中)成功运行的两个步骤。现在让我们看看如何向 docker 文件添加注释。

### 2.1.创建单行注释

要注释一行，它必须以#开头。

让我们看看如何修改 docker 文件来添加一些单行注释:

```java
# Declare parent image
FROM ubuntu:latest
# Print sentence
RUN echo 'This is a Baeldung tutorial'
```

让我们构建修改后的图像:

```java
$ docker build -t baeldungimage .
#4 [1/2] FROM docker.io/library/ubuntu:latest
#5 [2/2] RUN echo 'This is a Baeldung tutorial'
```

正如预期的那样，Docker 成功地运行了与之前相同的两个步骤。

### 2.2.多行注释

Docker 中没有专门的语法来编写多行注释。**因此，编写多行注释的唯一方法是在一行中编写多个单行注释**:

```java
# This file is a demonstration
# For a Baeldung article
FROM ubuntu:latest
RUN echo 'This is a Baeldung tutorial'
```

构建映像仍然打印与之前相同的步骤:

```java
$ docker build -t baeldungimage .
#4 [1/2] FROM docker.io/library/ubuntu:latest
#5 [2/2] RUN echo 'This is a Baeldung tutorial'
```

## 3.避免陷阱

在这一节中，我们将看看我们应该注意的几个陷阱。这些代码行看起来像注释，但实际上并不像。

### 3.1.注释或命令参数？

在 Docker 中，不可能在行尾添加注释。让我们看看，如果我们尝试在一条指令的末尾添加一个句子，格式类似单行注释，会发生什么情况:

```java
FROM ubuntu:latest
RUN echo 'This is a Baeldung tutorial' # Print sentence
```

我们现在将构建图像:

```java
$ docker build -t baeldungimage .
#4 [1/2] FROM docker.io/library/ubuntu:latest
#5 [2/2] RUN echo 'This is a Baeldung tutorial' # Print sentence
#5 0.371 This is a Baeldung tutorial
```

我们包含了由`echo`命令输出的句子，以强调结果确实与我们之前得到的一样。这里发生了什么？我们真的在 docker 文件的第二行末尾添加了注释吗？

实际上，`# Print sentence`是作为附加参数传递给`RUN`命令的。在这种情况下，这个命令碰巧忽略了额外的参数。为了说服我们自己，现在让我们在 docker 文件的第一行末尾添加一个类似的句子:

```java
FROM ubuntu:latest # Declare parent image
RUN echo 'This is a Baeldung tutorial'
```

让我们试着建立这样的形象:

```java
$ docker build -t baeldungimage .
failed to solve with frontend dockerfile.v0: failed to create LLB definition: dockerfile parse error line 1: FROM requires either one or three arguments
```

在这里，我们得到了一个非常明确的错误消息，证明了我们的肯定。

### 3.2.解析器指令不是注释

解析器指令告诉 Dockerfile 解析器如何处理文件的内容。与注释类似，解析器指令以#开头。

此外，我们应该注意解析器指令必须在 docker 文件的顶部。例如，我们将在文件中使用`escape`解析器指令。该指令更改文件中使用的转义符:

```java
# escape=`
FROM ubuntu:latest
RUN echo 'This is a Baeldung tutorial&' `
  && echo 'Print more stuff'
```

这里，我们在`RUN`命令中添加了另一个`echo`指令。为了提高可读性，我们将这条指令放在了新的一行。默认的行分隔符是`\`。然而，由于我们使用了解析器指令，我们需要使用`` ` ``来代替。现在让我们建立我们的形象，看看会发生什么:

```java
$ docker build -t baeldungimage .
#4 [1/2] FROM docker.io/library/ubuntu:latest
#5 [2/2] RUN echo 'This is a Baeldung tutorial&' && echo 'Print more stuff'
#5 0.334 This is a Baeldung tutorial&
#5 0.334 Print more stuff 
```

这两句话已经按预期打印出来了。

## 4.结论

在本文中，我们看到了如何在 Dockerfile 文件中编写注释。我们还了解到有一组称为解析器指令的指令，它们看起来非常类似于注释，但是对我们的文件有真正的影响。

和往常一样，代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221110103941/https://github.com/eugenp/tutorials/tree/master/docker-modules/docker-images)