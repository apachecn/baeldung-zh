# 无需使用 Docker Hub 即可共享 Docker 图像

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/ops/share-image-without-docker-hub>

## 1.概观

假设我们需要共享本地机器上的一个 [Docker](/web/20220529023903/https://www.baeldung.com/ops/docker-guide) 映像。为了解决这个问题，[码头枢纽](https://web.archive.org/web/20220529023903/https://hub.docker.com/)前来救援。

Docker Hub 是一个基于云的中央存储库，可以存储 Docker 图像。因此，我们需要做的就是将我们的 Docker 映像推送到 Docker Hub，稍后，任何人都可以拉同一个 Docker 映像。

作为一个基于云的存储库，Docker Hub 需要额外的网络带宽来上传和下载 Docker 映像。此外，随着图像大小的增加，上传/下载图像所需的时间也会增加。因此，这种共享 Docker 映像的方法并不总是有用的。

在本教程中，我们将讨论一种不使用 Docker Hub 共享 Docker 图片的方法。事实证明，当发送方和接收方连接到同一个专用网络时，这种方法非常方便。

## 2.将 Docker 图像保存为`tar`档案

假设有一个 Docker 映像`baeldung `，我们需要将它从机器 A 传输到机器 b。为了实现这一点，首先，我们将使用 [`docker save`](https://web.archive.org/web/20220529023903/https://docs.docker.com/engine/reference/commandline/save/) 命令将 Docker 映像转换成一个`.tar`文件:

```java
$ docker save --output baeldung.tar baeldung
```

上面的命令将创建一个名为`baeldung.tar. `的 tar 归档文件，或者，我们也可以使用文件重定向来实现类似的结果:

```java
$ docker save baeldung > baeldung.tar
```

**`docker save`命令可以使用多个 Docker 映像创建一个单独的 tar 归档文件:**

```java
$ docker save -o ubuntu.tar ubuntu:18.04 ubuntu:16.04 ubuntu:latest
```

## 3.转移 *tar* 存档

我们创建的 tar 归档文件存在于机器 a 上。现在让我们用[将`baeldung.tar`文件](/web/20220529023903/https://www.baeldung.com/linux/transfer-files-ssh)传输到机器 b 上。我们可以使用像`scp`或`ftp`这样的协议。

**这一步非常灵活，并且很大程度上取决于机器 A 和机器 B 所处的环境**。

## 4.将`tar`档案加载到 Docker 映像中

到目前为止，我们已经创建了 Docker 映像的 tar 归档文件，并将其移动到我们的目标机器 b 上。

现在，我们将使用 [`docker load`](https://web.archive.org/web/20220529023903/https://docs.docker.com/engine/reference/commandline/load/) 命令从 tar 归档文件`baeldung.tar` 中创建实际的 Docker 映像:

```java
$ docker load --input baeldung.tar 
Loaded image: baeldung:latest
```

同样，我们也可以使用文件重定向来转换`tar`档案:

```java
$ docker load < baeldung.tar
Loaded image: baeldung:latest
```

现在让我们通过运行`docker images `命令来验证映像是否成功加载:

```java
$ docker images
baeldung                                        latest                            277bcd6563ce        About a minute ago       466MB
```

**注意，如果 Docker 映像`baeldung`已经存在于目标机器上(在我们的例子中是机器 B)，那么`docker load`命令会将现有映像的标签重命名为空字符串< none > :**

```java
$ docker load --input baeldung.tar 
cfd97936a580: Loading layer [==================================================>]  466MB/466MB
The image baeldung:latest already exists, renaming the old one with ID sha256:
  277bcd6563ce2b71e43b7b6b7e12b830f5b329d21ab690d59f0fd85b01045574 to empty string
```

## 5.缺点

使用这种方法，我们失去了重用缓存的 Docker 图像层的自由。因此，**每次我们运行`docker save`命令，它都会创建整个 Docker 映像的`tar`档案。**

**另一个缺点是，我们需要通过保存所有的`tar`档案来手动维护 Docker 镜像版本。**

因此，建议在测试环境中或者当我们限制访问 Docker Hub 时使用这种方法。

## 6.结论

在本教程中，我们学习了`docker save`和`docker load`命令，以及如何使用这些命令传输 Docker 图像。

我们还讨论了相关的缺点以及这种方法被证明是有效的理想情况。