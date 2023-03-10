# Java 周刊，第 420 期

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-weekly-420>

## 1。Spring 和 Java

[**>>JVM 上容错库的初步比较**](https://web.archive.org/web/20220525135839/https://blog.frankel.ch/comparison-fault-tolerance-libraries/)[blog . frankel . ch]

JVM、故障和分布式系统—**处理 JVM 生态系统中的** **重试、超时、电路中断、速率限制、**等等。

[**> >让你的 RestTemplate 容错有弹性 4j**](https://web.archive.org/web/20220525135839/https://arnoldgalovics.com/resilience4j-resttemplate/)[【arnoldgalovics.com】T4

如何通过使用支持断路器的 RestTemplate 给有故障的下游服务一些时间来恢复。有趣的读物。

[**> >用 Spring Boot 追踪，OpenTelemetry，还有耶格**](https://web.archive.org/web/20220525135839/https://reflectoring.io/spring-boot-tracing/) [ reflectoring.io

分布式追踪的综合指南:为什么我们需要它，概念，以及在 Spring Boot 如何做。

#### 同样值得一读:

*   **[> >你不用把他们都抓住](https://web.archive.org/web/20220525135839/https://jaokim.github.io/2022/01/11/stackoverflow.html)**[Jao Kim . github . io
*   **[> >用弹簧键集分页](https://web.archive.org/web/20220525135839/https://vladmihalcea.com/keyset-pagination-spring/)**【vladmihalcea.com】
*   **[> >分发唯一的时间戳标识符](https://web.archive.org/web/20220525135839/http://blog.vanillajava.blog/2022/01/distributed-unique-time-stamp.html)**[blog . vanilla Java . blog
*   **[> >并发 linked hashset](https://web.archive.org/web/20220525135839/https://www.javaspecialists.eu/archive/Issue296-Concurrent-LinkedHashSet.html)**[javaspecialists . eu
*   [**> >引入 Spring 集成 Groovy DSL**](https://web.archive.org/web/20220525135839/https://spring.io/blog/2022/01/06/introducing-spring-integration-groovy-dsl)[Spring . io]
*   [**> > Java:对象重用如何减少延迟和提高性能**](https://web.archive.org/web/20220525135839/https://minborgsjavapot.blogspot.com/2022/01/java-how-object-reuse-can-reduce.html)【minborgsjavapot.com】

**网络研讨会和演示:**

*   **[> > Java 17 深潜](https://web.archive.org/web/20220525135839/https://inside.java/2022/01/11/java-17-deep-dive/)**【inside.java】
*   [**> > Java & JVM 面板**](https://web.archive.org/web/20220525135839/https://www.infoq.com/presentations/java-fast-release-cadence "Introduction to Spring Cloud Load Balancer")【infoq.com】
*   **[> >科特林](https://web.archive.org/web/20220525135839/https://www.infoq.com/presentations/differentiable-framework-kotlin/)中的可微编程**【infoq.com中的
*   **[> >记录模式、性能与系列化——JEP 咖啡馆# 8](https://web.archive.org/web/20220525135839/https://inside.java/2022/01/06/jepcafe8/)**【inside.java】
*   [**> >精彩播客:春云联合创始人斯潘塞·吉布(新年快乐！)**](https://web.archive.org/web/20220525135839/https://spring.io/blog/2022/01/06/a-bootiful-podcast-spring-cloud-cofounder-spencer-gibb-and-happy-new-year)spring . io
*   [**>>Java SE——写一次，永远运行**](https://web.archive.org/web/20220525135839/https://inside.java/2022/01/10/write-once-run-forever/ "Introduction to Spring Cloud Load Balancer")【inside.java】

**升级时间:**

*   [**> > Spring 框架 CVE-2021-22060 已发布**](https://web.archive.org/web/20220525135839/https://spring.io/blog/2022/01/05/spring-framework-cve-2021-22060-has-been-published)Spring . io
*   [**> >冬眠搜索 6.1.0.Beta2 发布**](https://web.archive.org/web/20220525135839/https://in.relation.to/2022/01/05/hibernate-search-6-1-0-Beta2/)
*   [**> >用 j releaser**](https://web.archive.org/web/20220525135839/https://andresalmiray.com/releasing-rust-binaries-with-jreleaser/)【andresalmiray.com】释放铁锈二进制文件
*   [**> >夸尔库斯 2.6.2 .最终发布——维护发布**](https://web.archive.org/web/20220525135839/https://quarkus.io/blog/quarkus-2-6-2-final-released/) [ 夸尔库斯. io

## 2。技术

[**> >分布式系统的模式:复制日志**](https://web.archive.org/web/20220525135839/https://martinfowler.com/articles/patterns-of-distributed-systems/replicated-log.html)【martinfowler.com】

通过在多个节点之间复制 WAL 日志，对分布式系统的状态达成**共识。**

**同样值得一读:**

*   [**> > Paxos:即使在节点断开**](https://web.archive.org/web/20220525135839/https://martinfowler.com/articles/patterns-of-distributed-systems/paxos.html)[【martinfowler.com】]的情况下，使用两个共识建立阶段来达成安全共识
*   [**> >不要重新发明日期格式**](https://web.archive.org/web/20220525135839/https://techblog.bozho.net/dont-reinvent-date-formats/)【techblog.bozho.net】
*   [**> >用 Azure DevOps 完成项目文档——用 plant UML**](https://web.archive.org/web/20220525135839/https://blog.codecentric.de/en/2022/01/getting-project-documentation-done-with-azure-devops-diagrams-with-plantuml/)[blog . code centric . de]绘制图表
*   [**> >锈任一部分 1:锈中扩展地图**](https://web.archive.org/web/20220525135839/https://lucumr.pocoo.org/2022/1/6/rust-extension-map/)【lucumr.pocoo.org】
*   [**> >服务对服务呼叫模式——使用 Anthos 服务网格**](https://web.archive.org/web/20220525135839/http://www.java-allandsundry.com/2022/01/service-to-service-call-pattern-using.html)【java-allandsundry.com】
*   [**> > Datafaker，替代生产数据**](https://web.archive.org/web/20220525135839/https://jworks.io/datafaker-an-alternative-to-production-data/) [ jworks.io
*   [**> >为远程团队制作极限编程**](https://web.archive.org/web/20220525135839/https://tanzu.vmware.com/content/blog/extreme-programming-remote-teams)【tanzu.vmware.com

## 3。沉思

[**> >解题正确(2022)**](https://web.archive.org/web/20220525135839/https://abdullin.com/solving-the-right-problem/)【abdullin.com】

当好的工程解决方案解决了错误的问题时，它们可能会成为一种负担！

**同样值得一读:**

*   [**> >依赖风险与资助**](https://web.archive.org/web/20220525135839/https://lucumr.pocoo.org/2022/1/10/dependency-risk-and-funding/)【lucumr.pocoo.org】
*   [**> >用 Zettelkasten 方法管理知识**](https://web.archive.org/web/20220525135839/https://blog.scottlogic.com/2022/01/04/managing-knowledge-zettelkasten.html)【blog.scottlogic.com】
*   [**> >你能用特斯拉机器人做什么？**](https://web.archive.org/web/20220525135839/https://pointersgonewild.com/2022/01/08/whats-could-you-use-tesla-bot-for/)【pointersgonewild.com】
*   [**> >模块化与性能**](https://web.archive.org/web/20220525135839/https://blog.thecodewhisperer.com/permalink/modularity-and-performance)【blog.thecodewhisperer.com】
*   [**> >暴民编程的一年，第四部分:it 的遥远**](https://web.archive.org/web/20220525135839/https://www.giorgiosironi.com/2022/01/a-year-of-mob-programming-part-4.html)【giorgiosironi.com
*   [**> >向骗子**](https://web.archive.org/web/20220525135839/https://www.bitquabit.com/post/learning-writing-and-coding-from-a-con-artist/)【bitquabit.com】学习写作和编码
*   [**> >更包容的未来。一起。**](https://web.archive.org/web/20220525135839/https://blog.scottlogic.com/2022/01/04/a-more-inclusive-future-together.html)【blog.scottlogic.com】

## 4。漫画

本周我最喜欢的迪尔伯特。

[**> >照顾它**](https://web.archive.org/web/20220525135839/https://dilbert.com/strip/2022-01-09)【dilbert.com】

[**> >机器人替 Boss**](https://web.archive.org/web/20220525135839/https://dilbert.com/strip/2022-01-08)【dilbert.com】

[**>>**](https://web.archive.org/web/20220525135839/https://dilbert.com/strip/2022-01-05)【dilbert.com】

## 5。本周精选

**[> >如何获得幸运:关注肥尾](https://web.archive.org/web/20220525135839/https://taylorpearson.me/luck/)**[Taylor Pearson . me]

Next **»**[Java Weekly, Issue 421](/web/20220525135839/https://www.baeldung.com/java-weekly-421)**«** Previous[Java Weekly, Issue 419](/web/20220525135839/https://www.baeldung.com/java-weekly-419)