# 列出 Docker 卷

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/ops/docker-volume-listing>

## 1.概观

Docker 是最流行的容器技术之一。它将应用程序与其依赖项打包在一起，并允许它在隔离的环境中运行。这项技术极大地提高了应用程序的可移植性。默认情况下，容器存储是短暂的，这对于有状态应用程序来说并不理想。但是，我们可以用[卷](/web/20220815215115/https://www.baeldung.com/ops/docker-volumes)来克服这个限制。

在本教程中，我们将了解如何列出卷并显示它们的详细信息。

## 2.树立榜样

让我们创建几个带有属性的卷作为示例:

```
$ docker volume create dangling-volume
$ docker volume create narendra-volume --driver local --opt type=tmpfs --opt device=tmpfs
$ docker volume create labeled-volume --label owner=narendra
```

本教程的后一部分将展示我们如何使用这些属性来过滤体积。在下一节中，我们将看到如何验证我们的示例已经被正确设置。

## 3.显示基本卷信息

**Docker 的`volume` `list`子命令会显示每卷**的简要摘要:

```
$ docker volume list
DRIVER    VOLUME NAME
local     dangling-volume
local     labeled-volume
local     narendra-volume
```

有时，我们只需要卷名。在这种情况下，我们可以使用`–quiet`选项:

```
$ docker volume list --quiet
dangling-volume
labeled-volume
narendra-volume
```

上面的方法对于少量的卷很有效，但是当有很多卷要列出时就很繁琐了。在这种情况下，我们可以使用`–filter`选项。此选项允许我们基于某些属性执行过滤。我们来看几个例子。

### 3.1.**根据**名称过滤

**我们可以使用`name` 过滤器列出其名称包含特定字符串的卷。**让我们列出名字中含有“纳伦德拉”的那一卷:

```
$ docker volume list --filter name=narendra
DRIVER    VOLUME NAME
local     narendra-volume
```

### 3.2.**在**标签上过滤

标签用于标记资源。一个非常常见的场景是对符合特定标准的资源进行分组。例如，开发人员可以使用他们的用户名作为标签，这样他们就可以很容易地识别他们自己创建的卷。让我们用一个例子来理解这一点:

```
$ docker volume list --filter label=owner=narendra
DRIVER    VOLUME NAME
local     labeled-volume
```

在本例中，过滤器匹配带有标签`owner=narendra`的卷，这是我们在设置示例卷时添加的。

### 3.3.**过滤** **上**司机的名字

有时，我们需要根据卷的驱动程序名称来隔离卷。在这种情况下，我们可以使用`driver`过滤器:

```
$ docker volume list --filter driver=local
DRIVER    VOLUME NAME
local     dangling-volume
local     labeled-volume
local     narendra-volume
```

### 3.4.悬垂体积

卷会占用 docker 主机上的空间，因此清理任何未使用的卷是一个好习惯。因为移除错误的卷会导致数据丢失，所以我们在删除它们之前必须格外小心。作为安全检查，**我们可以通过使用`dangling`过滤器**来确保卷不被任何容器引用。让我们来看看实际情况。

首先，让我们创建一个使用卷的容器:

```
$ docker container run -d --name dangling-volume-demo -v narendra-volume:/tmpwork \
   -v labeled-volume:/data busybox
fa3f6fd8261293a92da7efbca4b04040a1838cf57b2703795324eb70a3d84143 
```

在这个例子中，容器使用了三个卷中的两个:`narendra-volume`和`labeled-volume`。现在让我们确认第三个卷是唯一显示为悬空/未使用的卷:

```
$ docker volume list --filter dangling=true
DRIVER    VOLUME NAME
local     dangling-volume
```

## 4.显示详细的卷信息

`list`子命令显示非常有限的卷信息。有时候这还不够。例如，如果我们知道卷的详细信息，调试就会容易得多。在这种情况下，**我们可以使用`volume` `inspect`子命令来获得关于卷**的附加信息。该命令显示卷的创建时间戳、挂载点等信息。让我们看一个例子:

```
$ docker volume inspect labeled-volume
[
    {
        "CreatedAt": "2022-05-30T22:34:53+05:30",
        "Driver": "local",
        "Labels": {
            "owner": "narendra"
        },
        "Mountpoint": "/var/lib/docker/volumes/labeled-volume/_data",
        "Name": "labeled-volume",
        "Options": {},
        "Scope": "local"
    }
]
```

## 5.显示特定于容器的体积信息

另一个非常常见的场景是找到给定容器使用的卷。开发人员经常需要这些信息来调试应用程序。**我们可以使用`container` `inspect`子命令获取特定容器体积的信息。**这个命令返回 Docker 对象的底层信息，比如它们的状态、主机配置、网络设置等。我们可以指定`Mounts`部分来收集卷的挂载信息:

```
$ docker container inspect --format '{{ json .Mounts }}' dangling-volume-demo | python3 -m json.tool
[
    {
        "Type": "volume",
        "Name": "narendra-volume",
        "Source": "/var/lib/docker/volumes/narendra-volume/_data",
        "Destination": "/tmpwork",
        "Driver": "local",
        "Mode": "z",
        "RW": true,
        "Propagation": ""
    },
    {
        "Type": "volume",
        "Name": "labeled-volume",
        "Source": "/var/lib/docker/volumes/labeled-volume/_data",
        "Destination": "/data",
        "Driver": "local",
        "Mode": "z",
        "RW": true,
        "Propagation": ""
    }
]
```

注意，在这个例子中，我们将输出通过管道传输到了 [Python](https://web.archive.org/web/20220815215115/https://www.python.org/) 解释器，以便让 JSON 输出更容易阅读。然而，这是完全可选的。

## 6.结论

**在本文中，我们看到了一些列出 docker 卷的实例。**首先，我们使用了`list`子命令。然后我们看到了如何用它来使用`filters`。最后，我们使用`inspect`子命令来显示卷的详细信息。