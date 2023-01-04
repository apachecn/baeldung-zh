# 在 Docker 中设置内存和 CPU 限制

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/ops/docker-memory-limit>

## 1.概观

在很多情况下，我们需要限制 docker 主机上的资源使用。

在本教程中，我们将学习如何设置 docker 容器的内存和 CPU 限制。

## 2.用`docker run`设置资源限制

我们可以使用 [`docker run`](/web/20220816155046/https://www.baeldung.com/linux/shell-alpine-docker) 命令直接设置资源限制。这是一个简单的解决方案。但是，该限制仅适用于映像的一次特定执行。

### 2.1.记忆

例如，让我们将容器可以使用的内存限制为 512 兆字节。

**为了约束内存，我们需要使用`m`参数**:

```
$ docker run -m 512m nginx
```

我们还可以设置一个称为预留的软限制。

当 docker 检测到主机内存不足时，它会被激活:

```
$ docker run -m 512m --memory-reservation=256m nginx
```

### 2.2.中央处理器

默认情况下，对主机计算能力的访问是无限制的。**我们可以使用`cpus`参数设置 CPU 限制。**

让我们将容器限制为最多使用两个 CPU:

```
$ docker run --cpus=2 nginx
```

我们还可以指定 CPU 分配的优先级。

默认值为 1024，数字越大优先级越高:

```
$ docker run --cpus=2 --cpu-shares=2000 nginx
```

与内存预留类似，当计算能力不足并且需要在竞争进程之间分配时，CPU 份额起主要作用。

## 3.用 docker-compose 文件设置内存限制

我们可以使用 [`docker-compose`](/web/20220816155046/https://www.baeldung.com/docker-compose) 文件达到类似的效果。请记住，格式和可能性会因`docker-compose`的版本而异。

### 3.1.带`docker swarm`的版本 3 和更新版本

我们给 Nginx 服务限制一半 CPU 和 512 兆内存，预留四分之一 CPU 和 128 兆内存。

**我们需要在我们的服务配置**中创建`deploy`和`resources`段:

```
services:
  service:
    image: nginx
    deploy:
        resources:
            limits:
              cpus: 0.50
              memory: 512M
            reservations:
              cpus: 0.25
              memory: 128M
```

**为了利用 docker-compose 文件中的`deploy`段，我们需要使用 [`docker stack`](https://web.archive.org/web/20220816155046/https://docs.docker.com/engine/reference/commandline/stack_deploy/) 命令。**

为了将堆栈部署到集群，我们运行`deploy`命令:

```
$ docker stack deploy --compose-file docker-compose.yml bael_stack
```

### 3.2.带`docker-compose`的版本 2

在旧版本的`docker-compose`中，我们可以将资源限制放在与服务的主要属性相同的级别上。

它们的命名也略有不同:

```
service:
  image: nginx
  mem_limit: 512m
  mem_reservation: 128M
  cpus: 0.5
  ports:
    - "80:80"
```

为了创建已配置的容器，我们需要运行`docker-compose`命令:

```
$ docker-compose up
```

## 4.验证资源使用情况

设置限制后，我们可以使用`docker stats`命令验证它们:

```
$ docker stats
CONTAINER ID        NAME                                             CPU %               MEM USAGE / LIMIT   MEM %               NET I/O             BLOCK I/O           PIDS
8ad2f2c17078        bael_stack_service.1.jz2ks49finy61kiq1r12da73k   0.00%               2.578MiB / 512MiB   0.50%               936B / 0B           0B / 0B             2
```

## 5.结论

在本文中，我们探讨了限制 docker 访问主机资源的方法。

我们查看了`docker run`和`docker-compose`命令的用法。最后，我们用`docker stats`控制了资源消耗。