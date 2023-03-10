# Java 周刊，第 450 期

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-weekly-450>

## 1。Spring 和 Java

[**> >连环垃圾收集器——啜爪哇**T3【inside.java](https://web.archive.org/web/20220812105641/https://inside.java/2022/08/08/sip063/)

**串行 GC** 上的速成班，当它被 JVM 工效学选择时，以及用于调优的通用配置参数。

[**> > MySQL 重写批处理语句**](https://web.archive.org/web/20220812105641/https://vladmihalcea.com/mysql-rewritebatchedstatements/)【vladmihalcea.com】

一个有趣的低级阅读，关于使用 JDBC/JPA/Hibernate 时`rewriteBatchedStatements`配置参数如何影响**批处理。**

#### 同样值得一读:

*   [**> >发布 GraalVM 企业代码编辑器和云壳**T3【oracle.com】和](https://web.archive.org/web/20220812105641/https://blogs.oracle.com/java/post/announcement-of-graalvm-enterprise-in-oracle-cloud-infrastructure-code-editor-and-cloud-shell)
*   [**> > GraalVM 22.2 新增库配置库**](https://web.archive.org/web/20220812105641/https://www.infoq.com/news/2022/08/graalvm-22-2/)【infoq.com】
*   [**> >春季数据 JPA 面试问答**](https://web.archive.org/web/20220812105641/https://www.jpa-buddy.com/blog/spring-data-jpa-interview-questions-and-answers/)【jpa-buddy.com】
*   [**>>Cash App 如何利用 Gradle Enterprise 的远程构建缓存加速本地构建**](https://web.archive.org/web/20220812105641/https://gradle.com/blog/how-cash-app-speeds-up-local-builds-with-gradle-enterprises-remote-build-cache/)【gradle.com】
*   [**>>JDBC vs JPA**](https://web.archive.org/web/20220812105641/https://springbootlearning.medium.com/jdbc-vs-jpa-623fe8258e8)【springbootlearning.medium.com
*   [**> >介绍 Micronaut 测试资源**](https://web.archive.org/web/20220812105641/https://melix.github.io/blog//2022/08/micronaut-test-resources.html)melix . github . io
*   [**> >在 Eclipse 集合中应该使用 ForEach 还是 InjectInto？**](https://web.archive.org/web/20220812105641/https://betterprogramming.pub/should-you-use-foreach-or-injectinto-in-eclipse-collections-5f7f791022e2)[better programming . pub]

**网络研讨会和演示:**

*   [**>>JavaOne 大会上的测序系列、纯度和更多内容——了解 Java 新闻广播# 31**](https://web.archive.org/web/20220812105641/https://inside.java/2022/08/11/insidejava-newscast-031/)[【inside.java】T4
*   [**> > #81: Quarkus:超音速、亚原子 Java(嘉宾:霍利·康明斯)**](https://web.archive.org/web/20220812105641/https://nurkiewicz.com/81)【nurkiewicz.com
*   [**> >一个精彩的播客:可观测性大师乔纳森·伊万诺夫谈 Spring Boot 可观测性的未来**](https://web.archive.org/web/20220812105641/https://spring.io/blog/2022/08/04/a-bootiful-podcast-observability-guru-jonatan-ivanov-on-the-future-of-observability-in-spring-boot)【spring . io
*   [**> >第 25 集“JavaOne 回来了！”**](https://web.archive.org/web/20220812105641/https://inside.java/2022/08/03/podcast-025/)【inside.java】

**升级时间:**

*   [**> >冬眠搜索 6.1.6 .最终发布**](https://web.archive.org/web/20220812105641/https://in.relation.to/2022/08/04/hibernate-search-6-1-6-Final/)
*   [**>>Hibernate Validator 6 . 2 . 4 . final、7.0.5.Final 和 8.0.0.CR2 发布**](https://web.archive.org/web/20220812105641/https://in.relation.to/2022/08/04/hibernate-validator-705-624-final-800-cr2-released/)in . relation . to
*   [**> >弹簧工具 4.15.2 发布**](https://web.archive.org/web/20220812105641/https://spring.io/blog/2022/08/04/spring-tools-4-15-2-released)Spring . io
*   [**> >阿帕奇骆驼 3 . 18 . 1**T3【github.com/apache】](https://web.archive.org/web/20220812105641/https://github.com/apache/camel/releases/tag/camel-3.18.1)
*   [**> > Micronaut 框架 3.6.0 发布！**](https://web.archive.org/web/20220812105641/https://micronaut.io/2022/08/04/micronaut-framework-3-6-0-released/) [ micronaut.io ]
*   [**>>Grails 框架 5 . 2 . 2**](https://web.archive.org/web/20220812105641/https://docs.grails.org/5.2.2/guide/index.html)【grails.org】
*   [**> >网飞指挥 v 3 . 10 . 3**T3【github.com/Netflix】](https://web.archive.org/web/20220812105641/https://github.com/Netflix/conductor/releases/tag/v3.10.3)
*   [**> > JHipster 发布 v 7 . 9 . 1&v 7 . 9 . 2**](https://web.archive.org/web/20220812105641/https://www.jhipster.tech/2022/07/31/jhipster-release-7.9.2.html)JHipster . tech
*   [**> >夸尔库斯 2.11.2.Final 发布！**](https://web.archive.org/web/20220812105641/https://github.com/quarkusio/quarkus/releases/tag/2.11.2.Final)【github.com/quarkusio】

## 2。技术&思考

[**> >回归基础:访问 Kubernetes pods**](https://web.archive.org/web/20220812105641/https://blog.frankel.ch/basics-access-kubernetes-pods/)[blog . frankel . ch]

在 K8S `– `节点端口、负载平衡器和入口中**向外界公开服务的不同方式**

[**>>Kubernetes 1.25**](https://web.archive.org/web/20220812105641/https://kubernetes.io/blog/2022/08/04/upcoming-changes-in-kubernetes-1-25/)Kubernetes . io中的主要变化

更多的 Kubernetes-`PodSecurityPolicy`、贬低 GlusterFS、清理 IPTable 链所有权`– `**K8S 1.25**中值得期待的东西。

**同样值得一读:**

*   [**> >虚拟生产——虚幻引擎**](https://web.archive.org/web/20220812105641/https://netflixtechblog.com/virtual-production-a-validation-framework-for-unreal-engine-aab780b2f8c8)【netflixtechblog.com】的验证框架
*   [**> >开放 Apis 数字时代的公共基础设施**](https://web.archive.org/web/20220812105641/https://techblog.bozho.net/open-apis-public-infrastructure-in-the-digital-age/)【techblog.bozho.net
*   [**> >一些 README.md Love: Markdown 支持改进**](https://web.archive.org/web/20220812105641/https://blog.jetbrains.com/idea/2022/08/markdown-support-improvements/)【blog.jetbrains.com】
*   [**> >简介树桩**](https://web.archive.org/web/20220812105641/https://www.petrikainulainen.net/programming/testing/introduction-to-stubs/)【petrikainulainen.net】

## 3。漫画

本周我最喜欢的迪尔伯特。

[**> >把它塞回盒子里**](https://web.archive.org/web/20220812105641/https://dilbert.com/strip/2022-08-11)【dilbert.com】

[**> >需要序号**](https://web.archive.org/web/20220812105641/https://dilbert.com/strip/2022-08-10)【dilbert.com】

[**> >难返品**](https://web.archive.org/web/20220812105641/https://dilbert.com/strip/2022-08-08)【dilbert.com】

## 4。本周精选

**[> >不需要微服务](https://web.archive.org/web/20220812105641/https://itnext.io/you-dont-need-microservices-2ad8508b9e27)**it next . io