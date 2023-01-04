# 更新 Dockerfile 中的 PATH 环境变量

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/ops/dockerfile-path-environment-variable>

## 1.概观

在本文中，我们将看到如何更新 Docker 中的`PATH`变量。首先，我们将在全球范围内更新它。然后，我们将限制对指令子集的更改。

## 2.更新全局`PATH`变量

**`ENV`语句可用于更新`PATH` 变量。**让我们写一个例子`[Dockerfile](/web/20220926111009/https://www.baeldung.com/ops/docker-compose#2-building-an-image)`来展示这种行为:

```
FROM ubuntu:latest
RUN echo $PATH
ENV PATH="$PATH:/etc/profile"
RUN echo $PATH
```

第一行声明我们使用最新的 Ubuntu 映像。我们还记录了`ENV`指令前后的`PATH`变量的值。

让我们建立自己的形象:

```
$ docker build -t baeldungimage .
#4 [1/3] FROM docker.io/library/ubuntu:latest
#5 [2/3] RUN echo /usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
#5 0.683 /usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
#6 [3/3] RUN echo /usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/etc/profile
#6 0.893 /usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/etc/profile
```

不出所料，`/etc/profile`已经被追加到了`PATH`之后。

## 3.仅针对一系列指令更新`PATH`

**我们将使用一条`RUN`指令运行一个 sh 脚本来[导出一个新的`PATH`](/web/20220926111009/https://www.baeldung.com/linux/path-variable#adding-to-path) 。之后，我们将在同一个`RUN`语句中添加另一条指令。这将打印出`RUN`语句中`PATH`的局部值。之后，我们还将记录`PATH`全局变量，以确认它没有改变。**

这是我们的新`Dockerfile`:

```
FROM ubuntu:latest
RUN echo $PATH
RUN export PATH="$PATH:/etc/profile"; echo $PATH
RUN echo $PATH
```

我们现在可以构建图像:

```
$ docker build -t baeldungimage . 
#7 [1/4] FROM docker.io/library/ubuntu:latest
#4 [2/4] RUN echo /usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
#4 0.477 /usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
#5 [3/4] RUN export PATH="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/etc/profile"; echo /usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
#5 0.660 /usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/etc/profile
#6 [4/4] RUN echo /usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
#6 0.661 /usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
```

日志确认在导出`PATH`变量后，它的值被同一个`RUN`命令中的其他指令使用。然而，全局变量没有改变。

## 4.更新外壳会话内部的`PATH`

现在让我们看看如何只为 shell 会话更新`PATH`。首先，我们将修改`.bashrc`文件，以便在每个 shell 会话开始时更新`PATH`。然后，我们将启动一个 shell 会话来检查这种行为。

### 4.1.编辑`.bashrc` 文件

**我们将编辑 [`.bashrc`文件](/web/20220926111009/https://www.baeldung.com/linux/bashrc-vs-bash-profile-vs-profile#2-significance-of-bashrc)，以便在每次 shell 会话开始时导出一个新的`PATH`。**为此，我们将运行[一个快速脚本](/web/20220926111009/https://www.baeldung.com/linux/bash-variables-export#1-export-while-running-bash-scripts)来将导出附加到原始文件。正如我们之前所做的，我们将检查这个变化是否影响全局`PATH`变量。

下面是新的`Dockerfile`:

```
FROM ubuntu:latest
RUN echo $PATH
RUN echo "export PATH=$PATH:/etc/profile" >> ~/.bashrc
RUN cat ~/.bashrc
RUN echo $PATH
```

此外，让我们注意我们使用了`[cat](/web/20220926111009/https://www.baeldung.com/linux/files-cat-more-less#cat)`命令来查看文件。

让我们构建图像:

```
$ docker build -t baeldungimage . 
#4 [1/5] FROM docker.io/library/ubuntu:latest
#5 [2/5] RUN echo /usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
#5 0.447 /usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
#6 [3/5] RUN echo "export PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/etc/profile" >> ~/.bashrc
#7 [4/5] RUN cat ~/.bashrc
#7 0.956 # ~/.bashrc: executed by bash(1) for non-login shells.
#7 0.956 # see /usr/share/doc/bash/examples/startup-files (in the package bash-doc)
#7 0.956 # for examples
#7 0.956
#7 0.956 # If not running interactively, don't do anything
#7 0.956 [ -z "$PS1" ] && return
[... .bashrc full content]
#7 0.956 export PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/etc/profile
#7 DONE 1.0s
#8 [5/5] RUN echo /usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
#8 0.867 /usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
```

正如我们所看到的，全局`PATH`没有改变。然而，导出行确实被添加到了文件的末尾。因此，每当`.bashrc`被加载时，它就为正在运行的 shell 更新`PATH`变量。

### 4.2.以交互模式运行容器

现在让我们关注前面看到的输出中的`.bashrc`文件的前几行。这些行来自原始文件。上面明确写着“`If not running interactively, don't do anything”`”。

理解这一点很重要，当我们构建一个`Dockerfile`时，`RUN`命令不是交互式的。因此，在构建过程中，我们不能只获取我们的`.bashrc`文件和`RUN`脚本来检查路径是否已经更新。

**相反，我们可以[在交互模式](https://web.archive.org/web/20220926111009/https://baeldung-cn.com/ops/docker-container-shell)下运行一个容器，并打开一个 shell 会话:**

```
$ docker run -it --name interactiveimage baeldungimage
[[email protected]](/web/20220926111009/https://www.baeldung.com/cdn-cgi/l/email-protection):/# echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/etc/profile
```

一旦在 shell 会话中，我们已经打印了`PATH.` ,我们可以看到`/etc/profile`被附加，确认我们的`.bashrc`文件已经被考虑。

## 5.结论

在本教程中，我们已经看到了如何在 Docker 中更新`PATH`变量。最初，我们全局更新了变量，但我们也学会了如何用更多的限制来更新它。

和往常一样，代码可以在 GitHub 的[上获得。](https://web.archive.org/web/20220926111009/https://github.com/eugenp/tutorials/tree/master/docker-modules/docker-images)