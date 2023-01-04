# Java 周刊，第 235 期

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-weekly-235>

**开始了……**

## 1。Spring 和 Java

#### [**> >当前状态的 Java 值类型**](https://web.archive.org/web/20220701015450/https://www.infoq.com/news/2018/06/JavaValuesJun18)【infoq.com】

一篇有趣的帖子，强调了 Java 中值类型的可能语义，以及 Oracle 的 JVM 开发人员在发展这一经常被请求的特性时所面临的一些障碍。

#### [**> >描无功流——用春云探子带靴 2**](https://web.archive.org/web/20220701015450/http://www.java-allandsundry.com/2018/06/tracing-reactive-flow-using-spring.html)【java-allandsundry.com】

快速阅读，展示如何**为一个反应式 Spring Boot 应用程序捕获分布式跟踪数据**并在 Zipkin UI 中显示它。好东西。

#### [**> > WireMock 教程:请求匹配，第三部分**](https://web.archive.org/web/20220701015450/https://www.petrikainulainen.net/programming/testing/wiremock-tutorial-request-matching-part-three/)【petrikainulainen.net】

本系列的最新部分是关于指定对 JSON 请求体内容的期望。

#### [> >当使用 JPA 和 Hibernate](https://web.archive.org/web/20220701015450/https://vladmihalcea.com/entitymanager-find-getreference-jpa/)[vladmihalcea.com]时，查找和获取 Reference EntityManager 方法是如何工作的

这是一篇关于 JPA 中一个鲜为人知的方法——`getReference`——的精彩文章，该方法在创建`@OneToOne`和`@ManyToOne`关联时能够**提升性能。非常酷。**

#### 同样值得一读:

*   #### [**> > Java ternary automatic packing/unpacking is tricky**](https://web.archive.org/web/20220701015450/https://marxsoftware.blogspot.com/2018/06/javas-ternary-is-tricky.html) [marxsoftware.blogspot.com]

*   #### [**> > Spring Tips: Review of the fourth season**](https://web.archive.org/web/20220701015450/https://spring.io/blog/2018/06/20/spring-tips-season-4-recap) [ spring.io ]

*   #### [**> > Oracle Bone Inscriptions announced the new Java support pricing structure**](https://web.archive.org/web/20220701015450/https://www.infoq.com/news/2018/06/new-support-pricing-java) [infoq.com

*   #### [**> > The influence of micro-archive community on Jakarta EE**](https://web.archive.org/web/20220701015450/https://www.infoq.com/news/2018/06/microprofile-influence-jakartaee) [infoq.com]

*   #### [**> > Deferred execution by Java consumers**](https://web.archive.org/web/20220701015450/https://marxsoftware.blogspot.com/2018/06/deferred-execution-java-consumer.html) [marxsoftware.blogspot.com] Deferred execution

*   #### [**> > Check the compiled version and time at runtime [vojtechruzicka.com]**](https://web.archive.org/web/20220701015450/https://www.vojtechruzicka.com/spring-boot-version/)

*   #### [**> >美文:使用 JUnit 5 和 Spring Boot 进行单元和集成测试**](https://web.archive.org/web/20220701015450/https://info.michael-simons.eu/2018/06/18/maven-use-junit-5-with-spring-boot-for-unit-and-integration-tests/)信息。迈克尔.西蒙斯。欧盟

*   #### [**>带功能弹簧的中情局世界概况 API**【e4developer.comT4](https://web.archive.org/web/20220701015450/https://www.e4developer.com/2018/06/22/cia-world-factbook-api-with-functional-spring/)

*   #### [**> > Optional spring**](https://web.archive.org/web/20220701015450/http://blog.marcosbarbero.com/optional-di-spring/) [blog.marcosbarbero.com]

    can be used for injection.

**网络研讨会和演示:**

*   #### [**> > Distributed tracking: use Spring Cloud to analyze the delay of your microservices & Zipkin**](https://web.archive.org/web/20220701015450/https://www.infoq.com/presentations/distributed-tracing-spring-cloud-zipkin) [[infoq.com] T4

*   #### [**>提高您的命令行效率（视频**](https://web.archive.org/web/20220701015450/https://blog.sebastian-daschner.com/entries/unix-command-line-productivity))

*   #### [**> > Author Stefan Nicole/brian crozier @ springi/o2018**](https://web.archive.org/web/20220701015450/https://www.youtube.com/watch?v=E3I7SlZ2QdU) [youtube.com]

*   #### [**> > Introduction of Chunyun Gateway by Spencer Gibb @ springi/O 2018**](https://web.archive.org/web/20220701015450/https://www.youtube.com/watch?v=NkgooKSeF8w) [youtube.com]

*   #### [**>用弹簧支架文档记录 RESTful API**](https://web.archive.org/web/20220701015450/https://www.infoq.com/presentations/documentation-api-spring-rest)infoq . com

**升级时间:**

*   #### [**> >阿帕奇 Geode/Pivotal GemFire 2。0 .3 .释放；排放；发布发布的春季会议！**](https://web.archive.org/web/20220701015450/https://spring.io/blog/2018/06/21/spring-session-for-apache-geode-pivotal-gemfire-2-0-3-release-released) 春天。io

*   #### [**> > Dormancy Search Second Maintenance Release 5.10**](https://web.archive.org/web/20220701015450/http://in.relation.to/2018/06/22/hibernate-search-5-10-2-Final/)

*   #### [**> > IntelliJ 理念 2018.2 中的 Java 11**](https://web.archive.org/web/20220701015450/https://blog.jetbrains.com/idea/2018/06/java-11-in-intellij-idea-2018-2/)【blog . jetbrains . com】T5

*   #### [**> > IntelliJ 理念 2018.2 公测**](https://web.archive.org/web/20220701015450/https://blog.jetbrains.com/idea/2018/06/intellij-idea-2018-2-goes-beta/)【blog . jetbrains . com】

*   #### [**> >吉普斯特发布 v 5。0 .0**](https://web.archive.org/web/20220701015450/https://www.jhipster.tech/2018/06/20/jhipster-release-5.0.0.html) 吉普斯特。tech

## **2。技术和思考**

#### [**> >一个放之四海而皆准的数据库不适合任何人**](https://web.archive.org/web/20220701015450/https://www.allthingsdistributed.com/2018/06/purpose-built-databases-in-aws.html)【allthingsdistributed.com】

对我们可以使用的不同类型的非关系数据库以及它们最适合解决什么样的问题进行了令人耳目一新的回顾。

#### [**> >那个 bug 是怎么发生的？Git 平分救援！**](https://web.archive.org/web/20220701015450/https://odino.org/how-did-that-bug-happen-git-bisect-to-the-rescue/)【odino.org】

一个聪明的命令可以**通过确定哪个提交给你的库引入了一个 bug 来显著减少调试时间**。

#### [**> >如何编写一个 Kotlin DSL——例如为阿帕奇卡夫卡**](https://web.archive.org/web/20220701015450/https://blog.codecentric.de/en/2018/06/kotlin-dsl-apache-kafka/)[blog . code centric . de]

一篇很酷的文章，展示了在创建 DSL 时 Kotlin 的扩展函数和函数的 lambda 参数的效用。

#### [**> >代码评审中的礼貌还是直言？一劳永逸**](https://web.archive.org/web/20220701015450/https://daedtech.com/politeness-bluntness-code-review/)daedtech.com

一篇深思熟虑的文章提醒评论者**根据你的读者——被评论者——和他们的文化规范，在微妙和直率**之间寻求适当的平衡。

**同样值得一读:**

*   #### [**> > Let's encrypt the hook use case**](https://web.archive.org/web/20220701015450/https://advancedweb.hu/2018/06/26/letsencrypt_hook_use_cases/) [ Advanced Web.hu ]

*   #### [**>通过自制软件分发网络应用**](https://web.archive.org/web/20220701015450/https://blog.frankel.ch/distributing-desktop-webapps/2/) [ 博客。弗兰克尔。ch

*   #### [**>>6 git 啊哈力矩**](https://web.archive.org/web/20220701015450/https://henrikwarne.com/2018/06/25/6-git-aha-moments/) 【henrikawarne】。

    com】
*   #### [> >黑盒子写起来——话匣子](https://web.archive.org/web/20220701015450/https://codemonkeyism.co.uk/htb-chatterbox/)【【codemonkeyism.co.uk】T4

*   #### [**> > Six log management tools you need to know (and how to use them)**](https://web.archive.org/web/20220701015450/https://blog.takipi.com/6-log-management-tools-you-need-to-know-and-how-to-use-them/) [blog.takipi.com]

*   #### [**>整数和估计数**](https://web.archive.org/web/20220701015450/http://blog.cleancoder.com/uncle-bob/2018/06/21/IntegersAndEstimates.html)【blog . clean coder . com】

*   #### [**>用 Zsh 别名**](https://web.archive.org/web/20220701015450/https://blog.sebastian-daschner.com/entries/zsh-aliases)提高壳生产率

*   #### [**> > What is the best way to pair?**](https://web.archive.org/web/20220701015450/https://builttoadapt.io/whats-the-best-way-to-pair-a8699f9beb81) [ builttoadapt . io ]

## 3。漫画

本周我最喜欢的迪尔伯特。

#### [**> >都只是零和一**](https://web.archive.org/web/20220701015450/http://dilbert.com/strip/1996-09-18)【dilbert.com】

#### [**> >无知是福**](https://web.archive.org/web/20220701015450/http://dilbert.com/strip/2014-08-24)【dilbert.com】

#### [**> >老大哥在看你**](https://web.archive.org/web/20220701015450/http://dilbert.com/strip/2018-06-27)【dilbert.com】

## 4。本周精选

#### [> >说硬的](https://web.archive.org/web/20220701015450/http://randsinrepose.com/archives/say-the-hard-thing/)【randsinrepose.com】

下一期[Java 周刊，第 236 期](/web/20220701015450/https://www.baeldung.com/java-weekly-236)上一期[Java 周刊，第 234 期](/web/20220701015450/https://www.baeldung.com/java-weekly-234)