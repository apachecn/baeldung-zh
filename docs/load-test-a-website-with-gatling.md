# 用加特林进行负载测试

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/load-test-a-website-with-gatling>

## 1。概述

在[之前的教程](/web/20220120181206/https://www.baeldung.com/introduction-to-gatling)中，我们已经看到了如何使用 Gatling 来负载测试一个定制的 web 应用程序。

在本文中，我们将使用 Gatling 压力工具来衡量该网站的登台环境的性能。

## 2。测试场景

让我们首先设置我们的主要使用场景——一个接近可能浏览网站的典型用户的场景:

1.  转到主页
2.  从主页打开文章
3.  转到指南/休息
4.  转到休息类别
5.  转到完整存档
6.  从存档中打开一篇文章

## 3。记录场景

现在，我们将使用加特林录音机录制我们的场景，如下所示:

```java
$GATLING_HOME/bin/recorder.sh
```

对于 Windows 用户:

```java
%GATLING_HOME%\bin\recorder.bat
```

注意:`GATLING_HOME`是你的加特林安装目录。

加特林记录仪有**两种模式:HTTP 代理和 HAR 转换器。**

我们在[之前的教程](/web/20220120181206/https://www.baeldung.com/introduction-to-gatling)中详细讨论了 HTTP 代理模式——现在让我们看看 HAR 转换器选项。

### 3.1。HAR 转换器

HAR 是 HTTP Archive 的缩写——这是一种基本上**记录关于浏览会话**的完整信息的格式。

我们可以从浏览器中获得 HAR 文件，然后使用加特林录音机将其转换成模拟。

我们将在 Chrome 开发工具的帮助下创建我们的 HAR 文件:

*   菜单->更多工具->开发者工具
*   转到`Network`选项卡
*   确保勾选了`Preserve log`
*   浏览完网站后，右键单击要导出的请求
*   然后，选择全部复制为 HAR
*   将它们粘贴到一个文件中，然后从加特林录音机中导入

当您完成调整转管录音机到您的喜好，点击开始。

请注意，输出文件夹默认为`GATLING_HOME/user-files-simulations`

## 4。模拟

生成的模拟文件同样是用 Scala 编写的。总体还可以，但不是超级可读，所以我们会做一些调整来清理。这是我们最后的模拟:

```java
class RestSimulation extends Simulation {

    val httpProtocol = http.baseURL("http://staging.baeldung.com")

    val scn = scenario("RestSimulation")
      .exec(http("home").get("/"))
      .pause(23)
      .exec(http("article_1").get("/spring-rest-api-metrics"))
      .pause(39)
      .exec(http("rest_series").get("/rest-with-spring-series"))
      .pause(60)
      .exec(http("rest_category").get("/category/rest/"))
      .pause(26)
      .exec(http("archive").get("/full_archive"))
      .pause(70)
      .exec(http("article_2").get("/spring-data-rest-intro"))

    setUp(scn.inject(atOnceUsers(1))).protocols(httpProtocol)
}
```

这里需要注意的一点是，完整的模拟文件要大得多；这里，**为了简单起见，我们没有包括静态资源**。

## 5。运行负载测试

现在，我们可以运行我们的模拟，如下所示:

```java
$GATLING_HOME/bin/gatling.sh
```

对于 Windows 用户:

```java
%GATLING_HOME%\bin\gatling.bat
```

加特林工具将扫描`GATLING_HOME/user-files-simulations` 并列出所有找到的模拟供我们选择。

运行模拟后，结果如下:

对于一个用户:

```java
> request count                                304 (OK=304    KO=0)
> min response time                             75 (OK=75     KO=-)
> max response time                          13745 (OK=13745  KO=-)
> mean response time                          1102 (OK=1102   KO=-)
> std deviation                               1728 (OK=1728   KO=-)
> response time 50th percentile                660 (OK=660    KO=-)
> response time 75th percentile               1006 (OK=1006   KO=-)
> mean requests/sec                           0.53 (OK=0.53   KO=-)
---- Response Time Distribution ------------------------------------
> t < 800 ms                                           183 ( 60%)
> 800 ms < t < 1200 ms                                  54 ( 18%)
> t > 1200 ms                                           67 ( 22%)
> failed                                                 0 (  0%)
```

对于 5 个并发用户:

```java
> request count                               1520 (OK=1520   KO=0)
> min response time                             70 (OK=70     KO=-)
> max response time                          30289 (OK=30289  KO=-)
> mean response time                          1248 (OK=1248   KO=-)
> std deviation                               2079 (OK=2079   KO=-)
> response time 50th percentile                504 (OK=504    KO=-)
> response time 75th percentile               1440 (OK=1440   KO=-)
> mean requests/sec                          2.411 (OK=2.411  KO=-)
---- Response Time Distribution ------------------------------------
> t < 800 ms                                           943 ( 62%)
> 800 ms < t < 1200 ms                                 138 (  9%)
> t > 1200 ms                                          439 ( 29%)
> failed                                                 0 (  0%)
```

对于 10 个并发用户:

```java
> request count                               3058 (OK=3018   KO=40)
> min response time                              0 (OK=69     KO=0)
> max response time                          44916 (OK=44916  KO=30094)
> mean response time                          2193 (OK=2063   KO=11996)
> std deviation                               4185 (OK=3953   KO=7888)
> response time 50th percentile                506 (OK=494    KO=13670)
> response time 75th percentile               2035 (OK=1976   KO=15835)
> mean requests/sec                          3.208 (OK=3.166  KO=0.042)
---- Response Time Distribution ----------------------------------------
> t < 800 ms                                          1752 ( 57%)
> 800 ms < t < 1200 ms                                 220 (  7%)
> t > 1200 ms                                         1046 ( 34%)
> failed                                                40 (  1%)
```

请注意，在测试 10 个并发用户时，一些请求失败了——这仅仅是因为暂存环境无法处理这种负载。

## 6。结论

在这篇简短的文章中，我们探讨了在加特林中记录测试场景的 HAR 选项，并对 baeldung.com 做了一个简单的初始测试。