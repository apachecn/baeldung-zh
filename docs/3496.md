# 将环境变量传递给 Docker 容器

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/ops/docker-container-environment-variables>

## 1.概观

将我们的服务从它们的配置中分离出来通常是一个好主意。**对于一个[十二因素 app](/web/20221006144700/https://www.baeldung.com/spring-boot-12-factor) ，我们应该在环境**中存储配置。

当然，这意味着我们需要一种方法将配置注入到我们的服务中。

在本教程中，我们将通过将环境变量传递给一个 [Docker](/web/20221006144700/https://www.baeldung.com/tag/docker/) 容器来实现这一点。

## 2\. Using `–env` , `-e`

在本教程中，我们将使用一个名为 Alpine 的小型(5MB) Linux 映像。让我们从本地提取图像开始:

```
docker pull alpine:3
```

当我们启动 Docker 容器**时，我们可以使用参数`–env`(或其缩写`-e`)将环境变量作为键值对直接传递到命令行**。

例如，让我们执行以下命令:

```
$ docker run --env VARIABLE1=foobar alpine:3 env 
```

简而言之，我们将环境变量反射回控制台:

```
VARIABLE1=foobar
```

可以看到，Docker 容器正确地解释了变量`VARIABLE1`。

同样，**如果变量已经存在于本地环境**中，我们可以省略命令行中的值。

例如，让我们定义一个本地环境变量:

```
$ export VARIABLE2=foobar2
```

然后，让我们指定不带值的环境变量:

```
docker run --env VARIABLE2 alpine:3 env
```

我们可以看到 Docker 仍然获得了价值，这一次是从周围环境中获得的:

```
VARIABLE2=foobar2
```

## 3.使用`–env-file`

当变量数量较少时，上述解决方案就足够了。然而，一旦我们有了更多的变量，它很快就会变得繁琐和容易出错。

**另一个解决方案是使用文本文件存储我们的变量**，使用标准的`key=value`格式。

让我们在一个名为`my-env.txt`的文件中定义几个变量:

```
$ echo VARIABLE1=foobar1 > my-env.txt
$ echo VARIABLE2=foobar2 >> my-env.txt
$ echo VARIABLE3=foobar3 >> my-env.txt
```

现在，让我们将这个文件注入到 Docker 容器中:

```
$ docker run --env-file my-env.txt alpine:3 env
```

最后，让我们看看输出:

```
VARIABLE1=foobar1
VARIABLE2=foobar2
VARIABLE3=foobar3
```

## 4.使用 Docker 撰写

Docker Compose 还提供了定义环境变量的工具。对于那些对这个特定主题感兴趣的人，请查看我们的 [Docker 撰写教程](/web/20221006144700/https://www.baeldung.com/docker-compose#managing-environment-variables)了解更多细节。

## 5.小心敏感的价值观

通常，其中一个变量是数据库或外部服务的密码。我们必须小心如何将这些变量注入 Docker 容器。

直接通过命令行传递这些值可能是最不安全的，因为在我们意想不到的地方泄露敏感值的风险更大，比如在我们的源代码控制系统或 OS 进程列表中。

在本地环境或文件中定义敏感值是更好的选择，因为这两种方式都可以防止未经授权的访问。

**然而，重要的是要认识到，任何有权访问 Docker 运行时的用户都可以`inspect`运行容器并发现秘密值**。

让我们检查一个运行中的容器:

```
docker inspect 6b6b033a3240
```

输出显示了环境变量:

```
"Config": {
    // ...
    "Env": [
       "VARIABLE1=foobar1",
       "VARIABLE2=foobar2",
       "VARIABLE3=foobar3",
    // ...
    ]
}
```

对于那些关注安全性的情况，重要的是要提到 Docker 提供了一种叫做 [Docker Secrets](https://web.archive.org/web/20221006144700/https://docs.docker.com/engine/swarm/secrets/) 的机制。容器服务，像那些由 [Kubernetes](/web/20221006144700/https://www.baeldung.com/kubernetes) ，AWS 或者 Azure 提供的，也提供类似的功能。

## 6.结论

在这个简短的教程中，我们研究了将环境变量注入 Docker 容器的几种不同的方法。

虽然每种方法都很有效，但我们的选择最终取决于各种参数，如安全性和可维护性。