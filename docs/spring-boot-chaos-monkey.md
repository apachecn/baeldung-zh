# 混沌猴简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-chaos-monkey>

## 1.介绍

在本教程中，我们将为 Spring Boot 谈论[混沌猴](https://web.archive.org/web/20221114100953/https://codecentric.github.io/chaos-monkey-spring-boot/)。

这个工具帮助我们**将[混沌工程](https://web.archive.org/web/20221114100953/https://principlesofchaos.org/)的一些原理引入到我们的 Spring Boot web 应用**中，方法是给我们的 REST 端点增加延迟，抛出错误，甚至终止一个应用。

## 2.设置

为了将 Chaos Monkey 添加到我们的应用程序中，我们的项目中需要一个单独的 [Maven 依赖项](https://web.archive.org/web/20221114100953/https://search.maven.org/classic/#search%7Cgav%7C1%7Ca%3A%22chaos-monkey-spring-boot%22):

```java
<dependency>
    <groupId>de.codecentric</groupId>
    <artifactId>chaos-monkey-spring-boot</artifactId>
    <version>2.0.0</version>
</dependency>
```

## 3.配置

一旦我们在项目中设置了依赖关系，我们就需要配置并启动我们的 chaos。

我们可以通过几种方式做到这一点:

*   在应用程序启动时，使用`chaos-monkey `弹簧轮廓(推荐)
*   使用`chaos.monkey.enabled=true `属性

通过使用`chaos-monkey` spring profile **启动应用程序，当我们的应用程序运行时，如果我们想要启用或禁用它**，我们不必停止和启动应用程序:

```java
java -jar your-app.jar --spring.profiles.active=chaos-monkey
```

另一个有用的属性是`management.endpoint.chaosmonkey.enabled. `,将该属性设置为 true 将为我们的混沌猴子启用管理端点:

```java
http://localhost:8080/chaosmonkey
```

从这个端点，我们可以看到我们的图书馆的地位。这里是端点的完整[列表及其描述，将有助于更改配置、启用或禁用 Chaos Monkey 和其他更细粒度的控制。](https://web.archive.org/web/20221114100953/https://github.com/codecentric/chaos-monkey-spring-boot/blob/main/chaos-monkey-docs/src/main/asciidoc/endpoints.adoc)

使用所有可用的[属性](https://web.archive.org/web/20221114100953/https://codecentric.github.io/chaos-monkey-spring-boot/latest/#_properties)，我们可以对我们生成的混沌中发生的事情进行更细粒度的控制。

## 4.它是如何工作的

混沌猴由守望者和突击者组成。观察器是 Spring Boot 组件。它利用 [Spring AOP](https://web.archive.org/web/20221114100953/https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#aop-api) 来查看公共方法何时在用以下 Spring 注释注释的类中执行:

*   成分
*   控制器
*   rest 控制器
*   服务
*   贮藏室ˌ仓库

根据我们的应用程序属性文件中的配置，**我们的公共方法将会受到或不会受到以下攻击:**

*   延迟攻击–给请求增加随机延迟
*   异常攻击——抛出随机运行时异常
*   应用杀手攻击——嗯，应用死亡

让我们来看看如何配置我们的观察器和攻击，以便更好地控制攻击。

## 5.看守人

默认情况下，Watcher 只对我们的`services`启用。这意味着我们的攻击将只针对用`@Service.`标注的类中的公共方法来执行

但是我们可以通过配置属性轻松地改变这一点:

```java
chaos.monkey.watcher.controller=false
chaos.monkey.watcher.restController=false
chaos.monkey.watcher.service=true
chaos.monkey.watcher.repository=false
chaos.monkey.watcher.component=false
```

请记住，一旦应用程序启动，**我们就不能使用我们前面谈到的 Spring Boot 管理端口**的混沌猴子来动态更改观察器。

## 6.攻击

攻击基本上是我们想要在应用程序中测试的场景。让我们来看看每种类型的攻击，看看它有什么作用，以及我们如何配置它。

### 6.1.潜伏攻击

这种类型的攻击增加了我们呼叫的延迟。通过这种方式，我们的应用程序响应较慢，我们可以监控它在数据库响应较慢时的表现。

我们可以使用应用程序的属性文件来配置和开启这种类型的攻击:

```java
chaos.monkey.assaults.latencyActive=true
chaos.monkey.assaults.latencyRangeStart=3000
chaos.monkey.assaults.latencyRangeEnd=15000
```

另一种配置和开关这种类型攻击的方法是通过 Chaos Monkey 的管理端点。

让我们打开延迟攻击，并添加一个介于 2 到 5 秒之间的延迟范围:

```java
curl -X POST http://localhost:8080/chaosmonkey/assaults \
-H 'Content-Type: application/json' \
-d \
'
{
	"latencyRangeStart": 2000,
	"latencyRangeEnd": 5000,
	"latencyActive": true,
	"exceptionsActive": false,
	"killApplicationActive": false
}'
```

### 6.2.例外攻击

这测试了我们的应用程序处理异常的能力。根据配置，一旦启用，它将抛出一个随机运行时异常。

我们可以使用类似于延迟攻击的 curl 调用来启用它:

```java
curl -X POST http://localhost:8080/chaosmonkey/assaults \
-H 'Content-Type: application/json' \
-d \
'
{
	"latencyActive": false,
	"exceptionsActive": true,
	"killApplicationActive": false
}'
```

### 6.3.杀手攻击

这一个，嗯，我们的应用会在某个随机点死亡。我们可以通过一个简单的 curl 调用来启用或禁用它，就像前面两种攻击类型一样:

```java
curl -X POST http://localhost:8080/chaosmonkey/assaults \
-H 'Content-Type: application/json' \
-d \
'
{
	"latencyActive": false,
	"exceptionsActive": false,
	"killApplicationActive": true
}'
```

## 7.结论

在本文中，**我们为 Spring Boot** 谈论了混沌猴。我们已经看到，它采用了[混沌工程](https://web.archive.org/web/20221114100953/https://principlesofchaos.org/)的一些原理，并使我们能够将它们应用到 [Spring Boot](https://web.archive.org/web/20221114100953/https://spring.io/projects/spring-boot) 的应用中。

和往常一样，例子的完整代码可以在 Github 上找到[。](https://web.archive.org/web/20221114100953/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-performance)