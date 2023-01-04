# 如何在 Docker 容器中运行 Cron 作业？

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/ops/docker-cron-job>

## 1.概观

作为系统管理员，我们总是会遇到安排任务的需要。我们可以通过在 Linux 系统中使用`cron`服务来实现这一点。此外，我们可以在容器系统中启用`cron`调度服务。

现在，本教程将阐述在 Docker 容器中启用`cron`服务的两种不同方式。在第一种方法中，我们将使用 Dockerfile 在 docker 映像中嵌入`cron`服务，而另一种方法将演示如何在容器中安装调度服务。

## 2.Cron 服务——使用`Dockerfile`方法

使用`Dockerfile`构建图像是创建容器图像最简单的方法之一。那么，我们该怎么做呢？基本上，`Dockerfile`是一个简单的文本文件，包含一组构建图像的指令。我们需要提供调度任务、`cron`细节，并从`Dockerfile`调用`cron`服务。

### 2.1.写作`Dockerfile`

让我们快速看一个例子:

```java
$ tree
.
├── Dockerfile
└── get_date.sh
0 directories, 2 files 
```

一般来说，`Dockerfile`的第一行以`FROM`命令开始，它将从已配置的注册表中获取所请求的图像。在我们的例子中，默认的注册表被配置为`DockerHub`。然后是`MAINTAINER`，是捕捉作者信息的元数据。`ADD`指令将`get_date.sh`脚本从主机的映像构建路径复制到映像的目标路径。

在将脚本复制到构建映像之后，`RUN`指令给出了可执行的权限。不仅如此——`RUN`指令帮助执行任何 shell 命令，作为当前层之上的新图像层，并提交结果。`RUN`更新`apt`存储库并在镜像中安装最新的`cron`服务。它还执行`crontab`中的`cron`调度。

最后，我们将使用`CMD`指令启动`cron`服务:

```java
$ cat Dockerfile

# Dockerfile to create image with cron services
FROM ubuntu:latest
MAINTAINER baeldung.com

# Add the script to the Docker Image
ADD get_date.sh /root/get_date.sh

# Give execution rights on the cron scripts
RUN chmod 0644 /root/get_date.sh

#Install Cron
RUN apt-get update
RUN apt-get -y install cron

# Add the cron job
RUN crontab -l | { cat; echo "* * * * * bash /root/get_date.sh"; } | crontab -

# Run the command on container startup
CMD cron 
```

### 2.2.构建和运行 Cron 映像

一旦`Dockerfile`准备好了，我们就可以使用`docker build`命令构建映像。**圆点(。)指示 Docker 引擎从当前路径获取`Dockerfile`。构建命令为`Dockerfile`上给出的每个指令创建 docker 层，以形成最终的构建映像**。典型的构建输出如下所示:

```java
$ docker build .
Sending build context to Docker daemon  3.072kB
Step 1/8 : FROM ubuntu:latest
---> ba6acccedd29
Step 2/8 : MAINTAINER baeldung.com
---> Using cache
---> e6b3946b2382
Step 3/8 : ADD get_date.sh /root/get_date.sh
---> 4976f058d428
Step 4/8 : RUN chmod 0644 /root/get_date.sh
---> Running in 423a4e9adbab
Removing intermediate container 423a4e9adbab
---> 76d972a082ba
Step 5/8 : RUN apt-get update
---> Running in badc0d84f6ff
Get:1 http://security.ubuntu.com/ubuntu focal-security InRelease [114 kB]
...
... output truncated ...
...
Removing intermediate container badc0d84f6ff
---> edb8a19b891c
Step 6/8 : RUN apt-get -y install cron
---> Running in efd9b8a67d98
Reading package lists...
Building dependency tree...
...
... output truncated ...
...
Done.
invoke-rc.d: could not determine current runlevel
invoke-rc.d: policy-rc.d denied execution of start.
Removing intermediate container efd9b8a67d98
---> 2b80000d32a1
Step 7/8 : RUN crontab -l | { cat; echo "* * * * * bash /root/get_date.sh"; } | crontab -
---> Running in 1bdd3e0cc877
no crontab for root
Removing intermediate container 1bdd3e0cc877
---> aa7c82aa7c11
Step 8/8 : CMD cron
---> Running in cf2d44873b36
Removing intermediate container cf2d44873b36
---> 8cee091ca87d
Successfully built 8cee091ca87d 
```

因为我们已经将`cron`服务预安装到映像中，并将任务嵌入到`crontab`中，所以当我们运行容器时，`cron`作业会自动激活。或者，我们可以使用`docker run`命令启动容器。随后，`docker run`的“`-it`选项有助于进入带有 bash 提示符的容器。

下图显示了使用`cron`在容器中执行`get_date.sh`脚本:

```java
$ docker run -it 8cee091ca87d /bin/bash
[[email protected]](/web/20220628125038/https://www.baeldung.com/cdn-cgi/l/email-protection):/#
[[email protected]](/web/20220628125038/https://www.baeldung.com/cdn-cgi/l/email-protection):/# date
Mon Nov 15 14:30:21 UTC 2021

[[email protected]](/web/20220628125038/https://www.baeldung.com/cdn-cgi/l/email-protection):/# ls -ltrh ~/date.out
ls: cannot access '/root/date.out': No such file or directory

[[email protected]](/web/20220628125038/https://www.baeldung.com/cdn-cgi/l/email-protection):/# ls -ltrh /root/get_date.sh
-rw-r--r-- 1 root root 18 Nov 15 14:20 /root/get_date.sh

[[email protected]](/web/20220628125038/https://www.baeldung.com/cdn-cgi/l/email-protection):/# crontab -l
* * * * * bash /root/get_date.sh

[[email protected]](/web/20220628125038/https://www.baeldung.com/cdn-cgi/l/email-protection):/# ls -ltrh ~/date.out
-rw-r--r-- 1 root root 29 Nov 15 14:31 /root/date.out

[[email protected]](/web/20220628125038/https://www.baeldung.com/cdn-cgi/l/email-protection):/# cat /root/date.out
Mon Nov 15 14:31:01 UTC 2021 
```

## 3.Cron 服务——动态容器方法

或者，我们可以在 Docker 容器中设置`cron`服务来运行`cron`作业。那么，作案手法是什么？

让我们使用`docker run`命令快速运行一个`ubuntu`容器。通常，容器是一个轻量级的操作系统，不会将`cron`服务作为默认包。

因此，我们需要**进入容器的交互外壳，并使用`apt`存储库命令**安装`cron`服务:

```java
$ docker run -it ubuntu:latest /bin/bash
Unable to find image 'ubuntu:latest' locally
latest: Pulling from library/ubuntu
7b1a6ab2e44d: Pull complete
Digest: sha256:626ffe58f6e7566e00254b638eb7e0f3b11d4da9675088f4781a50ae288f3322
Status: Downloaded newer image for ubuntu:latest
[[email protected]](/web/20220628125038/https://www.baeldung.com/cdn-cgi/l/email-protection):/#
[[email protected]](/web/20220628125038/https://www.baeldung.com/cdn-cgi/l/email-protection):/# which cron
[[email protected]](/web/20220628125038/https://www.baeldung.com/cdn-cgi/l/email-protection):/# apt update -y
Get:1 http://archive.ubuntu.com/ubuntu focal InRelease [265 kB]
...
... output truncated ...
...
All packages are up to date.
[[email protected]](/web/20220628125038/https://www.baeldung.com/cdn-cgi/l/email-protection):/# apt upgrade -y
... 
... output truncated ...
...
[[email protected]](/web/20220628125038/https://www.baeldung.com/cdn-cgi/l/email-protection):/# apt install cron vim -y
Reading package lists... Done
...
... output truncated ...
...
Done.
invoke-rc.d: could not determine current runlevel
invoke-rc.d: policy-rc.d denied execution of start.
[[email protected]](/web/20220628125038/https://www.baeldung.com/cdn-cgi/l/email-protection):/# which cron
/usr/sbin/cron

$ docker cp get_data.sh 77483fc20fc9: /root/get_date.sh 
```

我们可以使用`docker cp`命令将`get_date.sh`从主机复制到容器中。`crontab -e`使用`vi`编辑器编辑`cron`任务。下面的`cron`配置每分钟运行一次脚本。此外，输出指示脚本执行的时间戳:

```java
[[email protected]](/web/20220628125038/https://www.baeldung.com/cdn-cgi/l/email-protection):/# export EDITOR=vi
[[email protected]](/web/20220628125038/https://www.baeldung.com/cdn-cgi/l/email-protection):/# crontab -e
* * * * * bash /root/get_date.sh

[[email protected]](/web/20220628125038/https://www.baeldung.com/cdn-cgi/l/email-protection):/# date
Mon Nov 17 11:15:21 UTC 2021

[[email protected]](/web/20220628125038/https://www.baeldung.com/cdn-cgi/l/email-protection):/# ls -ltrh ~/date.out
ls: cannot access '/root/date.out': No such file or directory

[[email protected]](/web/20220628125038/https://www.baeldung.com/cdn-cgi/l/email-protection):/# ls -ltrh /root/get_date.sh
-rw-r--r-- 1 root root 18 Nov 17 11:09 /root/get_date.sh

[[email protected]](/web/20220628125038/https://www.baeldung.com/cdn-cgi/l/email-protection):/# ls -ltrh ~/date.out
-rw-r--r-- 1 root root 29 Nov 17 11:16 /root/date.out

[[email protected]](/web/20220628125038/https://www.baeldung.com/cdn-cgi/l/email-protection):/# cat /root/date.out
Mon Nov 17 11:16:01 UTC 2021 
```

## 4.结论

总的来说，我们已经探索了在 Docker 容器中运行`cron`作业的具体细节。使用 Dockerfile 的**方法将`cron`服务和任务嵌入到镜像**中，镜像按照`cron`调度配置自动执行脚本。

虽然 **`cron`可以在运行的容器中安装和配置，但它只是一个运行时构造，除非我们使用`docker commit`** 构造一个映像。

此外，根据我们的使用环境，这两种方法都有各自的优点。