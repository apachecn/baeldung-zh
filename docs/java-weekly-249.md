# Java 周刊，第 249 期

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-weekly-249>

**开始了……**

## 1。Spring 和 Java

#### [**> >探索新 Java 10 "var "类型:介绍和动手教程**](https://web.archive.org/web/20220707143819/https://www.infoq.com/articles/java-10-var-type)【infoq.com】

一篇关于局部变量的**类型推理的精彩文章，一个旨在**减少样板代码**的闪亮新特性。**

#### [**>>spring one 平台上的无功革命 2018(part 1/N)**](https://web.archive.org/web/20220707143819/https://spring.io/blog/2018/09/27/the-reactive-revolution-at-springone-platform-2018-part-1-n)spring . io

一个伟大的新系列从两个很酷的话题开始——反应式 SQL 数据访问和 RSocket 协议。InfoQ 上有几篇关于 [R2DBC](https://web.archive.org/web/20220707143819/https://www.infoq.com/news/2018/10/springone-r2dbc) 和 [RSocket](https://web.archive.org/web/20220707143819/https://www.infoq.com/news/2018/10/rsocket-facebook) 的可靠报道。

#### [**> >结构化 JUnit 5 测试**](https://web.archive.org/web/20220707143819/https://blog.codecentric.de/en/2018/09/structured-junit-5-testing/)[blog . codecentric . de]

一个聪明的方法是**为一个类**组织 BDD 风格的测试，使用带有内部类的`@Nested`注释对具有共同前提条件的测试进行分组，并为跨设置执行相同行为的测试抽象超类。非常酷。

#### [**> >春季数据 Lovelace 有什么新内容？**](https://web.archive.org/web/20220707143819/https://spring.io/blog/2018/09/27/what-s-new-in-spring-data-lovelace)spring . io

最新的 Spring 数据发布系列现在是 GA，它包含了一些**强大的新特性**。并找出这次更新对 [Redis 和 Apache Cassandra](https://web.archive.org/web/20220707143819/https://spring.io/blog/2018/09/26/what-s-new-in-spring-data-lovelace-for-redis-and-apache-cassandra) 以及 [MongoDB](https://web.archive.org/web/20220707143819/https://spring.io/blog/2018/09/27/what-s-new-in-spring-data-lovelace-for-mongodb) 意味着什么。

#### [**> > Kotlin 和 MongoDB，绝配**](https://web.archive.org/web/20220707143819/https://blog.philipphauer.de/kotlin-mongodb-perfect-match/)[blog . Philipp hauer . de]

一篇可靠的文章展示了为什么 Kotlin 固有的语言特性使其比 Java 更适合使用 MongoDB 的动态模式。

#### [**> > 10 Maven 安全最佳实践**](https://web.archive.org/web/20220707143819/https://snyk.io/blog/10-maven-security-best-practices)snyk . io

当谈到**保护代码库中的敏感信息**时，构建工具配置很容易被新手忽略。请务必查看 Maven 项目的安全备忘单。

#### [**> >科特林集合 API 性能反模式**](https://web.archive.org/web/20220707143819/https://4comprehension.com/kotlin-collections-api-performance-antipatterns/)【4comprehension.com】

请记住，Kotlin 的集合并不像 Java Stream API 那样懒惰。

#### 同样值得一读:

*   #### [>>](https://web.archive.org/web/20220707143819/https://marxsoftware.blogspot.com/2018/09/a-tale-of-two-oracle-jdks.html)【max software . blogspot . com】

*   #### [> > Java 11 Manual: Has Java 11 checked all the correct boxes?](https://web.archive.org/web/20220707143819/https://jaxenter.com/manual-java-11-first-impression-part-2-150153.html) [【 jaxenter.com】T2

*   #### [**> >春天框架 5.1 搭载 Java 11 支持**](https://web.archive.org/web/20220707143819/https://www.infoq.com/news/2018/09/spring-51-java-11)【infoq . com】T5

*   #### [**> > Evolution of Spring**](https://web.archive.org/web/20220707143819/https://spring.io/blog/2018/10/02/the-evolution-of-spring-fu) Spring. IO

*   #### [**>如何将@RequestParam 绑定到春天中的对象**](https://web.archive.org/web/20220707143819/http://dolszewski.com/spring/how-to-bind-requestparam-to-object/)【dolszewski . com】T5

*   #### [**> > Detection coverage is dead-long live mutation detection**](https://web.archive.org/web/20220707143819/https://medium.com/appsflyer/tests-coverage-is-dead-long-live-mutation-testing-7fd61020330e) [medium.com]

*   #### [**> > Extend Swagger Codegen with new moustache template file and use new language**](https://web.archive.org/web/20220707143819/https://blog.arnoldgalovics.com/extending-swagger-codegen-with-new-mustache-template-files-using-a-new-language/) [blog.arnoldgalovics.com

*   #### [**>微文件容错,取 2【tomi tribe . com】T3**](https://web.archive.org/web/20220707143819/https://www.tomitribe.com/blog/microprofile-fault-tolerance-take-2/)T5[T7>微文件容错注解【T8](https://web.archive.org/web/20220707143819/https://www.tomitribe.com/blog/microprofile-fault-tolerance-annotations/)【tomi tribe . com】

#### 网络研讨会和演示:

*   上周在华盛顿举行的 SpringOne Platform 2018 大会上的几场演讲，包括 Juergen Hoeller 关于 **[新 Java SE 发布节奏如何影响 Spring 框架](https://web.archive.org/web/20220707143819/https://www.youtube.com/watch?v=onZJ8beVEtI)** ，Dave Syer 关于 **[云抽象](https://web.archive.org/web/20220707143819/https://www.youtube.com/watch?v=icZaMdNExNU)** ，Jon Schneider 关于 **[连续交付和 Spinnaker](https://web.archive.org/web/20220707143819/https://www.youtube.com/watch?v=xcD4mWo_YHE)** ，以及 Oliver gier ke&Rossen Stoyanchev 关于 **[反应式数据访问与](https://web.archive.org/web/20220707143819/https://www.youtube.com/watch?v=E3s5f-JF8z4)**
*   #### [**> > Java 11(播放列表)**](https://web.archive.org/web/20220707143819/https://www.youtube.com/playlist?list=PLX8CzqL3ArzXyA_lJzaNmrFqpLOL4aCEz) 【youtube.com】

*   #### [**> > Micro-service method in traditional enterprise environment boundary**](https://web.archive.org/web/20220707143819/https://www.infoq.com/presentations/microservices-blueprint) [infoq.com]

*   #### [**>传输层安全(TLS)1.3**](https://web.archive.org/web/20220707143819/https://www.youtube.com/watch?v=g79M8A-wg-s)【YouTube . com】

**升级时间:**

*   #### [> >春批 4。1 .0 .rc1 现已上市](https://web.archive.org/web/20220707143819/https://spring.io/blog/2018/09/26/spring-batch-4-1-0-rc1-is-now-available) 春天。io

*   #### [**>春安 5.1 谬**](https://web.archive.org/web/20220707143819/https://spring.io/blog/2018/09/27/spring-security-5-1-goes-ga) 嘎春。io

*   #### [**>春穹 2.1 GA 发布**](https://web.archive.org/web/20220707143819/https://spring.io/blog/2018/10/02/spring-vault-2-1-ga-released) 春天。io

*   #### [**> >帕亚拉基金会发布帕亚拉服务器和帕亚拉微 5.183，支持微简介 2.0**](https://web.archive.org/web/20220707143819/https://www.infoq.com/news/2018/09/payara-releases-version-5.183) 【infoq.com】

*   #### [**> >玻璃鱼的新时代**](https://web.archive.org/web/20220707143819/https://www.infoq.com/news/2018/09/a-new-era-for-glassfish)【infoq . com】T5

*   #### [**> >冬眠 GM 5。4 .0 .cr1 发布**](https://web.archive.org/web/20220707143819/http://in.relation.to/2018/10/01/hibernate-ogm-5-4-CR1-released/)

## 2。技术

#### [**> >建模不确定性与无功【infoq.com】**](https://web.archive.org/web/20220707143819/https://www.infoq.com/articles/modeling-uncertainty-reactive-ddd)

一篇关于在反应式分布式系统中应用领域驱动设计建模技术的深思熟虑的文章。

#### [**> >该不该学 C 来“学习计算机的工作原理”？**](https://web.archive.org/web/20220707143819/https://words.steveklabnik.com/should-you-learn-c-to-learn-how-the-computer-works)【words.steveklabnik.com】

或者更准确地说，你应该**“学习 C 来学习`more`计算机如何工作？”**

**同样值得一读:**

*   #### [**> > How to use SQL to update .. It is more efficient to run DML back**](https://web.archive.org/web/20220707143819/https://blog.jooq.org/2018/09/26/how-to-use-sql-update-returning-to-run-dml-more-efficiently/) [blog.jooq.org]

*   #### [**> > Forcibly solve a seemingly simple number puzzle**](https://web.archive.org/web/20220707143819/https://www.nurkiewicz.com/2018/09/brute-forcing-seemingly-simple-number.html) [nurkiewicz.com]

*   #### [**> > Playing porcupine with Prometheus instrument & Grafa Na** T3 [blog.sebastian-daschner.com] T4](https://web.archive.org/web/20220707143819/https://blog.sebastian-daschner.com/entries/porcupine-metrics-grafana)

*   #### [**>学习 Clojure:箭头和多托宏**](https://web.archive.org/web/20220707143819/https://blog.frankel.ch/learning-clojure/2/)博客。弗兰克尔。ch

*   #### [**> > Yarn receiving: the yarn in the yarn is started by swallowing, and when it is useful**](https://web.archive.org/web/20220707143819/https://dev.to/frosnerd/yarnception-starting-yarn-within-yarn-through-gulp-and-when-it-is-useful-og3) develops to

*   #### [**> > How to safely check your LastPass vault**](https://web.archive.org/web/20220707143819/https://advancedweb.hu/2018/10/02/lastpass_pwned_passwords/)

    according to the password database Advanced Web.hu ?
*   #### [**> >安卓测试:AWS Device Farm vs Firebase test lab**](https://web.archive.org/web/20220707143819/https://blog.codecentric.de/en/2018/10/android-testing-aws-device-farm-vs-firebase-testlab/)[博客。代码为中心。德

*   #### [**> > Debugging and tampering with HTTP traffic using ZAP-proxy and nginx-simulating timeout and other unexpected behaviors**](https://web.archive.org/web/20220707143819/https://vanwilgenburg.wordpress.com/2018/10/02/zap-proxy-and-nginx/) [[vanwilgenburg.wordpress.com]]

## 3。沉思

#### [**> >反思网飞的边缘负载均衡**](https://web.archive.org/web/20220707143819/https://netflixtechblog.com/netflix-edge-load-balancing-695308b5548c)【medium.com】

深入探究从 Zuul 中学到的**经验如何带来了几项改进**，包括减少了因服务器过载而导致的错误。

#### [**> >窄小的小生:什么时候窄小才算太窄小？**](https://web.archive.org/web/20220707143819/https://daedtech.com/narrow-niche-when-is-narrow-too-narrow/)【daedtech.com】

如果你正在考虑开一个博客，开始的时候要宽一些，然后反复缩小你的关注范围。

**同样值得一读:**

*   #### [**>型号为电子标识**](https://web.archive.org/web/20220707143819/https://techblog.bozho.net/models-for-electronic-identification/)【techblog . bozho . net】

*   #### [**> > What are upstream and downstream in software development?**](https://web.archive.org/web/20220707143819/https://reflectoring.io/upstream-downstream/) [ reflector ing . io ]

*   #### [**> > OverOps automatic timer and performance monitoring in Splunk**](https://web.archive.org/web/20220707143819/https://blog.takipi.com/overops-automated-timers-and-performance-monitoring-in-splunk/) [blog.takipi.com]

*   #### [**> > Is the event flow a new event in the financial industry?**](https://web.archive.org/web/20220707143819/https://www.confluent.io/blog/event-streaming-new-big-thing-finance) [ Huihe.io ]

## 4。漫画

本周我最喜欢的迪尔伯特。

#### [**> >呆伯特的心流在办公室被打断**](https://web.archive.org/web/20220707143819/http://dilbert.com/strip/2018-09-30)【dilbert.com】

#### [**> >为什么这么消极？**](https://web.archive.org/web/20220707143819/http://dilbert.com/strip/2018-09-28)【dilbert.com】

#### [**> >爱丽丝开创先例**](https://web.archive.org/web/20220707143819/http://dilbert.com/strip/2015-05-29)【dilbert.com】

## 5。本周精选

#### [**> >加班伤害你的软件&你的团队**](https://web.archive.org/web/20220707143819/https://medium.com/@plainprogrammer/overtime-hurts-your-software-your-team-1c16c99e28aa)【medium.com】

下一期[Java 周刊，第 250 期](/web/20220707143819/https://www.baeldung.com/java-weekly-250)上一期[Java 周刊，第 248 期](/web/20220707143819/https://www.baeldung.com/java-weekly-248)