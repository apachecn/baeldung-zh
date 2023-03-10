# 将卷添加到 Docker 时排除子文件夹

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/ops/docker-exclude-sub-folder-when-adding-volume>

## 1.概观

当我们需要将容器资源链接到主机时，我们挂载 [Docker 卷](/web/20221222154222/https://www.baeldung.com/ops/docker-volumes)。我们可以使用不同的卷，如命名卷或绑定装载。此外，无论它们是否持久，我们都可以使用本地或远程资源。但是，在挂载时，我们可能需要排除一些不需要的文件或文件夹。

在本教程中，**我们将通过一些 [Docker Compose](/web/20221222154222/https://www.baeldung.com/ops/docker-compose) 的例子来学习如何在挂载卷时排除文件夹。**

## 2.创建 nodejs docker 映像

那么为什么我们需要排除一些文件或者一个文件夹带有 [Docker](/web/20221222154222/https://www.baeldung.com/ops/docker-guide) ？首先，我们来讨论 Docker 图像。

当我们构建映像时，我们通常会添加应用程序文件。为了演示，我们将使用 [Nodejs](https://web.archive.org/web/20221222154222/https://nodejs.org/en/) 创建一个 [Docker 示例应用程序](https://web.archive.org/web/20221222154222/https://www.docker.com/blog/getting-started-with-docker-using-node-jspart-i/)。

一旦我们设置了主应用程序，让我们看看我们的`[Dockerfile](https://web.archive.org/web/20221222154222/https://docs.docker.com/engine/reference/builder/)`:

```java
FROM node:12.18.1
ENV NODE_ENV=production

WORKDIR /app

COPY ["package.json", "package-lock.json*", "./"]

RUN npm install --production

COPY . .

CMD [ "node", "server.js" ]
```

现在，我们可以构建一个图像，我们称之为`node-docker`:

```java
$ docker build -t node-docker .
```

在这种情况下，构建上下文是我们的本地`/app`文件夹。然而，它可能是位于指定路径或 URL 中的一组文件，例如 Git 存储库。

当我们构建 Docker 映像时，我们会将文件发送到存储映像的 Docker 服务器。Docker 基于`copy`或`run`等命令创建分层图像。

让我们运行`docker [history](https://web.archive.org/web/20221222154222/https://docs.docker.com/engine/reference/commandline/history/)`命令来查看图像的不同图层:

```java
$ docker history --format "ID-> {{.ID}} | Created-> {{.CreatedSince}} | Created By-> {{.CreatedBy}} | Size: {{.Size}}" e870a50eed97
```

我们可以使用`–format`选项创建自定义输出，并显示相关命令或大小等信息:

```java
ID-> e870a50eed97 | Created-> 36 hours ago | Created By-> /bin/sh -c #(nop)  CMD ["node" "server.js"] | Size: 0B
ID-> 708a43cd0ef2 | Created-> 36 hours ago | Created By-> /bin/sh -c #(nop) COPY dir:7cc2842dd32649457… | Size: 11.3MB
ID-> d49b84f48e41 | Created-> 36 hours ago | Created By-> /bin/sh -c npm install --production | Size: 14.7MB
ID-> a351be0717a1 | Created-> 36 hours ago | Created By-> /bin/sh -c #(nop) COPY multi:9959dc16241ba60… | Size: 80.7kB
ID-> 56b22d35f315 | Created-> 36 hours ago | Created By-> /bin/sh -c #(nop) WORKDIR /app | Size: 0B
ID-> c28b64493ce8 | Created-> 36 hours ago | Created By-> /bin/sh -c #(nop)  ENV NODE_ENV=production | Size: 0B
ID-> f5be1883c8e0 | Created-> 2 years ago | Created By-> /bin/sh -c #(nop)  CMD ["node"] | Size: 0B 
```

## 3.从映像构建中排除文件和文件夹

让我们考虑这样一种情况，我们有一个大文件，例如，一个日志、一个 zip 或一个 jar 文件。或者我们可能有不想在最终版本中公开的文件，比如密钥或密码。

**我们可以使用 [`.dockerignore`](https://web.archive.org/web/20221222154222/https://docs.docker.com/engine/reference/builder/#dockerignore-file) 文件来避免将这些文件发送到 Docker 服务器。**与 [`.gitignore`](https://web.archive.org/web/20221222154222/https://git-scm.com/docs/gitignore) 文件类似。

假设我们的项目中有一个带密码的文件。我们可以创建一个，例如:

```java
$ echo 'password' >secret.txt
```

现在，我们想从映像中排除这个文件。我们可以将它添加到我们的。`dockerignore`文件:

```java
# Ignoring the password file 
secret.txt
```

这样，我们可以从构建上下文中排除资源。此外，遵循[最佳实践](https://web.archive.org/web/20221222154222/https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)，可以提高性能。要上传的图像将会更小。

此外，如果我们添加一个要复制的文件，我们也不会有新层的缓存失效问题。我们也可以用这种方法排除文件夹:

```java
# Ignore the logs directory
logs/
```

让我们用。`dockerignore`文件。然后，我们可以启动一个容器:

```java
$ docker run -d --publish 8000:8000 node-docker
```

或者，如果我们使用 Docker Compose，我们可以用一个`docker-compose.yml`文件运行`[docker-compose up](https://web.archive.org/web/20221222154222/https://docs.docker.com/engine/reference/commandline/compose_up/)`，例如:

```java
services:
  node-app:
    image: node-docker:latest
    ports:
      - 8080:8080
```

我们可以通过在容器中执行`bash`命令来进行双重检查:

```java
$ docker exec -it d8938bc93406 bash
```

一旦进入容器，如果我们检查内容，例如，用`ls` 命令，没有`secret.txt`文件或其他被排除的资源必须在运行的容器中。

## 4.排除 Docker 卷中的文件和文件夹

我们已经了解了如何在构建映像时排除文件和文件夹。我们可能想对运行使用[卷](/web/20221222154222/https://www.baeldung.com/ops/docker-volumes)的容器做同样的事情。

**原因之一可能是添加文件或文件夹，而不影响我们主机中的内容。**

假设我们现在想要向 Docker 容器添加一个项目文件夹。我们可以使用[挂载绑定](https://web.archive.org/web/20221222154222/https://docs.docker.com/storage/bind-mounts/)来实现它。

让我们用 Nodejs 应用程序制作一个 Docker Compose 示例。我们来看一下`docker-compose.yml`文件:

```java
services:
  node-app:
    build: .
    ports:
      - 8081:8080
    volumes:
      - .:/app
```

我们可以在项目的根目录下运行我们的容器:

```java
$ docker-compose up -d
```

容器启动，如预期的那样，如果容器中有要挂载的文件或目录，Docker 会将内容复制到卷中。

假设现在，由于某种原因，我们删除了容器中的一个文件或文件夹。同样的结果发生在我们的宿主身上，我们会失去那些资源。

让我们来看几个避免删除主机中资源的解决方案。

### 4.1.排除文件

我们可以从排除容器中文件的挂载开始。一旦我们设置了音量，我们可以使用`/dev/null`命令来解决这个问题:

同样，让我们看看我们的`docker-compose.yml`文件:

```java
services:
  node-app:
    build: .
    ports:
      - 8080:8080
    volumes:
      - .:/app/
      - /dev/null:/app/secret.txt
```

当使用`/dev/null`时，我们丢弃任何写入文件的东西。在这种情况下，我们将以空的`secret.txt`结束。此外，由于该文件与`/dev/null`绑定，因此无法修改。

### 4.2.排除文件夹

更有趣的是，我们可以排除文件夹和子文件夹。我们可以通过在特定目录或子目录上创建一个匿名或[命名的卷](https://web.archive.org/web/20221222154222/https://docs.docker.com/storage/volumes/)来实现这一点。

让我们看看我们的 YAML 文件:

```java
services:
  node-app:
    build: .
    ports:
      - 8080:8080
    volumes:
      - .:/app
      - /app/node_modules/
```

**这里的顺序是相关的。首先，我们对之前创建的`/app`目录进行绑定。然后，我们为我们想要排除的内容挂载一个卷，在本例中，是子目录`.`** 的`/node_modules`

如果我们要保存数据，我们可以使用一个命名卷:

```java
volumes:
  - .:/app
  - my-vol:/app/node_modules/

volumes:
  my-vol:
    driver: local
```

最后，我们可以尝试修改容器中`/node_modules`目录的内容。在这种情况下，对我们的主机不会有任何影响。

## 5.结论

在本文中，我们学习了如何使用 Docker 排除文件或文件夹等资源。我们查看了从映像构建到运行容器的例子。对于容器，我们看到了使用 Docker Compose 的例子以及使用卷来排除子文件夹的可能性。

一如既往，我们可以在 GitHub 上找到工作代码示例[。](https://web.archive.org/web/20221222154222/https://github.com/eugenp/tutorials/tree/master/docker-modules/docker-compose/)