# 使用配置文件在 Docker 中启动 Spring Boot 应用程序

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-docker-start-with-profile>

## 1.介绍

我们都知道 Docker 有多流行，对 Java 开发人员来说，将他们的 Spring Boot 应用程序容器化是多么时髦。然而，对于一些开发人员来说，我们如何在 dockerized Spring Boot 应用程序中设置概要文件可能是一个问题。

在本教程中，我们将解释如何在 Docker 容器中启动带有概要文件的 Spring Boot 应用程序。

## 2.基本`Dockerfile`

通常，[要对一个 Spring Boot 应用程序](/web/20221026040453/https://www.baeldung.com/dockerizing-spring-boot-application)进行 dockerize，我们只需提供一个`Dockerfile`。

让我们来看看我们的 Spring Boot 应用程序的一个最小的`Dockerfile`:

```
FROM openjdk:11
COPY target/*.jar app.jar
ENTRYPOINT ["java", "-jar", "/app.jar"]
```

当然，我们可以通过`docker build`建立我们的码头工人形象:

```
docker build --tag=docker-with-spring-profile:latest .
```

因此，我们可以从图像`docker-with-spring-profile`运行我们的应用程序:

```
docker run docker-with-spring-profile:latest
```

正如我们注意到的，我们的 Spring Boot 应用程序从`“default”`概要文件开始:

```
2022-04-22 22:34:25.268 INFO 1 --- [main] c.b.docker.spring.DemoApplication: Starting DemoApplication using Java 11.0.14.1 on ea8851bea75f with PID 1 (/app.jar started by root in /)
2022-04-22 22:34:25.270 INFO 1 --- [main] c.b.docker.spring.DemoApplication: No active profile set, falling back to 1 default profile: "default"
//... 
```

## 3.在`Dockerfile`中设置轮廓

为我们的 dockerized 应用程序设置概要文件的一种方法是使用 Spring Boot 的命令行参数`“-Dspring.profiles.active”`。

因此，为了将概要文件设置为`“test”`，我们在我们的`Dockerfile's ENTRYPOINT`行中添加了一个新的参数`“-Dspring.profiles.active=test”,`:

```
//...
ENTRYPOINT ["java", "-Dspring.profiles.active=test", "-jar", "/app.jar"]
```

要查看配置文件的变化，让我们用同样的命令再次运行我们的容器:

```
docker run docker-with-spring-profile:latest
```

相应地，我们可以看到配置文件`“test”`被我们的应用程序成功选取:

```
2022-04-22 22:39:33.210 INFO 1 --- [main] c.b.docker.spring.DemoApplication: Starting DemoApplication using Java 11.0.14.1 on 227974fa84b2 with PID 1 (/app.jar started by root in /)
2022-04-22 22:39:33.212 INFO 1 --- [main] c.b.docker.spring.DemoApplication: The following 1 profile is active: "test"
//... 
```

## 4.使用环境变量设置配置文件

有时，在我们的`Dockerfile`中使用硬编码的概要文件并不方便。如果我们需要不止一个概要文件，那么在运行容器时选择其中一个可能会很麻烦。

尽管如此，还有更好的选择。**在启动期间，Spring Boot 寻找一个特殊的环境变量`SPRING_PROFILES_ACTIVE`。**

因此，我们实际上可以利用`docker run`命令在启动时设置 Spring 配置文件:

```
docker run -e "SPRING_PROFILES_ACTIVE=test" docker-with-spring-profile:latest
```

此外，根据我们的用例，我们可以通过逗号分隔的字符串一次设置多个概要文件:

```
docker run -e "SPRING_PROFILES_ACTIVE=test1,test2,test3" docker-with-spring-profile:latest
```

但是，我们要注意，Spring Boot 的房产之间有一个[特定的顺序](https://web.archive.org/web/20221026040453/https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.external-config)。**命令行参数优先于环境变量。**由于这个原因，为了让`SPRING_PROFILES_ACTIVE`工作，我们需要恢复我们的`Dockerfile`。

因此，我们从我们的`Dockerfile's ENTRYPOINT`行中删除了`“-Dspring.profiles.active=test”`参数:

```
//...
ENTRYPOINT ["java", "-jar", "/app.jar"]
```

最后，我们可以看到我们通过`SPRING_PROFILES_ACTIVE`设置的配置文件被考虑在内:

```
2022-04-22 22:50:28.924 INFO 1 --- [main] c.b.docker.spring.DemoApplication: Starting DemoApplication using Java 11.0.14.1 on 18eacb6362f8 with PID 1 (/app.jar started by root in /)
2022-04-22T22:50:28.926562249Z 2022-04-22 22:50:28.926 INFO 1 --- [main] c.b.docker.spring.DemoApplication: The following 3 profiles are active: "test1", "test2", "test3"
//.. 
```

## 5.在 Docker 撰写文件中设置配置文件

作为替代方法，**环境变量也可以在`[docker-compose](/web/20221026040453/https://www.baeldung.com/ops/docker-compose)`文件**中提供。

此外，为了更好地利用我们的`docker run`操作，我们可以为每个概要文件创建一个`docker-compose`文件。

让我们为`“test”`概要文件创建一个`docker-compose-test.yml`文件:

```
version: "3.5"
services:
  docker-with-spring-profile:
    image: docker-with-spring-profile:latest
    environment:
      - "SPRING_PROFILES_ACTIVE=test" 
```

类似地，我们为`“prod”`概要文件创建另一个文件`docker-compose-prod.yml`——唯一的不同是第二个文件中的概要文件`“prod”`:

```
//...
environment:
  - "SPRING_PROFILES_ACTIVE=prod" 
```

因此，我们可以通过两个不同的`docker-compose`文件运行我们的容器:

```
# for the profile 'test'
docker-compose -f docker-compose-test.yml up

# for the profile 'prod'
docker-compose -f docker-compose-prod.yml up
```

## 6.结论

在本教程中，我们描述了在 Docker 化的 Spring Boot 应用程序中设置配置文件的不同方法，还展示了一些使用 Docker 和 Docker Compose 的例子。

像往常一样，本教程中显示的所有代码示例都可以从 GitHub 上的[处获得。](https://web.archive.org/web/20221026040453/https://github.com/eugenp/tutorials/tree/master/docker-modules/docker-spring-boot)