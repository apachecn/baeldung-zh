# 从容器名称获取码头容器 ID

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/ops/docker-container-id-from-name>

## 1.概观

Docker 是一种被广泛采用的集装箱化技术。各种应用程序可以在容器中运行。

虽然我们可以在启动容器时控制它的名称，但是 ID 是由 Docker 生成的。我们可能需要这个 ID 来在 Docker 主机上执行某些操作，因此从名称中找到容器的 ID 是一个非常常见的需求。

在这个简短的教程中，我们将讨论从名称中找到容器 ID 的各种方法。

## 2.树立榜样

让我们创建几个容器作为示例:

```
$ docker container run --rm --name web-server-1 -d nginx:alpine
$ docker container run --rm --name web-server-10 -d nginx:alpine
$ docker container run --rm --name web-server-11 -d nginx:alpine
```

现在，让我们检查这些容器是否已经创建:

```
$ docker container ls -a
CONTAINER ID   IMAGE          COMMAND                  CREATED          STATUS          PORTS     NAMES
80f1bc1e7feb   nginx:alpine   "/docker-entrypoint.…"   36 seconds ago   Up 36 seconds   80/tcp    web-server-11
acdea168264a   nginx:alpine   "/docker-entrypoint.…"   36 seconds ago   Up 36 seconds   80/tcp    web-server-10
0cbfc6c17009   nginx:alpine   "/docker-entrypoint.…"   37 seconds ago   Up 36 seconds   80/tcp    web-server-1
```

如我们所见，我们有三个容器处于使用`nginx`图像的`running`状态。

## 3.显示短容器 ID

Docker 给每个集装箱分配一个唯一的 ID。完整的容器 ID 是 64 个字符的十六进制字符串。然而，在大多数情况下，这个容器 ID 的简短版本就足够了。短容器 ID 代表完整容器 ID 的前 12 个字符。

让我们使用 Docker 的 [`container ls`](https://web.archive.org/web/20221005204656/https://docs.docker.com/engine/reference/commandline/container_ls/) 子命令来显示短容器 ID:

```
$ docker container ls --all --quiet --filter "name=web-server-10"
acdea168264a
```

在这个例子中，我们使用了`–filter`选项，它根据条件过滤输出。在我们的例子中，过滤是根据容器的名称进行的。

此外，我们还在命令中使用了`–all`和`–quiet`选项。需要使用`–all`选项来显示所有容器，因为默认情况下，它只显示正在运行的容器。`–quiet`选项仅用于显示集装箱 ID。

我们也可以使用 [`grep`](/web/20221005204656/https://www.baeldung.com/linux/grep-sed-awk-differences#grep) 和 [`awk`](/web/20221005204656/https://www.baeldung.com/linux/awk-guide) 命令的组合来显示短集装箱 ID:

```
$ docker container ls --all | grep web-server-10 | awk '{print $1}'
acdea168264a
```

这里，`awk`命令打印输出的第一列，它表示短容器 ID。

我们应该注意的是， **`grep`和`awk`命令并不是在所有平台上都可用。因此这种方法的可移植性较差**。

## 4.显示完整的集装箱 ID

在大多数情况下，短的容器 ID 就足够了。然而，在极少数情况下，需要完整的容器 ID 以避免歧义。

我们可以使用 Docker 的`container ls`子命令来显示完整的容器 ID:

```
$ docker container ls --all --quiet --no-trunc --filter "name=web-server-10"
acdea168264a08f9aaca0dfc82ff3551418dfd22d02b713142a6843caa2f61bf
```

这里，我们在命令中使用了`–no-trunc`选项。此选项覆盖默认行为并禁用输出截断。

我们可以通过组合使用`grep`和`awk`命令获得相同的结果:

```
$ docker container ls --all --no-trunc | grep web-server-10 | awk '{print $1}'
acdea168264a08f9aaca0dfc82ff3551418dfd22d02b713142a6843caa2f61bf
```

Docker 的 [`container inspect`](https://web.archive.org/web/20221005204656/https://docs.docker.com/engine/reference/commandline/container_inspect/) 子命令以 JSON 格式显示容器的详细信息。我们可以用它来显示容器 ID:

```
$ docker container inspect web-server-10 --format={{.Id}}
acdea168264a08f9aaca0dfc82ff3551418dfd22d02b713142a6843caa2f61bf
```

在这个例子中，我们使用了`–format`选项，它使用 Go 模板从 JSON 输出中提取`Id`字段。

## 5.使用精确匹配显示容器 ID

我们不能在所有场景中使用基本的`grep`或`container ls`子命令。例如，如果容器名部分匹配，这种简单的方法就不起作用。让我们看一个例子。

让我们显示`web-server-1`容器的 ID:

```
$ docker container ls --all --quiet --filter "name=web-server-1"
80f1bc1e7feb
acdea168264a
0cbfc6c17009
```

这里，输出显示了三个容器 id。这是因为名称`web-server-1`与另外两个容器`web-server-10`和`web-server-11`部分匹配。为了避免这种情况，我们可以使用正则表达式。

现在，让我们使用带有容器名的正则表达式:

```
$ docker container ls --all --quiet --filter "name=^web-server-1$"
0cbfc6c17009
```

在这个例子中，我们使用了`caret(^)`和`dollar($)`符号来强制容器名称的精确匹配。

以类似的方式，我们可以使用带有`grep`命令的`-w`选项来强制执行精确匹配:

```
$ docker container ls --all | grep -w web-server-1 | awk '{print $1}'
0cbfc6c17009
```

## 6.结论

在本文中，我们看到了如何使用名称来查找容器 ID。

首先，我们使用了`container` `ls`子命令以及`grep`和 awk `commands`的组合来显示简短的容器 ID。

然后我们使用`–no-trunc`选项和 `container inspect`子命令来显示完整的容器 ID。

最后，我们使用正则表达式来确保容器名的精确匹配。