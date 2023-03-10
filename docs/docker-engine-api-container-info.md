# 从 Docker 引擎 API 获取 Docker 容器

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/ops/docker-engine-api-container-info>

## 1.概观

在本教程中，我们将看到如何使用 Docker 引擎 API 从容器内部访问 Docker 容器信息。

## 2.设置

我们可以通过多种方式连接 Docker 引擎。我们将讨论 Linux 下最有用的，但是它们也适用于其他操作系统。

然而，我们应该**非常小心**、**，因为启用远程访问会带来安全风险**。当容器可以访问引擎时，**它打破了与主机操作系统的隔离**。

对于设置部分，我们将考虑我们对主机的完全控制。

### 2.1.转发默认的 Unix 套接字

默认情况下，Docker 引擎使用安装在主机操作系统上的`/var/run/docker.sock`下的 Unix 套接字:

```java
$ ss -xan | grep var

u_str LISTEN 0      4096              /var/run/docker/libnetwork/dd677ae5f81a.sock 56352            * 0           
u_dgr UNCONN 0      0                                 /var/run/chrony/chronyd.sock 24398            * 0           
u_str LISTEN 0      4096                                      /var/run/nscd/socket 23131            * 0           
u_str LISTEN 0      4096                              /var/run/docker/metrics.sock 42876            * 0           
u_str LISTEN 0      4096                                      /var/run/docker.sock 53704            * 0    
... 
```

使用这种方法，我们可以严格控制哪个容器可以访问 API。这就是 Docker CLI 在幕后的工作方式。

让我们启动`alpine` Docker 容器，**使用`-v` 标志**挂载这个路径:

```java
$ docker run -it -v /var/run/docker.sock:/var/run/docker.sock alpine

(alpine) $
```

接下来，让我们在容器中安装一些实用程序:

```java
(alpine) $ apk add curl && apk add jq

fetch http://dl-cdn.alpinelinux.org/alpine/v3.11/community/x86_64/APKINDEX.tar.gz
(1/4) Installing ca-certificates (20191127-r2)
(2/4) Installing nghttp2-libs (1.40.0-r1)
...
```

现在让**使用带有`–unix-socket` 标志的`curl`** 和 [`Jq`](/web/20220727020703/https://www.baeldung.com/linux/jq-command-json) 来获取和过滤一些容器数据:

```java
(alpine) $ curl -s --unix-socket /var/run/docker.sock http://dummy/containers/json | jq '.'

[
  {
    "Id": "483c5d4aa0280ca35f0dbca59b5d2381ad1aa455ebe0cf0ca604900b47210490",
    "Names": [
      "/wizardly_chatelet"
    ],
    "Image": "alpine",
    "ImageID": "sha256:e7d92cdc71feacf90708cb59182d0df1b911f8ae022d29e8e95d75ca6a99776a",
    "Command": "/bin/sh",
    "Created": 1595882408,
    "Ports": [],
...
```

这里，**我们在`/containers/json`** 端点上发出一个`GET` ，**得到当前运行的容器**。然后我们使用`jq`美化输出。

稍后我们将讨论引擎 API 的细节。

### 2.2.启用 TCP 远程访问

我们还可以使用 TCP 套接字实现远程访问。

对于带有`systemd`的 Linux 发行版，我们需要定制 Docker 服务单元。对于其他 Linux 发行版，我们需要定制通常位于`/etc/docker`的`daemon.json`。

我们将只讨论第一种设置，因为大多数步骤都是相似的。

默认 Docker 设置包括一个桥接网络。这里是**，所有的集装箱都在这里连接，除非另有说明**。

因为我们希望只允许容器访问引擎 API，所以让我们首先确定它们的网络:

```java
$ docker network ls

a3b64ea758e1        bridge              bridge              local
dfad5fbfc671        host                host                local
1ee855939a2a        none                null                local 
```

让我们来看看它的细节:

```java
$ docker network inspect a3b64ea758e1

[
    {
        "Name": "bridge",
        "Id": "a3b64ea758e1f02f4692fd5105d638c05c75d573301fd4c025f38d075ed2a158",
...
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.17.0.0/16",
                    "Gateway": "172.17.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
...
```

接下来，让我们看看 Docker 服务单元位于何处:

```java
$ systemctl status docker.service

docker.service - Docker Application Container Engine
     Loaded: loaded (/usr/lib/systemd/system/docker.service; disabled; vendor preset: disabled)
...
     CGroup: /system.slice/docker.service
             ├─6425 /usr/bin/dockerd --add-runtime oci=/usr/sbin/docker-runc
             └─6452 docker-containerd --config /var/run/docker/containerd/containerd.toml --log-level warn 
```

现在让我们来看看服务单元的定义:

```java
$ cat /usr/lib/systemd/system/docker.service

[Unit]
Description=Docker Application Container Engine
Documentation=http://docs.docker.com
After=network.target lvm2-monitor.service SuSEfirewall2.service

[Service]
EnvironmentFile=/etc/sysconfig/docker
...
Type=notify
ExecStart=/usr/bin/dockerd --add-runtime oci=/usr/sbin/docker-runc $DOCKER_NETWORK_OPTIONS $DOCKER_OPTS
ExecReload=/bin/kill -s HUP $MAINPID
...
```

属性定义了由`systemd`(`dockerd`可执行文件)运行的命令。**我们将`-H` 标志传递给它，并指定相应的网络和端口在**上监听。

我们可以直接修改这个服务单元(不推荐)，但是让我们使用`$DOCKER_OPTS` 变量(在`EnvironmentFile=/etc/sysconfig/docker`中定义):

```java
$ cat /etc/sysconfig/docker 

## Path           : System/Management
## Description    : Extra cli switches for docker daemon
## Type           : string
## Default        : ""
## ServiceRestart : docker
#
DOCKER_OPTS="-H unix:///var/run/docker.sock -H tcp://172.17.0.1:2375"
```

这里，**我们使用桥接网络的网关地址作为绑定地址**。**这对应于主机**上的`docker0`界面:

```java
$ ip address show dev docker0

3: docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:42:6c:7d:9c:8d brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
    inet6 fe80::42:6cff:fe7d:9c8d/64 scope link 
       valid_lft forever preferred_lft forever 
```

我们还启用了本地 Unix 套接字，以便 Docker CLI 仍然可以在主机上工作。

我们还需要做一步。让我们**允许我们的容器包到达主机**:

```java
$ iptables -I INPUT -i docker0 -j ACCEPT
```

这里，我们设置 Linux 防火墙接受所有通过`docker0`接口的包。

现在，让我们重新启动 Docker 服务:

```java
$ systemctl restart docker.service
$ systemctl status docker.service
 docker.service - Docker Application Container Engine
     Loaded: loaded (/usr/lib/systemd/system/docker.service; disabled; vendor preset: disabled)
...
     CGroup: /system.slice/docker.service
             ├─8110 /usr/bin/dockerd --add-runtime oci=/usr/sbin/docker-runc -H unix:///var/run/docker.sock -H tcp://172.17.0.1:2375
             └─8137 docker-containerd --config /var/run/docker/containerd/containerd.toml --log-level wa
```

让我们再次运行我们的`alpine` 容器:

```java
(alpine) $ curl -s http://172.17.0.1:2375/containers/json | jq '.'

[
  {
    "Id": "45f13902b710f7a5f324a7d4ec7f9b934057da4887650dc8fb4391c1d98f051c",
    "Names": [
      "/unruffled_cray"
    ],
    "Image": "alpine",
    "ImageID": "sha256:a24bb4013296f61e89ba57005a7b3e52274d8edd3ae2077d04395f806b63d83e",
    "Command": "/bin/sh",
    "Created": 1596046207,
    "Ports": [],
... 
```

我们应该**意识到所有连接到桥接网络的容器都可以访问守护程序 API** 。

此外，**我们的 TCP 连接没有加密**。

## 3.坞站引擎 API

现在我们已经设置了远程访问，让我们来看看 API。

我们将只探索几个有趣的选项，但我们可以随时查看[完整文档](https://web.archive.org/web/20220727020703/https://docs.docker.com/engine/api/latest)了解更多信息。

让我们**获取一些关于我们的容器**的信息:

```java
(alpine) $ curl -s http://172.17.0.1:2375/containers/"$(hostname)"/json | jq '.'

{
  "Id": "45f13902b710f7a5f324a7d4ec7f9b934057da4887650dc8fb4391c1d98f051c",
  "Created": "2020-07-29T18:10:07.261589135Z",
  "Path": "/bin/sh",
  "Args": [],
  "State": {
    "Status": "running",
... 
```

在这里**我们使用`/containers/{container-id}/json`** URL 来获取关于我们容器的详细信息。

在这种情况下，我们运行`hostname`命令来获取`container-id`。

接下来，让我们**监听 Docker 守护进程**上的事件:

```java
(alpine) $ curl -s http://172.17.0.1:2375/events | jq '.'
```

现在，在另一个终端中，让我们启动`hello-world` 容器:

```java
$ docker run hello-world

Hello from Docker!
This message shows that your installation appears to be working correctly.
... 
```

回到我们的`alpine` 容器，我们得到一系列事件:

```java
{
  "status": "create",
  "id": "abf881cbecfc0b022a3c1a6908559bb27406d0338a917fc91a77200d52a2553c",
  "from": "hello-world",
  "Type": "container",
  "Action": "create",
...
}
{
  "status": "attach",
  "id": "abf881cbecfc0b022a3c1a6908559bb27406d0338a917fc91a77200d52a2553c",
  "from": "hello-world",
  "Type": "container",
  "Action": "attach",
...
```

到目前为止，我们一直在做非侵入性的事情。是时候改变一下了。

让我们创建并启动一个容器。首先，我们定义它的清单:

```java
(alpine) $ cat > create.json << EOF
{
  "Image": "hello-world",
  "Cmd": ["/hello"]
}
EOF 
```

现在让**使用清单**调用`/containers/create`端点:

```java
(alpine) $ curl -X POST -H "Content-Type: application/json" -d @create.json http://172.17.0.1:2375/containers/create

{"Id":"f96a6360ad8e36271cc75a3cff05348761569cf2f089bbb30d826bd1e2d52f59","Warnings":[]}
```

然后，我们使用 id 来启动容器:

```java
(alpine) $ curl -X POST http://172.17.0.1:2375/containers/f96a6360ad8e36271cc75a3cff05348761569cf2f089bbb30d826bd1e2d52f59/start
```

最后，我们可以浏览日志:

```java
(alpine) $ curl http://172.17.0.1:2375/containers/f96a6360ad8e36271cc75a3cff05348761569cf2f089bbb30d826bd1e2d52f59/logs?stdout=true --output -

Hello from Docker!
KThis message shows that your installation appears to be working correctly.

;To generate this message, Docker took the following steps:
3 1\. The Docker client contacted the Docker daemon.
...
```

注意我们在每一行的开头都有一些奇怪的字符。这是因为传输日志的流被[复用](https://web.archive.org/web/20220727020703/https://docs.docker.com/engine/api/v1.40/#operation/ContainerAttach)以区分`stderr` 和`stdout`。

因此，输出需要进一步处理。

我们可以通过在创建容器时启用`TTY`选项来避免这种情况:

```java
(alpine) $ cat create.json

{
  "Tty":true,	
  "Image": "hello-world",
  "Cmd": ["/hello"]
}
```

## 4.结论

在本教程中，我们学习了如何使用 Docker 引擎远程 API。

我们从设置来自 UNIX 套接字或 TCP 的远程访问开始，并进一步展示了如何使用远程 API。