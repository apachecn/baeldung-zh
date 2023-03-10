# 弹簧靴 vs quartus

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-vs-quarkus>

## 1.概观

在本文中，我们将对两个著名的 Java 框架 Spring Boot 和 Quarkus 进行简单的比较。在它的结尾，我们将更好地理解它们的不同点、相似点和一些特殊性。

此外，我们将执行一些测试来衡量他们的表现，并观察他们的行为。

## 2.Spring Boot

Spring Boot 是一个基于 Java 的框架，专注于企业应用。它连接了所有的 Spring 项目，**通过提供许多生产就绪的集成来帮助提高开发人员的生产力**。

通过这样做，它减少了配置和样板文件的数量。此外，**得益于其优于配置方法**的约定，即基于运行时类路径中可用的依赖项自动注册默认配置，Spring Boot 大大缩短了许多 Java 应用程序的上市时间。

## 3.夸库斯

Quarkus 是另一个框架，它的方法与上面提到的 Spring 类似，但是有一个额外的承诺，即以更快的启动时间、更好的资源利用率和效率交付更小的工件。

它针对云、无服务器和容器化环境进行了优化。尽管关注点略有不同，但 Quarkus 也能与最流行的 Java 框架很好地集成。

## 4.比较

如上所述，这两个框架都可以很好地与其他项目和框架集成。但是，它们的内部实现和架构是不同的。例如，Spring Boot 提供了两种风格的 web 功能:[阻塞(servlet)](/web/20221027145810/https://www.baeldung.com/spring-mvc-tutorial)和[非阻塞(WebFlux)](/web/20221027145810/https://www.baeldung.com/spring-webflux) 。

另一方面， **Quarkus 也提供了两种方法，但与 Spring Boot 不同，它[允许我们同时使用阻塞和非阻塞策略](https://web.archive.org/web/20221027145810/https://developers.redhat.com/blog/2019/11/18/how-quarkus-brings-imperative-and-reactive-programming-together)** 。此外， **Quarkus 在其架构中嵌入了反应式方法**。

出于这个原因，**我们将使用两个完全反应式的应用程序来实现 Spring WebFlux 和 Quarkus 反应式功能，以便在我们的比较中有一个更精确的场景**。

此外，Quarkus 项目中最重要的特性之一是创建本机映像(二进制和特定于平台的可执行文件)的能力。因此，我们也将两种原生映像包括在比较中，但是对于 Spring 来说，原生映像支持仍然处于实验阶段。为此，我们需要 [GraalVM](https://web.archive.org/web/20221027145810/https://www.graalvm.org/) 。

### 4.1.测试应用

我们的应用程序将公开三个 API:一个允许用户创建邮政编码，另一个查找特定邮政编码的信息，最后，按城市查询邮政编码。这些 API 是使用 Spring Boot 和 Quarkus 实现的，完全使用了反应式方法，如前所述，还使用了 MySQL 数据库。

目标是有一个简单的示例应用程序，但是比 HelloWorld 应用程序稍微复杂一点。当然，这将影响我们的比较，因为数据库驱动程序和序列化框架等东西的实现将影响结果。然而，大多数应用程序也可能处理这些事情。

因此，我们的比较并不旨在成为哪个框架更好或更有性能的最终事实，而是一个分析这些特定实现的案例研究。

### 4.2.测试计划

为了测试这两个实现，我们将使用 [Wrk](https://web.archive.org/web/20221027145810/https://github.com/wg/wrk) 来执行测试，并使用它的度量报告来分析我们的发现。此外，我们将使用 [VisualVM](https://web.archive.org/web/20221027145810/https://visualvm.github.io/) 来监控测试执行期间应用程序的资源利用率。

测试将运行 7 分钟，调用所有 API，从预热阶段开始，增加连接数量，直到达到 100 个。使用此设置，Wrk 会产生大量负载:

[![](img/edc32deffc6be3cec8580ca8771c33ed.png)](/web/20221027145810/https://www.baeldung.com/wp-content/uploads/2021/10/test-planning-wrk.png)

所有测试都是在具有以下规格的机器上进行的:

[![](img/de67d609e8f9fc2c3ef0d6528a784040.png)](/web/20221027145810/https://www.baeldung.com/wp-content/uploads/2021/10/machine-specifications-1.png)

虽然由于缺乏与其他后台进程的隔离而不理想，但该测试仅旨在说明所提议的比较。如前所述，我们无意对这两种框架的性能进行广泛而详细的分析。

值得一提的另一点是，根据我们的机器规格，我们可能需要调整连接、线程等的数量。

### 4.3.了解我们的测试

确保我们正在测试正确的东西是至关重要的，因此要做到这一点，我们将使用 Docker 容器来部署我们的基础架构。这将允许我们控制应用程序和数据库的资源约束。目标是强调应用程序，现在是底层系统，我们的数据库。对于这个例子，仅仅限制可用 CPU 的数量就足够了，但是这可能会根据我们机器中可用的资源而改变。

为了限制可用的源，我们可以使用 [Docker 设置](/web/20221027145810/https://www.baeldung.com/ops/docker-memory-limit)、`[cpulimit](https://web.archive.org/web/20221027145810/https://manpages.ubuntu.com/manpages/trusty/man1/cpulimit.1.html)`命令或任何其他我们喜欢的工具。此外，我们可以使用`[docker stats](https://web.archive.org/web/20221027145810/https://docs.docker.com/engine/reference/commandline/stats/)`和`[top](/web/20221027145810/https://www.baeldung.com/linux/top-command)`命令来监控系统的资源。最后，关于内存，我们将测量堆的使用情况以及 [RSS](https://web.archive.org/web/20221027145810/https://en.wikipedia.org/wiki/Resident_set_size) ，为此我们使用`[ps](https://web.archive.org/web/20221027145810/https://man7.org/linux/man-pages/man1/ps.1.html)` ( `ps -o pid,rss,command -p <pid>`)命令。

## 5.调查的结果

这两个项目的开发者体验都很棒，但值得一提的是，Spring Boot 的文档和资料比我们在网上能找到的更好。Quarkus 正在这方面进行改进，并拥有大量有助于提高生产率的功能。然而，考虑到文档和堆栈溢出问题，它仍然落后。

就指标而言，我们有:

[![](img/84183074bee4fded08c0d145eaa3bb8f.png)](/web/20221027145810/https://www.baeldung.com/wp-content/uploads/2021/10/quarkus-metrics.png)

通过这个实验，我们可以观察到在 JVM 和本地版本中，Quarkus 的启动时间都比 Spring Boot 快。此外，在原生映像的情况下，Quarkus 的构建时间也要快得多。构建用了 91 秒(Quarkus)对 113 秒(Spring Boot)，JVM 构建用了 5.24 秒(Quarkus)对 1.75 秒(Spring Boot)，所以这一次指向 Spring。

关于工件大小，Spring Boot 和 Quarkus 生产的可运行工件在 JVM 版本方面是相似的，但是在本地工件方面，Quarkus 做得更好。

然而，关于其他指标，结论并不简单。所以，让我们更深入地看看其中的一些。

### 5.1.中央处理器

如果我们关注 CPU 的使用情况，我们会发现在预热阶段的开始阶段, JVM 版本消耗了更多的 CPU。**之后 CPU 使用率稳定**，消耗变得相对等于所有版本。

以下是 JVM 和本地版本中 Quarkus 的 CPU 消耗，顺序如下:

(Spring JVM)

[![](img/f9127c2b899ceffd974f645c97e6fd18.png)](/web/20221027145810/https://www.baeldung.com/wp-content/uploads/2021/10/spring-cpu.png)

(夸尔库斯 JVM)

[![](img/e3b3a3b7b608913f8d6da80b77c30849.png)](/web/20221027145810/https://www.baeldung.com/wp-content/uploads/2021/10/quarkus-cpu.png)

(春原生)

[![](img/16c6217d2350540e117c8067817da350.png)](/web/20221027145810/https://www.baeldung.com/wp-content/uploads/2021/10/spring-native-cpu.png)

(夸尔库斯土著)

[![](img/5409a49dd7dac848052ac66e32a49927.png)](/web/20221027145810/https://www.baeldung.com/wp-content/uploads/2021/10/quarkus-native-cpu.png)

夸库斯在这两种情况下都做得更好。然而，差别很小，也可以考虑打个平手。值得一提的另一点是，在图表中，我们看到了基于机器中可用 CPU 数量的消耗。不过，为了确保我们强调的是选项而不是系统的其他部分，我们将应用程序可用的内核数量限制为三个。

### 5.2.记忆

关于内存，就更复杂了。首先，**两个框架的 JVM 版本为堆保留了更多的内存，几乎是相同的内存量**。关于堆的使用，JVM 版本比本地版本消耗更多的内存，但是看一下这些对，在 JVM 版本中，Quarkus 消耗的内存似乎比 Spring 稍少。但是，同样的，差别非常小。

(Spring Boot JVM)

[![](img/34bba74e4870b269c7ad36b976b8a931.png)](/web/20221027145810/https://www.baeldung.com/wp-content/uploads/2021/10/spring-memory.png)

(夸尔库斯 JVM)

[![](img/ef398cf7b9863a398c7cb44db531853c.png)](/web/20221027145810/https://www.baeldung.com/wp-content/uploads/2021/10/quarkus-memory.png)

然后，看着原生图像，事情似乎发生了变化。**Spring 原生版本似乎更频繁地收集内存，并保持更低的内存占用量**。

(Spring Boot 本地人)

[![](img/1c41b28898829e4f8a981c43f3a5112c.png)](/web/20221027145810/https://www.baeldung.com/wp-content/uploads/2021/10/spring-native-memory.png)

(夸尔库斯土著)

[![](img/2feaba6933d4000d01d9531939b4285d.png)](/web/20221027145810/https://www.baeldung.com/wp-content/uploads/2021/10/quarkus-native-memory.png)

另一个重要的亮点是，在 RSS 内存测量方面，Quarkus 似乎在两个版本中都超过了 Spring。我们只在启动时添加了 RSS 比较，但是我们也可以在测试期间使用相同的命令。

然而，在这个比较中，我们只使用了默认参数。因此，没有对 GC、JVM 选项或任何其他参数进行任何更改。不同的应用程序可能需要不同的设置，我们在现实环境中使用它们时应该记住这一点。

### 5.3.响应时间

**最后，我们将使用一种不同的方法来考虑响应时间，因为许多可用的基准测试工具都存在一个叫做[协调遗漏](https://web.archive.org/web/20221027145810/https://www.slideshare.net/InfoQ/how-not-to-measure-latency-60111840)的问题。我们将使用 [hyperfoil](https://web.archive.org/web/20221027145810/https://hyperfoil.io/) ，一个旨在避免这个问题的工具**。在测试过程中，会创建许多请求，但是这样做的目的是不要给应用程序施加太大的压力，而只是为了度量它的响应时间。

尽管如此，测试结构与前一个非常相似。

(Spring Boot JVM)

[![](img/bbb3f2e816da0f50ef78159cd43bb4df.png)](/web/20221027145810/https://www.baeldung.com/wp-content/uploads/2021/10/spring-response-time.png)

(夸尔库斯 JVM)

[![](img/ff364c8abdaab7668a826bbe315c0b16.png)](/web/20221027145810/https://www.baeldung.com/wp-content/uploads/2021/10/quarkus-response-time.png)

吞吐量和响应时间虽然相关，但不是一回事，它们衡量的是不同的东西。Quarkus JVM 版本在压力下有很好的表现，在中等负载时也是如此。它似乎具有更高的吞吐量和略低的响应时间。

(Spring Boot 本地人)

[![](img/0c5404d25b20a7c29470b6ef538117c0.png)](/web/20221027145810/https://www.baeldung.com/wp-content/uploads/2021/10/spring-native-response-time.png)

(夸尔库斯土著)

[![](img/578c167d6e0c614247d5083ef2c37a9d.png)](/web/20221027145810/https://www.baeldung.com/wp-content/uploads/2021/10/quarkus-native-response-time.png)

再看原生版本，数字又变了。现在，Spring 的响应时间似乎稍微短了一点，但总体吞吐量却更高了。然而，纵观所有的数字，我们可以看到，差异太小，无法定义任何明确的赢家。

### 5.4.将这些点连接起来

总的来说，这两个框架都被证明是实现 Java 应用程序的很好的选择。

本机应用程序已经证明速度快，资源消耗低，是无服务器、寿命短的应用程序和低资源消耗至关重要的环境的绝佳选择。

另一方面，JVM 应用程序似乎有更多的开销，但随着时间的推移，它具有出色的稳定性和高吞吐量，非常适合健壮、长寿的应用程序。

**最后，关于性能，所有版本都具有强大的性能，至少在我们的例子中是这样。差别如此之小，以至于我们可以说它们具有相似的性能**。当然，我们可以说 JVM 版本在吞吐量方面更好地处理了繁重的负载，同时消耗了更多的资源，另一方面，本机版本消耗的资源更少。然而，根据不同的用例，这种差异甚至可能无关紧要。

最后，我必须指出，在 Spring 应用程序中，我们必须切换 DB 驱动程序，因为文档中推荐的一个驱动程序有一个[问题](https://web.archive.org/web/20221027145810/https://github.com/spring-projects/spring-data-r2dbc/issues/766)。相比之下，Quarkus 没有任何问题。

## 6.结论

本文比较了 Spring Boot 和 Quarkus 框架及其不同的部署模式，JVM 和 Native。我们还研究了这些应用程序的其他指标和方面。像往常一样，测试应用程序的代码和用于测试它们的脚本可以从 GitHub 上的[处获得。](https://web.archive.org/web/20221027145810/https://github.com/eugenp/tutorials/tree/master/quarkus-modules/quarkus-vs-springboot)