# Gatling vs JMeter vs Grinder:比较负载测试工具

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/gatling-jmeter-grinder-comparison>

## 1.介绍

为工作选择合适的工具可能会令人望而生畏。在本教程中，我们将通过比较三个 web 应用程序负载测试工具——Apache JMeter、Gatling 和 Grinder——和一个简单的 REST API 来简化这个过程。

## 2.负载测试工具

首先，让我们快速回顾一下每一项的背景。

### 2.1.格林机关枪

[Gatling](https://web.archive.org/web/20220628145036/https://gatling.io/) 是一个在 Scala 中创建测试脚本的负载测试工具。**加特林的记录器生成 Scala 测试脚本，这是加特林的一个关键特性。**查看我们的[加特林简介](/web/20220628145036/https://www.baeldung.com/introduction-to-gatling)教程了解更多信息。

### 2.2.JMeter

JMeter 是 Apache 的一个负载测试工具。它提供了一个很好的 GUI，我们可以使用它进行配置。**一个被称为逻辑控制器的独特功能为在 GUI 中设置测试提供了极大的灵活性。**

访问我们的[JMeter 简介](/web/20220628145036/https://www.baeldung.com/jmeter)教程，获取截图和更多解释。

### 2.3.研磨机

我们的最后一个工具，Grinder 提供了一个比其他两个更基于编程的脚本引擎，并且使用了 Jython。然而，Grinder 3 确实具有记录脚本的功能。

Grinder 与其他两个工具的不同之处还在于它支持控制台和代理进程。此功能为代理进程提供了能力，以便负载测试可以跨多个服务器扩展。它被特别宣传为一个为开发人员构建的负载测试工具，用于发现死锁和变慢。

## 3.测试用例设置

接下来，对于我们的测试，我们需要一个 API。我们的 API 功能包括:

*   添加/更新奖励记录
*   查看一个/所有奖励记录
*   将交易链接到客户奖励记录
*   查看客户奖励记录的交易

**我们的场景:**

一家商店正在全国范围内开展促销活动，新老客户需要客户奖励帐户来节省开支。奖励 API 通过客户 id `.`检查客户奖励账户，如果不存在奖励账户，添加它，然后链接到交易。

之后，我们查询交易。

### 3.1。我们的 REST API

让我们通过查看一些方法存根来快速了解 API:

```java
@PostMapping(path="/rewards/add")
public @ResponseBody RewardsAccount addRewardsAcount(@RequestBody RewardsAccount body)

@GetMapping(path="/rewards/find/{customerId}")
public @ResponseBody Optional<RewardsAccount> findCustomer(@PathVariable Integer customerId)

@PostMapping(path="/transactions/add")
public @ResponseBody Transaction addTransaction(@RequestBody Transaction transaction)

@GetMapping(path="/transactions/findAll/{rewardId}")
public @ResponseBody Iterable<Transaction> findTransactions(@PathVariable Integer rewardId) 
```

注意一些关系，比如通过奖励 id 查询交易和通过客户 id `. `获取奖励账户。这些关系为我们的测试场景创建强制了一些逻辑和一些响应解析。

测试中的应用程序还使用 H2 内存数据库来实现持久性。

幸运的是，我们的工具都处理得相当好，有些比其他的好。

### 3.2.我们的测试计划

接下来，我们需要测试脚本。

为了进行公平的比较，我们将对每个工具执行相同的自动化步骤:

1.  生成随机的客户帐户 id
2.  过账交易
3.  解析随机客户 id 和交易 id 的响应
4.  使用客户 id 查询客户奖励帐户 id
5.  解析奖励账户 id 的响应
6.  如果没有奖励帐户 id 存在，然后添加一个帖子
7.  使用交易 id，用更新的奖励 id 过帐相同的初始交易
8.  通过奖励账户 id 查询所有交易

让我们仔细看看每个工具的第 4 步。此外，请务必查看所有三个已完成脚本的示例[。](https://web.archive.org/web/20220628145036/https://github.com/eugenp/tutorials/tree/master/testing-modules/load-testing-comparison/src/main/resources/scripts)

### 3.3.格林机关枪

对于 Gatling 来说，熟悉 Scala 对开发人员来说是一个福音，因为 Gatling API 是健壮的，包含了很多特性。

加特林的 API 采用了一种构建 DSL 的方法，正如我们在它的步骤 4 中看到的:

```java
.exec(http("get_reward")
  .get("/rewards/find/${custId}")
  .check(jsonPath("$.id").saveAs("rwdId"))) 
```

特别值得注意的是，当我们需要读取和验证 HTTP 响应时，Gatling 支持 JSON Path。在这里，我们将拾取奖励 id，并将其保存到加特林的内部状态。

另外，加特林的表达式语言使得动态请求体变得更加容易

```java
.body(StringBody(
  """{ 
    "customerRewardsId":"${rwdId}",
    "customerId":"${custId}",
    "transactionDate":"${txtDate}" 
  }""")).asJson) 
```

最后，我们的比较配置。将 1000 次运行设置为整个场景的重复，`atOnceUsers `方法设置线程/用户:

```java
val scn = scenario("RewardsScenario")
  .repeat(1000) {
  ...
  }
  setUp(
    scn.inject(atOnceUsers(100))
  ).protocols(httpProtocol)
```

整个 Scala 脚本可以在我们的 Github repo 上看到。

### 3.4.JMeter

**JMeter 在 GUI 配置后生成一个 XML 文件。**该文件包含 JMeter 特定对象，这些对象具有已设置的属性及其值，例如:

```java
<HTTPSamplerProxy guiclass="HttpTestSampleGui" testclass="HTTPSamplerProxy" testname="Add Transaction" enabled="true">
```

```java
<JSONPostProcessor guiclass="JSONPostProcessorGui" testclass="JSONPostProcessor" testname="Transaction Id Extractor" enabled="true">
```

检查一下`testname`属性，当我们识别出它们与上面的逻辑步骤相匹配时，可以给它们贴上标签。添加孩子、变量和依赖步骤的能力为 JMeter 提供了脚本提供的灵活性。此外，我们甚至为变量设置了范围！

我们在 JMeter 中对运行和用户的配置使用了`ThreadGroups`:

```java
<stringProp name="ThreadGroup.num_threads">100</stringProp>
```

[查看整个`jmx`文件作为参考](https://web.archive.org/web/20220628145036/https://github.com/eugenp/tutorials/tree/master/testing-modules/load-testing-comparison/src/main/resources/scripts)。虽然可能，但是用 XML 作为`.jmx`文件编写测试对于全功能 GUI 来说没有意义。

### 3.5.研磨机

没有 Scala 和 GUI 的函数式编程，我们为 Grinder 编写的 Jython 脚本看起来非常简单。添加一些系统 Java 类，我们的代码就少了很多。

```java
customerId = str(random.nextInt());
result = request1.POST("http://localhost:8080/transactions/add",
  "{"'"customerRewardsId"'":null,"'"customerId"'":"+ customerId + ","'"transactionDate"'":null}")
txnId = parseJsonString(result.getText(), "id")
```

然而，更少的测试设置代码行被更多的字符串维护代码(比如解析 JSON 字符串)的需求所平衡。此外， [HTTPRequest](https://web.archive.org/web/20220628145036/http://grinder.sourceforge.net/g3/script-javadoc/net/grinder/plugin/http/HTTPRequest.html) API 在功能上也很单薄。

使用 Grinder，我们在外部属性文件中定义线程、进程和运行值:

```java
grinder.threads = 100
grinder.processes = 1
grinder.runs = 1000
```

我们为 Grinder 编写的完整 Jython 脚本将看起来像是这个。

## 4.试运转

### 4.1.测试执行

这三个工具都推荐使用命令行进行大负载测试。

为了运行测试，我们将使用加特林[开源](https://web.archive.org/web/20220628145036/https://gatling.io/open-source/)版本 3.4.0 作为独立工具， [JMeter 5.3](https://web.archive.org/web/20220628145036/https://jmeter.apache.org/download_jmeter.cgi) 和 Grinder [版本 3](https://web.archive.org/web/20220628145036/https://sourceforge.net/projects/grinder/) 。

加特林只要求我们设置好`JAVA_HOME`和`GATLING_HOME`。为了执行加特林，我们使用:

```java
./gatling.sh
```

在 GATLING_HOME/bin 目录下。

当启动 GUI 进行配置时，JMeter 需要一个参数来禁用测试的 GUI:

```java
./jmeter.sh -n -t TestPlan.jmx -l log.jtl
```

和加特林一样，Grinder 要求我们设置`JAVA_HOME`和`GRINDERPATH`。然而，它还需要几个属性:

```java
export CLASSPATH=/home/lore/Documents/grinder-3/lib/grinder.jar:$CLASSPATH
export GRINDERPROPERTIES=/home/lore/Documents/grinder-3/examples/grinder.properties
```

如上所述，我们为线程、运行、进程和控制台主机等附加配置提供了一个`grinder.properties`文件。

最后，我们通过以下方式引导控制台和代理:

```java
java -classpath $CLASSPATH net.grinder.Console
```

```java
java -classpath $CLASSPATH net.grinder.Grinder $GRINDERPROPERTIES
```

### 4.2.试验结果

每个测试运行 1000 次，有 100 个用户/线程。让我们来解开一些亮点:

|  | **成功的请求** | **错误** | **总测试时间** | **平均响应时间(毫秒)** | **平均吞吐量** |
| **加特林** | 500000 个请求 | Zero | 218s | forty-two | 2283 请求/秒 |
| **JMeter** | 499997 次请求 | Zero | 237s | Forty-six | 2101 请求/秒 |
| **研磨机** | 499997 次请求 | Zero | 221s | Forty-three | 2280 请求/秒 |

**结果显示，这 3 种工具的速度相似，根据平均吞吐量，Gatling 略微超过另外 2 种。**

每个工具还在更友好的用户界面中提供附加信息。

**Gatling 将在运行结束时生成一份 HTML 报告，其中包含针对整个运行以及每个请求的多个图表和统计数据。**下面是测试结果报告的一个片段:

[![gatling result new 1](img/49a45dbd2a13285ff09f5c8eedf2dc66.png)](/web/20220628145036/https://www.baeldung.com/wp-content/uploads/2018/11/gatling-result-new-1.png)

**使用 JMeter 时，我们可以在测试运行后打开 GUI，并根据保存结果的日志文件**生成一个 HTML 报告:

[![2020/10/jmeter-result-new.png](img/ba5ebaef1ea53e286b43978ad60b6bab.png)](/web/20220628145036/https://www.baeldung.com/wp-content/uploads/2020/10/jmeter-result-new.png)

JMeter HTML 报告还包含每个请求的统计数据的细分。

**最后，Grinder 控制台记录每个代理的统计数据并运行:**

[![grinder result new 1](img/b36596d266316857297f7d0d4538dbd6.png)](/web/20220628145036/https://www.baeldung.com/wp-content/uploads/2020/10/grinder-result-new-1.png)

虽然 Grinder 是高速的，但它的代价是额外的开发时间和输出数据多样性的减少。

## 5.摘要

现在是时候全面了解一下每个负载测试工具了。

|  | **加特林** | **JMeter** | **研磨机** |
| 项目和社区 | nine | nine | six |
| 表演 | nine | eight | nine |
| 可脚本化/API | seven | nine | eight |
| 用户界面 | nine | eight | six |
| 报告 | nine | seven | six |
| 综合 | seven | nine | seven |
| **总结** | **8.3** | **8.3** | **7** |

**加特林:**

*   可靠、完美的负载测试工具，通过 Scala 脚本输出漂亮的报告
*   产品的开源和企业支持级别

**JMeter:**

*   用于测试脚本开发的健壮 API(通过 GUI ),无需编码
*   Apache Foundation 支持以及与 Maven 的高度集成

**研磨机:**

*   面向使用 Jython 的开发人员的快速性能负载测试工具
*   跨服务器的可伸缩性为大型测试提供了更大的潜力

简单地说，如果速度和可伸缩性是一种需要，那么使用 Grinder。

如果漂亮的交互式图形有助于显示性能的提高，以支持改变，那么就使用加特林。

JMeter 是用于复杂业务逻辑的工具，或者是具有许多消息类型的集成层。作为 Apache 软件基础的一部分，JMeter 提供了一个成熟的产品和一个大型社区。

## 6.结论

总之，我们看到这些工具在一些领域具有相似的功能，而在另一些领域则表现出色。用于正确工作的正确工具是在软件开发中起作用的通俗智慧。

最后，API 和脚本可以在 Github 上找到[。](https://web.archive.org/web/20220628145036/https://github.com/eugenp/tutorials/tree/master/testing-modules/load-testing-comparison)