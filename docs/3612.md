# 春天 YAML vs 房产

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-yaml-vs-properties>

## 1.介绍

**YAML 是一种在配置文件**中使用的友好符号。为什么我们更喜欢这种数据序列化而不是 Spring Boot[的属性文件呢？除了可读性和减少重复，YAML 是编写部署配置代码的完美语言。](/web/20220628142642/https://www.baeldung.com/spring-yaml)

同样，使用 Spring [DevOps](/web/20220628142642/https://www.baeldung.com/devops-overview) 的 YAML 有助于按照 [12 因子认证器](https://web.archive.org/web/20220628142642/https://12factor.net/config)的建议在环境中存储配置变量。

在本教程中，我们将比较 Spring YAML 和属性文件，以检查使用其中一个的主要优势。但是请记住，选择 YAML 而不是属性文件配置有时是个人喜好的决定。

## 2.YAML 符号

YAML 代表“ **YAML 不是标记语言**”的递归首字母缩写词。它具有以下特点:

*   更加清晰和人性化
*   非常适合分层配置数据
*   它支持增强功能，如映射、列表和标量类型

这些功能使 YAML 成为 Spring 配置文件的完美伴侣。对于那些从 YAML 开始的人，这里有一个警告:由于它的[缩进](https://web.archive.org/web/20220628142642/https://yaml.org/spec/1.2/spec.html#id2777534)规则，开始时编写它可能会有点乏味。

让我们看看它是如何工作的！

## 3.弹簧 YAML 配置

正如前面提到的，YAML 是一种非常好的配置文件数据格式。它的可读性更好，并且提供了比属性文件更强的功能。因此，在属性文件配置中推荐这种符号是有意义的。此外，从 1.2 版本开始，YAML 是 JSON 的超集。

此外，在 Spring 中，放置在工件外部的[配置文件会覆盖打包的 jar](/web/20220628142642/https://www.baeldung.com/spring-yaml#yaml-property-overriding) 中的配置文件。Spring 配置的另一个有趣的特性是可以在运行时分配环境变量。这对于 DevOps 部署极其重要。

[Spring profiles](/web/20220628142642/https://www.baeldung.com/spring-profiles) 允许分离环境并对它们应用不同的属性。YAML 增加了在同一个文件中包含几个配置文件的可能性。

注意:Spring Boot 2.4.0 的属性文件也支持此功能。

在我们的例子中，出于部署的目的，我们将有三个:测试、开发和生产:

```
spring:
  profiles:
    active:
    - test

---

spring:
  config:
    activate:
      on-profile: test
name: test-YAML
environment: testing
servers:
  - www.abc.test.com
  - www.xyz.test.com

---

spring:
  config:
    activate:
      on-profile: prod
name: prod-YAML
environment: production
servers:
  - www.abc.com
  - www.xyz.com

---

spring:
  config:
    activate:
      on-profile: dev
name: ${DEV_NAME:dev-YAML}
environment: development
servers:
  - www.abc.dev.com
  - www.xyz.dev.com
```

注意:如果我们使用 2.4.0 之前的 Spring Boot 版本，我们应该使用`spring.profiles` 属性，而不是这里使用的`spring.config.activate.on-profile `。

现在让我们检查默认分配测试环境的`spring.profiles.active`属性。我们可以使用不同的概要文件重新部署工件，而无需重新构建源代码。

Spring 中另一个有趣的特性是您可以通过环境变量启用概要文件:

```
export SPRING_PROFILES_ACTIVE=dev
```

我们将在测试部分看到这个环境变量的相关性。最后，我们可以配置 YAML 属性，直接从环境中分配值:

```
name: ${DEV_NAME:dev-YAML}
```

我们可以看到，如果没有配置环境变量，将使用默认值`dev-YAML` 。

## 4.减少重复和可读性

**YAML 的分层结构提供了减少配置属性文件**上层的方法。让我们通过一个例子来看看它们的区别:

```
component:
  idm:
    url: myurl
    user: user
    password: password
    description: >
      this should be a long 
      description
  service:
    url: myurlservice
    token: token
    description: >
      this should be another long 
      description
```

使用属性文件，相同的配置会变得多余:

```
component.idm.url=myurl
component.idm.user=user
component.idm.password=password
component.idm.description=this should be a long \
                          description
component.service.url=myurlservice
component.service.token=token
component.service.description=this should be another long \ 
                              description
```

**YAML 的等级性质极大地增强了可读性**。这不仅是一个避免重复的问题，而且缩进，用得好，完美地描述了配置是什么，是什么。使用 YAML，就像在属性文件中使用反斜杠\，可以使用`>`字符将内容分成多行。

## 5.列表和地图

**我们可以使用 [YAML](/web/20220628142642/https://www.baeldung.com/spring-yaml) 和[属性文件](/web/20220628142642/https://www.baeldung.com/configuration-properties-in-spring-boot)T5 来配置列表和地图。**

有两种方法分配值并将它们存储在列表中:

```
servers:
  - www.abc.test.com
  - www.xyz.test.com

external: [www.abc.test.com, www.xyz.test.com]
```

两个例子提供了相同的结果。使用属性文件的等效配置将更加难以阅读:

```
servers[0]=www.abc.test.com
servers[1]=www.xyz.test.com

external=www.abc.test.com, www.xyz.test.com
```

同样，YAML 版本更加易读和清晰。

同样，我们可以配置地图:

```
map:
  firstkey: key1
  secondkey: key2
```

## 6.测试

现在，让我们检查一下是否一切都按预期运行。如果我们检查应用程序的日志，我们可以看到默认选择的环境正在测试:

```
2020-06-11 13:58:28.846  INFO 10720 --- [main] com.baeldung.yaml.MyApplication: ...
using environment:testing
name:test-YAML
servers:[www.abc.test.com, www.xyz.test.com]
external:[www.abc.test.com, www.xyz.test.com]
map:{firstkey=key1, secondkey=key2}
Idm:
   Url: myurl
   User: user
   Password: password
   Description: this should be a long description

Service:
   Url: myurlservice
   Token: token
   Description: this should be another long description
```

我们可以通过在环境中配置`DEV_NAME`来覆盖该名称:

```
export DEV_NAME=new-dev-YAML
```

我们可以看到，使用 dev 配置文件执行应用程序时，环境的名称发生了变化:

```
2020-06-11 17:00:45.459  INFO 19636 --- [main] com.baeldung.yaml.MyApplication: ...
using environment:development
name:new-dev-YAML
servers:[www.abc.dev.com, www.xyz.dev.com]
```

让我们使用`SPRING_PROFILES_ACTIVE=prod`运行生产环境:

```
export SPRING_PROFILES_ACTIVE=prod

2020-06-11 17:03:33.074  INFO 20716 --- [main] ...
using environment:production
name:prod-YAML
servers:[www.abc.com, www.xyz.com]
```

## 7.结论

在本教程中，我们描述了与属性文件相比，使用 YAML 配置的复杂性。

我们展示了 **YAML 提供了人性化的能力，它减少了重复，比它的属性文件变体**更简洁。

和往常一样，代码可以在 GitHub 的[上获得。](https://web.archive.org/web/20220628142642/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-properties)