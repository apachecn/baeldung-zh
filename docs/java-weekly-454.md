# Java 周刊，第 454 期

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-weekly-454>

## 1. **Spring 和 Java**

**[> >通过反编译理解 JIT 优化](https://web.archive.org/web/20220911115357/https://www.infoq.com/presentations/jit-optimize-decompile-shopify/)**【infoq.com】

演示如何为优化的 Java 代码开发一个**伪代码反编译器，以及它如何帮助您理解 JIT 的内部机制。**

[**>>Project Loom for Java 到底是个什么鬼？**](https://web.archive.org/web/20220911115357/https://foojay.io/today/what-the-heck-is-project-loom-for-java/) [ foojay.io

Java 并发性的一点历史，以及 project Loom 将如何改变我们开发高吞吐量并发应用程序的方式。

[**> >退而求其次取一个春季数据 JPA DTO 投影**T3【blog.jooq.org】](https://web.archive.org/web/20220911115357/https://blog.jooq.org/the-second-best-way-to-fetch-a-spring-data-jpa-dto-projection/)

以及如何使用 **multiset 值构造函数和聚合函数**将数据库实体映射到 jOOQ 中的 dto。好东西。

#### 同样值得一读:

*   [**> >介绍新的 Redis API——如何用 Redis 缓存？**](https://web.archive.org/web/20220911115357/https://quarkus.io/blog/redis-api-intro/) [ 夸库斯. io ]
*   [**>>ska ffold for Local Java App 开发**](https://web.archive.org/web/20220911115357/http://www.java-allandsundry.com/2022/09/skaffold-for-local-java-app-development.html)【java-allandsundry.com】
*   [**> >微软致力于 Java 开发者的成功**](https://web.archive.org/web/20220911115357/https://spring.io/blog/2022/08/30/microsoft-is-committed-to-the-success-of-java-developers)spring . io

**网络研讨会和演示:**

*   [**> >字符串模板、JavaFX 19、反序列化，以及 JavaOne-Inside Java news cast # 32**](https://web.archive.org/web/20220911115357/https://inside.java/2022/08/23/insidejava-newscast-032/)[【inside.java】T4]
*   [**> > #206 Java 19:瞬间百万线程**](https://web.archive.org/web/20220911115357/https://airhacks.fm/#episode_206) [ airhacks.fm ]
*   [**> >一个精彩的播客:Kris De Volder 博士谈 Spring 工具、VS 代码等等**](https://web.archive.org/web/20220911115357/https://spring.io/blog/2022/09/01/a-bootiful-podcast-dr-kris-de-volder-on-spring-tools-vs-code-and-so-much-more) [ spring.io

**升级时间:**

*   [**> >冬眠 ORM 5.6.11 .最终发布**](https://web.archive.org/web/20220911115357/https://in.relation.to/2022/08/30/hibernate-orm-5611-final/)
*   [**> > Micronaut 框架 3.6.2 发布！**](https://web.archive.org/web/20220911115357/https://micronaut.io/2022/09/04/micronaut-framework-3-6-2-released/) [ micronaut.io ]
*   [**> >夸尔库斯 2.12.0 .最终发布**](https://web.archive.org/web/20220911115357/https://quarkus.io/blog/quarkus-2-12-0-final-released/) [ 夸尔库斯. io
*   [**> > Pi4J V2.2.0 发布**](https://web.archive.org/web/20220911115357/https://foojay.io/today/pi4j-v2-2-0-released/)foojay . io
*   [**> >网飞指挥 v 3 . 11 . 1**T3【github.com/Netflix】](https://web.archive.org/web/20220911115357/https://github.com/Netflix/conductor/releases/tag/v3.11.1)
*   [**> >阿帕奇骆驼 3 . 18 . 2**T3【github.com/apache】](https://web.archive.org/web/20220911115357/https://github.com/apache/camel/releases/tag/camel-3.18.2)
*   [**> > Elasticsearch 版本 8 . 4 . 1**](https://web.archive.org/web/20220911115357/https://www.elastic.co/guide/en/elasticsearch/reference/8.4/release-notes-8.4.1.html)【elastic.co】
*   [**> > JHipster 版本 7 . 9 . 3**](https://web.archive.org/web/20220911115357/https://www.jhipster.tech/2022/09/02/jhipster-release-7.9.3.html)JHipster . tech

## 2。技术&思考

[**> >请求批**](https://web.archive.org/web/20220911115357/https://martinfowler.com/articles/patterns-of-distributed-systems/request-batch.html)【martinfowler.com】

**合并多个请求**以优化利用网络容量来处理它们，从而提高吞吐量。

[**> >介绍 COSI:使用 Kubernetes APIs 的对象存储管理**](https://web.archive.org/web/20220911115357/https://kubernetes.io/blog/2022/09/02/cosi-kubernetes-object-storage-management/) [ kubernetes.io ]

满足**容器对象存储接口**:Kubernetes 1.25 中提供和消费对象存储的标准。

**同样值得一读:**

*   [**> >一瞥 Kubernetes 网关 API**](https://web.archive.org/web/20220911115357/https://blog.frankel.ch/kubernetes-gateway-api/)[blog . frankel . ch]
*   [**> >为什么无功而返是削减你每月云账单的秘诀！**](https://web.archive.org/web/20220911115357/https://springbootlearning.medium.com/why-reactive-streams-are-the-secret-to-cutting-your-monthly-cloud-bill-c75cf2f328ce)【springbootlearning.medium.com】
*   [**>>【立方链不是 API】**](https://web.archive.org/web/20220911115357/https://kubernetes.io/blog/2022/09/07/iptables-chains-not-api/)【立方链】我
***   [**> >迁移单片到实践中的微服务**](https://web.archive.org/web/20220911115357/https://foojay.io/today/migrating-monoliths-to-microservices-in-practice/)foojay . io*   [**> >分布式系统是可知的:对开发者来说是不可能的事**](https://web.archive.org/web/20220911115357/https://www.infoq.com/news/2022/09/distributed-system-knowable/)【infoq.com】*   [**> >远程工作提示**](https://web.archive.org/web/20220911115357/http://blog.code-cop.org/2022/09/tips-for-remote-work.html)【blog.code-cop.org】**

 **## 3。漫画

本周我最喜欢的迪尔伯特。

[**> >做不可能的事**](https://web.archive.org/web/20220911115357/https://dilbert.com/strip/2022-09-08)**dilbert.com**

 **[**> >道格伯特咨询公司更名为**](https://web.archive.org/web/20220911115357/https://dilbert.com/strip/2022-09-05)**dilbert.com**

 **[**> >荒谬绝对的家伙**](https://web.archive.org/web/20220911115357/https://dilbert.com/strip/2022-09-04)**dilbert.com**

 **## 4。本周精选

vFunction 终于发布了他们的技术债务平台的云托管版本:

**[> >考核枢纽快车](/web/20220911115357/https://www.baeldung.com/vfunction-assessment-hub-express-d3u8)**

你可以直接在这里注册，下载这个工具，用你的项目设置它，然后快速进入你的技术债务的要点。

**«** Previous[Java Weekly, Issue 453](/web/20220911115357/https://www.baeldung.com/java-weekly-453)********