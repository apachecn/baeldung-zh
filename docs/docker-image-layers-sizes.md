# 查找 Docker 图像的图层和图层大小

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/ops/docker-image-layers-sizes>

## 1.概观

集装箱化技术帮助我们以更低的成本快速构建和配置部署环境。秉承“一次编写，随处部署”的座右铭，我们使用容器化来解决现代应用程序的复杂性。

在本教程中，**我们将深入探讨 Docker 图像层，它是容器化技术的基础构件。**

## 2.图像层

Docker 图像是通过连接许多只读层创建的，这些只读层堆叠在一起形成一个完整的图像。像 Docker 和 Podman 这样的平台将这些层放在一起，将它们描绘成一个统一的对象。

举个例子，让我们从注册表中取出一个 MySQL 图像，快速浏览一下:

```java
# docker pull mysql
Using default tag: latest
latest: Pulling from library/mysql
492d84e496ea: Pull complete
bbe20050901c: Pull complete
e3a5e171c2f8: Pull complete
c3aceb7e4f48: Pull complete
269002e5cf58: Pull complete
d5abeb1bd18e: Pull complete
cbd79da5fab6: Pull complete
Digest: sha256:cdf3b62d78d1bbb1d2bd6716895a84014e00716177cbb7e90f6c6a37a21dc796
Status: Downloaded newer image for mysql:latest
```

**上面代码片段中以“Pull complete”结尾的每一行代表从注册表中提取的层，以形成一个图像。**正如我们所见，MySQL 图像有七层。

### 2.1.图像层创建

现在让我们通过一个`Dockerfile`图更深入地了解这些层是如何构建的。

`Dockerfile`中的`RUN`、`COPY`和`ADD`等指令创建新层，而其他指令仅创建中间层。前一个命令会影响图层大小，但后一个命令不会。

让我们通过一个`Dockerfile`来建立一个形象。我们可以从这个[链接](/web/20220911170553/https://www.baeldung.com/ops/docker-cron-job)中引用`Dockerfile`。我们使用`docker build`命令通过`Dockerfile`创建图像:

```java
# docker build -t layer-demo/latest .
Sending build context to Docker daemon  3.072kB
Step 1/8 : FROM ubuntu:latest
 ---> df5de72bdb3b
Step 2/8 : MAINTAINER baeldung.com
 ---> Running in 2c90e21f29e2
Removing intermediate container 2c90e21f29e2
 ---> 460d0651cc3d
Step 3/8 : ADD get_date.sh /root/get_date.sh
 ---> 492d1b205a94
Step 4/8 : RUN chmod 0644 /root/get_date.sh
 ---> Running in 08d04f1db0de
Removing intermediate container 08d04f1db0de
 ---> 480ba7f4bc50
Step 5/8 : RUN apt-get update
...
... output truncated ...
...
 ---> 28182a44db71
Step 6/8 : RUN apt-get -y install cron
...
... output truncated ...
...
 ---> 822f3eeca346
Step 7/8 : RUN crontab -l | { cat; echo "* * * * * bash /root/get_date.sh"; } | crontab -
 ---> Running in 635190dfb8d7
no crontab for root
Removing intermediate container 635190dfb8d7
 ---> 2822aac1f51b
Step 8/8 : CMD cron
 ---> Running in 876f0d5aca27
Removing intermediate container 876f0d5aca27
 ---> 5fc87be0f286
Successfully built 5fc87be0f286
```

这里发生了什么事？为了创建这个映像，我们注意到它需要八个步骤，每个步骤对应于`Dockerfile`中的一条指令。最初，从注册表中取出 Ubuntu 映像[映像 ID: `df5de72bdb3b` ]:

1.  它使用前一步骤的图像[图像 ID: `df5de72bdb3b` ]旋转中间容器[容器 ID: `2c90e21f29e2` ]。
2.  之后，在中间容器[容器 ID: `2c90e21f29e2` ]上执行指令。
3.  随后，中间容器通过提交转换成图像[图像 ID: `460d0651cc3d` ]，导致中间容器[容器 ID: `2c90e21f29e2` ]的移除。
4.  移除中间容器后，图像变成只读层。然后执行`Dockerfile`中的下一条指令。

但是，创建新层的步骤与上面相同。**中间层不能影响图像尺寸，而使用`RUN`、`ADD`和`COPY`的普通层能够增加尺寸。**

## 3.图层大小

通常，**图像的大小完全由与其相关的层决定**。`docker history`命令显示与图像相关的每个层的大小。

在下面的例子中，大小为 0B 的层代表一个中间层，而`RUN`、`COPY`和`ADD`指令对图像大小有影响:

```java
# docker history layer-demo/latest
IMAGE          CREATED       CREATED BY                                      SIZE      COMMENT
5fc87be0f286   8 hours ago   /bin/sh -c #(nop)  CMD ["/bin/sh" "-c" "cron…   0B      
2822aac1f51b   8 hours ago   /bin/sh -c crontab -l | { cat; echo "* * * *…   208B
822f3eeca346   8 hours ago   /bin/sh -c apt-get -y install cron              987kB
28182a44db71   8 hours ago   /bin/sh -c apt-get update                       36MB
480ba7f4bc50   8 hours ago   /bin/sh -c chmod 0644 /root/get_date.sh         5B
492d1b205a94   8 hours ago   /bin/sh -c #(nop) ADD file:1f79f73be93042145…   5B
460d0651cc3d   8 hours ago   /bin/sh -c #(nop)  MAINTAINER baeldung.com      0B
df5de72bdb3b   4 weeks ago   /bin/sh -c #(nop)  CMD ["bash"]                 0B
      4 weeks ago   /bin/sh -c #(nop) ADD file:396eeb65c8d737180…   77.8MB 
```

现在，让我们合计所有层的大小，从基本图像开始:

*   df5de 72 BDB 3b–77.800000 MB # #步骤 1:基本 Ubuntu 映像
*   492 D1 b205a 94–0.000005 MB # #步骤 3:添加指令
*   480 ba 7 F4 BC 50–0.000005 MB # #步骤 4:运行指令
*   28182 a 44 db 71–36.000000 MB # #步骤 5:运行指令
*   822 F3 eeca 346–0.987000 MB # #步骤 6:运行指令
*   2822 AAC 1 f 51 b–0.000208 MB # #步骤 7:运行指令

将上述所有数字加在一起得到 114.787 MB，可以进一步四舍五入到 115 MB。如我们所见，计算出的总和与来自`docker image`命令的`layer-demo:latest`图像大小完全匹配:

```java
# docker images 
REPOSITORY            TAG       IMAGE ID       CREATED       SIZE
layer-demo/latest     latest    5fc87be0f286   8 hours ago   115MB
ubuntu                latest    df5de72bdb3b   4 weeks ago   77.8MB
```

## 4.悬空图像

**悬空图像是在图像形成期间创建的图像层。但是，创建图像后，这些图层将不会与任何标记的图像有任何关系。**因此，删除所有这些图像是安全的，因为它们会消耗不必要的磁盘空间。

要列出所有悬挂图像，我们可以使用`docker image`命令，在搜索过滤器中将悬挂属性设置为 true:

```java
# docker images --filter "dangling=true"
```

下面的命令显示悬挂的图像，然后删除它们:

```java
# docker images --quiet --filter=dangling=true | xargs --no-run-if-empty docker rmi
```

## 5.结论

在本文中，我们看了 Docker 图像层的概念和层的创建。此外，我们还讨论了可以用来识别与图像相关的层列表和每个层的大小的命令。

最后，我们看到了中间层是如何创建的，并了解到如果我们不经常清除悬挂图像，它们就会留在我们的系统中。