# 使用 Docker 在卷中挂载单个文件

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/ops/docker-mount-single-file-in-volume>

## 1.概观

众所周知，我们可以使用 Docker 将我们的应用程序容器化，使软件交付更加容易。除了仅嵌入我们的应用程序，我们还可以使用各种选项来扩展目标容器，例如，挂载外部文件。

在本教程中，**我们将重点关注使用 Docker** 挂载单个文件，展示不同的方法来实现这一点。此外，我们将讨论过程中的常见错误以及如何修复它们。

## 2.将数据保存在 Docker 中

在我们继续之前，让我们快速回顾一下[Docker 如何管理应用数据](/web/20220909090223/https://www.baeldung.com/ops/docker-volumes#what-is-a-volume)。

我们知道 **Docker 为每个运行的容器**创建了一个隔离的环境。默认情况下，文件存储在内部容器的可写层上。这意味着**当容器不再存在时，容器内所做的任何更改都将丢失**。

为了防止数据丢失，Docker 提供了两种主要的机制来将我们的文件持久化到主机上:[卷](https://web.archive.org/web/20220909090223/https://docs.docker.com/storage/volumes/)和[绑定挂载](https://web.archive.org/web/20220909090223/https://docs.docker.com/storage/bind-mounts/)。

卷是由 Docker 自己创建和管理的，外部进程不应该修改它们。另一方面，绑定安装可由主机系统存储和管理，而不阻止非对接进程编辑数据。

为了实现我们的目标，我们将关注绑定机制。

## 3.使用 Docker CLI 绑定文件

与 Docker 交互的最简单方法是使用专用的 Docker CLI，它执行带有配置标志的各种命令。正如我们所知，要创建一个单独的容器，我们可以对所需的图像使用`docker run`命令:

```
docker run alpine:latest
```

[根据官方参考](https://web.archive.org/web/20220909090223/https://docs.docker.com/engine/reference/commandline/run/),`run`命令支持许多允许预配置容器的附加选项。让我们浏览一下可用选项列表:

```
--mount		        Attach a filesystem mount to the container
--volume , -v		Bind mount a volume
```

**两个[的工作方式相似，允许我们在本地保存数据](https://web.archive.org/web/20220909090223/https://docs.docker.com/storage/bind-mounts/#choose-the--v-or---mount-flag)** 。现在，让我们来看看每一个。

### 3.1.`–mount`选项

**的语法`–mount`选项由多个键值对**组成，使用`<key>=<value>`元组。**按键的顺序并不重要**，用逗号分隔多对。

首先，让我们在工作目录中创建一个虚拟文件:

```
$ echo 'Hi Baeldung! >> file.txt
```

要将单个本地文件挂载到容器，我们可以扩展前面的`run`命令:

```
$ docker run -d -it \
   --mount type=bind,source="$(pwd)"/file.txt,target=/file.txt,readonly \
   alpine:latest
```

我们刚刚创建并启动了一个装载本地文件的新容器。现在让我们来看看配置键。

`type `指定安装机构，可用值:`bind`、`volume,`或`tmpfs`。在我们的例子中，我们应该总是给`bind`设置一个值。

`source`(或者–`src`)是主机上应该挂载的文件或目录的绝对路径。我们也可以使用本地 shell 命令来计算结果。

`target `(或者–` destination, dst`)采用文件或目录在容器中挂载的绝对路径。

最后，还有一个`readonly`选项使绑定挂载只读。该标志是可选的。

最后，让我们验证安装结果:

```
$ docker exec ... cat /file.txt
Hi Baeldung!
```

我们还可以检查容器细节，使用`docker inspect`命令检查所有装载:

```
"Mounts": [
    {
        "Type": "bind",
        "Source": ".../file.txt",
        "Destination": "/file.txt",
        "Mode": "",
        "RW": false,
        "Propagation": "rprivate"
    }
],
```

现在，我们可以看看与路径相关的常见错误。**如果我们提供非绝对路径，Docker CLI 将返回一个错误**，终止命令执行:

```
docker: Error response from daemon: invalid mount config for type "bind": invalid mount path: 'file.txt+' mount path must be absolute.
```

有时，我们提供主机上丢失的文件的绝对源路径。在这种情况下，**容器将开始在目标路径**中挂载一个空目录。此外，如果我们在 Windows 上工作，我们 **[应该负责路径转换](https://web.archive.org/web/20220909090223/https://docs.docker.com/desktop/windows/troubleshoot/#path-conversion-on-windows)** 。

### 3.2.`–volume`选项

正如我们前面提到的，我们可以用相同的功能替换`–mount` 和`–volume` ( `–v) `标志)。我们还必须记住语法是完全不同的。

**`–volume` 语法由三个字段**组成，用冒号分隔。此外，**值的顺序是重要的**。

让我们通过使用`–v`选项来转换前面的例子:

```
$ docker run -d -it
    -v "$(pwd)"/file.txt:/file.txt:ro \
    alpine:latest
```

结果是一样的。我们刚刚将本地文件挂载到容器中。

正如我们所见，`–v`选项给出的三个值类似于与`–mount`标志一起使用的键。

第一个值和`source`键一样，指定主机上的文件或目录的路径。

第二个字段提供了容器内部的路径和`target`键。

最后，我们有一个可选的 `ro` 选项来指定`read-only`属性。

检查完语法后，我们来看看一些常见的错误。和以前一样，**我们应该记住 Windows 路径分隔符**。**选择一个不存在的文件也会导致创建一个空目录**。

但是非绝对路径略有不同。**如果我们为第二个值提供一个** **无效路径，和前面一样，Docker CLI 将返回一个错误**。但是，如果我们提供这样一个路径作为源值，Docker 将创建一个命名卷，这是另一种持久化文件的机制。

总之，`–mount`和`–volume`标志之间最显著的区别是它们的语法。我们可以互换使用这两个词。

## 4.使用 Docker 合成绑定文件

在我们学习了如何使用 Docker CLI 绑定文件之后，现在让我们检查一下使用 docker-compose 文件是否还能得到相同的结果。

众所周知， [docker-compose 是一种通过提供配置文件](/web/20220909090223/https://www.baeldung.com/ops/docker-compose)来创建容器的便捷方式。对于每个服务，我们可以声明一个**[`volumes`部分来配置绑定选项](https://web.archive.org/web/20220909090223/https://docs.docker.com/compose/compose-file/#volumes)** 。可以使用长语法或短语法来指定`volumes`部分，这两种语法分别与`–mount`和`–volumes`标志有很多共同之处。

### 4.1.长语法

**长语法允许我们分别配置每个键**来指定卷挂载。对于同一个示例，让我们准备一个 docker-compose 条目:

```
services:
  alpine:
    image: alpine:latest
    tty: true
    volumes:
      - type: bind
        source: ./file.txt
        target: file.txt
        read_only: true
```

与 Docker CLI 一样，我们的容器现在被预先配置为在其中挂载一个本地文件。**我们使用`type`、`source`、`target,`和可选的`read_only`键来确定配置**，就像我们使用`–mount`标志一样。此外，我们可以使用一个相对路径作为从 docker-compose 文件计算的`source`值。

### 4.2.短语法

**短语法使用由冒号分隔的单个字符串值** **来指定卷挂载**:

```
services:
  alpine:
    image: alpine:latest
    tty: true
    volumes:
      - ./file.txt:/file.txt:ro
```

字符串几乎和`–volume`标志一样。**前两个值分别代表源路径和目标路径。最后一部分指定了额外的标志**，在这里我们可以指定只读属性。与长语法一样，我们也可以使用相对源路径。

我们必须记住，长格式允许我们配置不能用短语法表达的附加字段。此外，我们可以在单个`volumes`部分中混合使用这两种语法。

## 5.结论

在本文中，我们刚刚讨论了 Docker 中数据持久性的一部分。我们尝试使用 Docker CLI 和 docker-compose 文件在容器中挂载一个本地文件。

**Docker CLI 为`–mount` 和`–volume` 选项提供了一个`run `命令**来绑定单个文件或目录。两种标志的工作方式相似，但语法不同。因此，我们可以互换使用它们。

我们也可以使用 docker-compose 文件达到相同的结果。**在每个服务的`volumes`部分中，我们可以使用长语法或短语法**来配置卷挂载。和以前一样，这些语法互换产生相同的结果。