# 探索 Docker 容器的文件系统

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/ops/docker-container-filesystem>

## 1.概观

当我们使用 Docker 时，有时我们需要检查容器中的配置或日志文件。

在这个快速教程中，我们将看到如何检查一个 [Docker 容器](/web/20220901074910/https://www.baeldung.com/docker-images-vs-containers)的文件系统，以帮助我们解决这种情况。

## 2.互动探索

如果我们获得了对大多数容器的 shell 访问，我们就可以交互地探索它们的文件系统。

### 2.1.使用 Shell 访问运行容器

让**使用带有`-it` 选项**的`docker run`命令通过 shell 访问直接启动一个容器:

```java
$ docker run -it alpine
/# ls -all
...
-rwxr-xr-x    1 root     root             0 Mar  5 13:21 .dockerenv
drwxr-xr-x    1 root     root           850 Jan 16 21:52 bin
drwxr-xr-x    5 root     root           360 Mar  5 13:21 dev
drwxr-xr-x    1 root     root           508 Mar  5 13:21 etc
drwxr-xr-x    1 root     root             0 Jan 16 21:52 home
.... 
```

在这里，我们以交互模式启动了 Alpine Linux 容器，并连接到它的 shell。

但是如果我们想探索一些不是 Linux 发行版的东西，会发生什么呢？

```java
$ docker run -it cassandra
 ... 
INFO [MigrationStage:1] 2020-03-05 13:44:36,734 - Initializing system_auth.resource_role_permissons_index 
INFO [MigrationStage:1] 2020-03-05 13:44:36,739 - Initializing system_auth.role_members 
INFO [MigrationStage:1] 2020-03-05 13:44:36,743 - Initializing system_auth.role_permissions 
INFO [MigrationStage:1] 2020-03-05 13:44:36,747 - Initializing system_auth.roles 
INFO [main] 2020-03-05 13:44:36,764 - Waiting for gossip to settle... 
...
```

Cassandra docker 容器带有一个默认的启动命令，它运行 Cassandra。因此，我们不再连接到外壳。

相反，我们只看到应用程序的日志消息填充的标准输出。

但是，**我们可以绕过默认的启动命令。**

**让我们将`/bin/bash` 附加参数传递给`docker run`命令** :

```java
$ docker run -it cassandra /bin/bash
[[email protected]](/web/20220901074910/https://www.baeldung.com/cdn-cgi/l/email-protection):/# ls -all
total 4
...
-rwxr-xr-x   1 root root    0 Mar  5 13:30 .dockerenv
drwxr-xr-x   1 root root  920 Aug 14  2019 bin
drwxr-xr-x   1 root root    0 Mar 28  2019 boot
drwxr-xr-x   5 root root  360 Mar  5 13:30 dev
lrwxrwxrwx   1 root root   34 Aug 14  2019 docker-entrypoint.sh -> usr/local/bin/docker-entrypoint.sh
drwxr-xr-x   1 root root 1690 Mar  5 13:30 etc
... 
```

不幸的是，这有一个严重的副作用。实际的 Cassandra 应用程序不再启动，我们必须从 shell 中手动完成这项工作。

当我们使用这种方法时，我们假设我们可以控制容器的启动。在生产环境中，这可能是不可能的。

### 2.2.在运行的容器中生成 Shell

幸运的是，我们可以使用**和`docker exec` 命令，其中**和**允许我们连接到正在运行的容器**。

让我们首先开始我们想要探索的容器:

```java
$ docker run cassandra
...
INFO  [MigrationStage:1] 2020-03-05 13:44:36,734 - Initializing system_auth.resource_role_permissons_index
INFO  [MigrationStage:1] 2020-03-05 13:44:36,739 - Initializing system_auth.role_members
INFO  [MigrationStage:1] 2020-03-05 13:44:36,743 - Initializing system_auth.role_permissions
INFO  [MigrationStage:1] 2020-03-05 13:44:36,747 - Initializing system_auth.roles
INFO  [main] 2020-03-05 13:44:36,764 - Waiting for gossip to settle...
... 
```

接下来，我们用 `**docker ps**`来标识容器 id:

```java
$ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             
00622c0645fb        cassandra           "docker-entrypoint.s…"   2 minutes ago 
```

然后，我们**将`/bin/bash`作为带有`-it` 选项的自变量传递给 `docker exec`** :

```java
$ docker exec -it 00622c0645fb /bin/bash
[[email protected]](/web/20220901074910/https://www.baeldung.com/cdn-cgi/l/email-protection):/# ls -all
...
-rwxr-xr-x   1 root root    0 Mar  5 13:44 .dockerenv
drwxr-xr-x   1 root root  920 Aug 14  2019 bin
drwxr-xr-x   1 root root    0 Mar 28  2019 boot
drwxr-xr-x   5 root root  340 Mar  5 13:44 dev
lrwxrwxrwx   1 root root   34 Aug 14  2019 docker-entrypoint.sh -> usr/local/bin/docker-entrypoint.sh
drwxr-xr-x   1 root root 1690 Mar  5 13:44 etc
...
```

在这里，**我们使用 Bash 作为我们选择的 shell**。**这取决于容器基于哪个 Linux 发行版。**

相比之下，我们的第一个例子使用 Alpine Linux，默认情况下它带有 Bourne Shell:

```java
$ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED            
8408c85b3c57        alpine              "/bin/sh"           3 seconds ago 
```

由于 Bash 不可用，我们将`/bin/sh`作为参数传递给`docker exec`:

```java
$ docker exec -it 8408c85b3c57 /bin/sh
/ # ls -all
...
-rwxr-xr-x    1 root     root             0 Mar  5 14:19 .dockerenv
drwxr-xr-x    1 root     root           850 Jan 16 21:52 bin
drwxr-xr-x    5 root     root           340 Mar  5 14:19 dev
drwxr-xr-x    1 root     root           508 Mar  5 14:19 etc
drwxr-xr-x    1 root     root             0 Jan 16 21:52 home
...
```

## 3.非互动探索

有时，**容器被停止，我们不能交互地运行它，**或者它根本没有外壳。

例如， [hello-world](https://web.archive.org/web/20220901074910/https://hub.docker.com/_/hello-world) 是 **从 [scratch](https://web.archive.org/web/20220901074910/https://hub.docker.com/_/scratch)** 开始的**最小容器。因此，shell 访问是不可能的。**

幸运的是，在这两种情况下，我们都可以将文件系统转储到我们的主机上，以便进一步研究。

让我们看看我们如何能做到这一点。

### 3.1.导出文件系统

我们可以通过使用 `**docker export**` 命令将容器的文件系统导出到 [tar](/web/20220901074910/https://www.baeldung.com/linux/zip-unzip-command-line) 文件中。

让我们首先运行 hello-world 容器:

```java
$ docker run hello-world

Hello from Docker!
This message shows that your installation appears to be working correctly.
.... 
```

类似地，我们首先通过将`-a`标志传递给`docker ps` **:** 来让**获得一个停止的集装箱的集装箱 id**

```java
$ docker ps -a
CONTAINER ID        IMAGE                 COMMAND                  CREATED             
a0af60c72d93        hello-world           "/hello"                 3 minutes ago       
... 
```

然后我们使用`docker export` 的`-o`选项将文件系统转储到 hello.tar 文件中**:**

```java
$ docker export -o hello.tar a0af60c72d93 
```

最后，**我们使用带有`-tvf` 标志**的`tar` 实用程序打印归档文件的内容:

```java
$ tar -tvf hello.tar
-rwxr-xr-x root/0            0 2020-03-05 16:55 .dockerenv
....
drwxr-xr-x root/0            0 2020-03-05 16:55 dev/pts/
drwxr-xr-x root/0            0 2020-03-05 16:55 dev/shm/
....
-rwxr-xr-x root/0            0 2020-03-05 16:55 etc/resolv.conf
-rwxrwxr-x root/0         1840 2019-01-01 03:27 hello
... 
```

或者，我们可以使用任何归档浏览器来查看里面的内容。

### 3.2.复制文件系统

**我们也可以使用`docker cp` 命令**复制整个文件系统。

让我们也试试这个。

**首先，我们将从`the root (/)`** 开始的完整文件系统从我们的容器复制到`test`目录:

```java
$ docker cp a0af60c72d93:/ ./test 
```

接下来，让我们打印`test`目录的内容:

```java
$ ls -all test/
total 28
..
drwxr-xr-x  4 baeldung users 4096 Mar  5 16:55 dev
-rwxr-xr-x  1 baeldung users    0 Mar  5 16:55 .dockerenv
drwxr-xr-x  2 baeldung users 4096 Mar  5 16:55 etc
-rwxrwxr-x  1 baeldung users 1840 Jan  1  2019 hello 
```

## 4.结论

在这个快速教程中，我们讨论了如何探索 Docker 容器的文件系统。

我们可以直接用`docker` `run` 命令启动大多数带有 shell 访问的容器。此外，我们可以在`docker exec.`的帮助下生成一个运行容器的外壳

对于停止的容器或最小容器，我们可以简单地导出甚至本地复制整个文件系统。