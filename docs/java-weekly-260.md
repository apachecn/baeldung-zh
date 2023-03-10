# Java 周刊，第 260 期

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-weekly-260>

**开始了……**

## 1。Spring 和 Java

#### [**> >春天有多快？**](https://web.archive.org/web/20220626194537/https://spring.io/blog/2018/12/12/how-fast-is-spring)spring . io

概述了 Spring Boot 2.1 和 Spring 5.1 中最近围绕启动时间和堆使用的优化，加上一些提示**让你的应用启动和运行得更快**。

#### [**> >网飞 OSS 和 Spring Boot——来了一圈**](https://web.archive.org/web/20220626194537/https://medium.com/@NetflixTechBlog/netflix-oss-and-spring-boot-coming-full-circle-4855947713a0)【medium.com】

经过多年的内部基础设施建设，网飞完全接受了 Spring Boot。

#### [**> > Hibernate 技巧:如何将 DISTINCT 应用于你的 JPQL 而不是你的 SQL 查询**](https://web.archive.org/web/20220626194537/https://thoughts-on-java.org/hibernate-tips-apply-distinct-to-jpql-but-not-sql-query/)【thoughts-on-java.org】

快速浏览一下使用 Hibernate 的`QueryHints`到**使不同的查询更有效**。

#### [**> >如何将自定义 Hibernate 参数类型绑定到 JPA 查询**](https://web.archive.org/web/20220626194537/https://vladmihalcea.com/bind-custom-hibernate-parameter-type-jpa-query/)【vladmihalcea.com】

关于在 Hibernate 实体和查询中使用定制类型的**的精彩文章，以及 PostgreSQL 中的完整示例。非常酷。**

#### [**> >带协程的奇偶**](https://web.archive.org/web/20220626194537/https://blog.frankel.ch/even-odd-coroutines/) [ blog.frankel.ch

还有一篇很好的文章**比较了并发算法**的两种方法——一种使用 Kotlin 协程，另一种使用 Java 线程。

#### 同样值得一读:

*   #### [**> > Access strategies in JPA and Hibernate-which is better, field access or attribute access?**](https://web.archive.org/web/20220626194537/https://thoughts-on-java.org/access-strategies-in-jpa-and-hibernate/) 【 thoughts-on-java.org】

*   #### [**> > JDK 12 现处于下降阶段一**T4【邮件】open JDK . Java . net](https://web.archive.org/web/20220626194537/http://mail.openjdk.java.net/pipermail/jdk-dev/2018-December/002405.html)

#### 网络研讨会和演示:

*   #### [**> > Spring prompt: use Spring data R2DBC**](https://web.archive.org/web/20220626194537/https://spring.io/blog/2018/12/19/spring-tips-reactive-sql-data-access-with-spring-data-r2dbc) [ spring.io ] to access reactive SQL data

*   #### [**> > Leaping into Cotrim: How to make magic more magical**](https://web.archive.org/web/20220626194537/https://www.infoq.com/presentations/spring-kotlin) [infoq.com]

*   #### [**> > Genetic programming in the real world: a brief overview [infoq.com]**](https://web.archive.org/web/20220626194537/https://www.infoq.com/presentations/genetic-programming-overview)

*   #### [**> > Provide tools, robots and automation for better open source projects**](https://web.archive.org/web/20220626194537/https://blog.scottlogic.com/2018/12/14/automation-and-bots-for-open-source.html) [blog.scottlogic.com]

*   #### [**> > Buckets, funnels, monsters and cats or: How did we learn to like to extend applications to the cloud**](https://web.archive.org/web/20220626194537/https://www.infoq.com/presentations/migration-cloud-scalability-resilience) [infoq.com]

*   #### [**>现代消息与兔子 q、春天云和**](https://web.archive.org/web/20220626194537/https://www.infoq.com/presentations/rabbitmq-reactor-spring-cloud) 【infoq.com】

    反应堆
*   #### [**>网飞播放 API——一种进化架构**](https://web.archive.org/web/20220626194537/https://www.infoq.com/presentations/netflix-play-api)【infoq . com】T5

*   #### [**> > Large-scale payment of technical debt-migration @ stripe**](https://web.archive.org/web/20220626194537/https://www.infoq.com/presentations/stripe-technical-debt) [[infoq.com] T4

*   #### [**>生产中的 crdt**](https://web.archive.org/web/20220626194537/https://www.infoq.com/presentations/crdt-production)【infoq . com】T5

*   #### [**> > Build toughness in production migration**](https://web.archive.org/web/20220626194537/https://www.infoq.com/presentations/netflix-migration-resilience) [infoq.com]

*   #### [**> > Slack in volume-good, unexpected, and the road ahead**](https://web.archive.org/web/20220626194537/https://www.infoq.com/presentations/slack-scalability-2018) [infoq.com]

**升级时间:**

*   #### [**> > Hibernation ORM 5.4.0\. Finally released**](https://web.archive.org/web/20220626194537/http://in.relation.to/2018/12/12/hibernate-orm-540-final-out/)

*   #### [**> >休眠 OGM 5。4 .1 .最终发布**](https://web.archive.org/web/20220626194537/http://in.relation.to/2018/12/18/hibernate-ogm-5-4-1-Final-released/) 在。与的关系

*   #### [**> >春天 CredHub 2。0 .0 .rc1 发布**](https://web.archive.org/web/20220626194537/https://spring.io/blog/2018/12/14/spring-credhub-2-0-0-rc1-released) 春天。io

*   #### [**> > Chunyun Greenwich. RC1 Now available**](https://web.archive.org/web/20220626194537/https://spring.io/blog/2018/12/12/spring-cloud-greenwich-rc1-available-now) Spring.io

*   #### [**> > Lunar eclipse 4.10-a new noteworthy**](https://web.archive.org/web/20220626194537/https://www.eclipse.org/eclipse/news/4.10/) [eclipse.org]

## 2。技术和思考

#### [**>>FP vs OO 列表处理**](https://web.archive.org/web/20220626194537/http://blog.cleancoder.com/uncle-bob/2018/12/17/FPvsOO-List-processing.html)【blog.cleancoder.com】

一个有趣的 Clojure 函数算法的例子，带有**递归循环和尾调用优化**。

#### [**> >如何让跨职能运营成为一个团队的努力**](https://web.archive.org/web/20220626194537/https://www.infoq.com/articles/cross-functional-operations-team)【infoq.com】

一项对跨职能团队的研究显示，缺乏协作会让公司每天损失数千美元。下面来看看如何补救这种情况。

#### [**> >保持线路畅通**](https://web.archive.org/web/20220626194537/https://builttoadapt.io/tk-65faab4cb826)builttoadapt . io

一篇关于为什么沟通和友情对于一个分布式团队来说是必要的的精彩文章。

**同样值得一读:**

*   #### [**> > Git 2.20 brings improved workflow and performance**](https://web.archive.org/web/20220626194537/https://www.infoq.com/news/2018/12/git-2.20-released) [infoq.com]

*   #### [**> > ctop——管理和监控你的码头工人容器**](https://web.archive.org/web/20220626194537/https://blog.codecentric.de/en/2018/12/ctop-monitor-docker-containers/)博客。代码为中心。德

*   #### [**>剖析一个带有简单希腊字母的第 11 个函数的云的形成模板**](https://web.archive.org/web/20220626194537/https://advancedweb.hu/2018/12/18/lambda_cf_template/) [ 高级网页。胡

*   #### [**> > Clojure 的稳定性：经验教训**](https://web.archive.org/web/20220626194537/https://words.steveklabnik.com/why-is-clojure-so-stable)【words . steveklabnik . com】T5

*   #### [**> > Realize Netflix media database**](https://web.archive.org/web/20220626194537/https://medium.com/netflix-techblog/implementing-the-netflix-media-database-53b5a840b42a) [medium.com]

*   #### [**> > Testing will not make your software correct**](https://web.archive.org/web/20220626194537/https://codewithoutrules.com/2018/12/12/tests-are-not-enough/) [codewithoutrules.com]

*   #### [**>日食发展过程 2018**](https://web.archive.org/web/20220626194537/https://waynebeaton.wordpress.com/2018/12/19/eclipse-development-process-2018/)【waynebeaton . WordPress . com】

*   #### [**> > How to start contributing to TomEE or any open source project**](https://web.archive.org/web/20220626194537/https://www.tomitribe.com/blog/how-to-get-started-contributing-to-tomee-or-any-open-source-project/) [tomitribe.com]

## 3。漫画

本周我最喜欢的迪尔伯特。

#### [**> >与雄鹰一起翱翔**](https://web.archive.org/web/20220626194537/https://dilbert.com/strip/2018-12-19)【dilbert.com】

#### [**> >糖果荣誉系统**](https://web.archive.org/web/20220626194537/https://dilbert.com/strip/2018-12-17)【dilbert.com】

#### [**> >跟进**](https://web.archive.org/web/20220626194537/https://dilbert.com/strip/2018-12-16)【dilbert.com】

## 4。本周精选

#### [**> >扯淡网**](https://web.archive.org/web/20220626194537/https://pxlnv.com/blog/bullshit-web/)【pxlnv.com】

下一期[Java 周刊，第 261 期](/web/20220626194537/https://www.baeldung.com/java-weekly-261)上一期[Java 周刊，第 259 期](/web/20220626194537/https://www.baeldung.com/java-weekly-259)