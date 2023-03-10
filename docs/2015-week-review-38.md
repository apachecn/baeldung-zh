# Java Web Weekly 38(前称“Baeldung Weekly Review”)

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/2015-week-review-38>

本周，我宣布并实施**“`Baeldung Weekly Review`”的更名**。新的周报将被命名为**“Java Web Weekly”**。

改变背后的原因很简单——我发现旧名字并没有真正向新读者传达评论的内容。新名字更加清晰，也非常符合内容和我自己的关注点。

当然，除了名字之外，评论也是一样的。

**开始了……**

## 1。Spring 和 Java

#### [> > React.js 和 Spring Data REST:第二部分——超媒体](https://web.archive.org/web/20220126113444/https://spring.io/blog/2015/09/15/react-js-and-spring-data-rest-part-2-hypermedia) [ spring.io

Spring Data REST 的杰作，它让在 API 中烘焙许多超媒体的优点变得多么容易。只有少数 API 能做到这一点，做得好的就更少了。

超媒体控件是我最喜欢的将 API 提升一个档次的东西之一，尤其是现在我正接近录制课程 7(进化、发现和记录 REST API)的时候。我正在考虑有一个关于 Spring Data REST 的部分，但是为了公平起见，我可能不得不为它奉献一整个额外的课程。

#### [> >用 JPA 静态元模型](https://web.archive.org/web/20220126113444/http://www.thoughts-on-java.org/static-metamodel/)【thoughts-on-java.org】创建类型安全查询

探索 JPA 中很酷的静态元模型助手类。一旦你完成了生成这些的过程——它们对于**编写流畅、干净的持久级逻辑**真的很方便。

#### [> >流表演——你的想法](https://web.archive.org/web/20220126113444/http://blog.codefx.org/java/stream-performance-your-ideas/)【codefx.org】

除了上周的结果之外，还有关于 Java 8 流性能的新数据。

#### [> >哈希策略优化简介](https://web.archive.org/web/20220126113444/https://vanillajava.blogspot.ro/2015/09/an-introduction-to-optimising-hashing.html?view=classic)vanillajava

有趣的深入探讨**改进我们日常使用的散列策略**。

#### AssertJ 的软断言——我们需要它们吗？ [ codeleak.pl ]

**软断言是一个新概念**(对我来说)–我可能会在实际使用它们时非常有选择性，但我肯定能看到在一些场景中，它们会非常非常方便。

#### [> > JDK 9:高亮显示来自模块系统的状态](https://web.archive.org/web/20220126113444/https://marxsoftware.blogspot.ro/2015/09/jdk9-state-of-module-system.html)Marx software

从上周发布的关于 Java 9 模块化将如何表现的官方信息中摘录一些内容。我喜欢短小精悍的音符。

#### [> >使用 MoreUnit 进行自动化测试](https://web.archive.org/web/20220126113444/https://advancedweb.hu/2015/09/17/eclipse_tests_with_moreunit/)[advanced web . Hu]

可能很容易忽略 IDE 的一些小毛病，但是改进您的技术和工作流程总是有好处的。如果你正在做 TDD，这里有一个看起来很有前途的 Eclipse 插件。

也值得一读:

*   #### [>覆盖用格雷德](https://web.archive.org/web/20220126113444/http://blog.codeleak.pl/2015/09/override-spring-framework-version-in.html)代码泄露。pl ]构建的 Spring Boot 应用中的春天框架版本

*   #### [> > Compare imperative and functional algorithms in Java 8](https://web.archive.org/web/20220126113444/http://blog.jooq.org/2015/09/17/comparing-imperative-and-functional-algorithms-in-java-8/) [jooq.org]

*   #### [> > Build a real-time chat application](https://web.archive.org/web/20220126113444/http://code.tutsplus.com/tutorials/build-a-real-time-chat-application-with-modulus-and-spring-boot--cms-22513)

    with modulus and Spring Boot [tutsplus.com]
*   #### [> > JAR manifest class-path is not used for Java application launcher, but only for](https://web.archive.org/web/20220126113444/https://marxsoftware.blogspot.ro/2015/09/jar-manifest-class-path-javac.html) [ Marx Software

*   #### [> > Write clean log code Lambdas](https://web.archive.org/web/20220126113444/https://garygregory.wordpress.com/2015/09/16/a-gentle-introduction-to-the-log4j-api-and-lambda-basics/) Gary Gregory

    in Java 8.

网络研讨会和演示:

*   #### [>>Maurice NAFTA Lin on Java Lambdas，Java 8 Streams，Parallelism](https://web.archive.org/web/20220126113444/http://www.infoq.com/interviews/naftalin-lambda-streams)

*   #### [> > Java 9 模块化](https://web.archive.org/web/20220126113444/http://paulbakker.io/java/java-9-modularity/) 保尔巴克。io

*   #### [> > Common sense drives development [talk]](https://web.archive.org/web/20220126113444/http://techblog.bozho.net/common-sense-driven-development-talk/) [bozho.net]

#### 升级时间:

*   #### [>>2.7 发布](https://web.archive.org/web/20220126113444/https://discuss.gradle.org/t/gradle-2-7-released/11623)

*   #### [> > Elasticity Search 1.7.2](https://web.archive.org/web/20220126113444/https://www.elastic.co/downloads/past-releases/elasticsearch-1-7-2)

*   #### [> T5【Spring Boot】1。2 .6](https://web.archive.org/web/20220126113444/https://search.maven.org/classic/#artifactdetails|org.springframework.boot|spring-boot|1.2.6.RELEASE|jar)

## 2。技术

#### [> >在 CQRS 实现与线性事件存储](https://web.archive.org/web/20220126113444/http://squirrel.pl/blog/2015/09/14/achieving-consistency-in-cqrs-with-linear-event-store/) [ squirrel.pl ]的一致性

这个以事件为中心的系列文章的第二部分——详细介绍**选择事件存储并与之有效交互的选择**。

也值得一读:

*   #### [> > Be good at testing your own software](https://web.archive.org/web/20220126113444/http://www.daedtech.com/get-good-at-testing-your-own-software) [daedtech.com]

*   #### [>伐木——终极指南](https://web.archive.org/web/20220126113444/https://www.loggly.com/ultimate-guide/) 【loggly.com】

*   #### [> > Improvement of Line 53-Computational health check and delayed check](https://web.archive.org/web/20220126113444/https://aws.amazon.com/blogs/aws/route-53-improvements-calculated-health-checks-and-latency-checks/) [aws.amazon.com]

*   #### [> > Benchmark Aurora vs MySQL: Is Amazon's new DB really 5 times faster?](https://web.archive.org/web/20220126113444/http://blog.takipi.com/benchmarking-aurora-vs-mysql-is-amazons-new-db-really-5x-faster/) 【 takipi.com】

*   #### [> >建筑分布式日志:Twitter 的高性能复制日志服务](https://web.archive.org/web/20220126113444/https://blog.twitter.com/2015/building-distributedlog-twitter-s-high-performance-replicated-log-service) 【twitter.com】

## 3。沉思

#### [> >戴上航运护目镜](https://web.archive.org/web/20220126113444/https://signalvnoise.com/posts/3931-putting-on-the-shipping-goggles)【signalvnoise.com】

如果你曾经运送过任何东西，尤其是你自己的作品，这件作品真的很棒。

也值得一读:

*   #### [> > Hacker vs Scholar: Who is responsible for progress?](https://web.archive.org/web/20220126113444/http://lemire.me/blog/archives/2015/09/11/hackers-vs-academics-who-is-responsible-for-progress/) [ lemire.me ]

*   #### [> > Things I was unprepared for as the chief developer](https://web.archive.org/web/20220126113444/http://dev-human.com/entries/2015/09/07/things-i-was-unprepared-for/) [dev-human.com]

*   #### [>如何熟练工游](https://web.archive.org/web/20220126113444/http://blog.code-cop.org/2015/09/how-to-journeyman-tour.html) 【code-cop.org】

## 4。本周精选

除了名称的变化，pick 部分的另一个小变化是我去掉了锁机制。

这里有一个非常酷的减价应用程序——如果你正在写作的话:

#### [> >迪林杰](https://web.archive.org/web/20220126113444/https://dillinger.io/)