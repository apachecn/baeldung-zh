# Java 周刊，第 452 期

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-weekly-452>

## 1. **Spring 和 Java**

[**> >在 G1 并发阅卷**](https://web.archive.org/web/20220826133735/https://tschatzl.github.io/2022/08/04/concurrent-marking.html)[tschatzl . github . io]

G1 如何使用标记位图存储活跃度信息`– ` **的详细故事降低了本机内存消耗！**

[**> > G1 前置障实现**](https://web.archive.org/web/20220826133735/https://albertnetymk.github.io/2022/07/22/g1_barrier/)[albertnetymk . github . io]

深入了解**G1 如何维护其不变量之一**:预屏障、开始时的快照和 OpenJDK 实现。复杂但值得一读。

[**> >局部类改进——啜 Java**](https://web.archive.org/web/20220826133735/https://inside.java/2022/08/22/sip065/)【inside.java

快速回顾一下最近的一些 Java 增强:类型推断、**扩展的本地类和本地/内部类的静态成员**。

#### 同样值得一读:

*   [**> >如何在 Maven**](https://web.archive.org/web/20220826133735/https://vladmihalcea.com/different-java-main-test-maven/)【vladmihalcea.com】中使用不同 Java 版本进行 main 和 test
*   [**> >测试特征标志**](https://web.archive.org/web/20220826133735/https://reflectoring.io/testing-feature-flags/)reflector ing . io
*   [**> >用 Kotlin Dataframe 处理数据预览**](https://web.archive.org/web/20220826133735/https://www.infoq.com/news/2022/08/kotlin-dataframe-preview/)【infoq.com】
*   [**> >如何用 jOOQ**](https://web.archive.org/web/20220826133735/https://blog.jooq.org/how-to-integration-test-stored-procedures-with-jooq/)【blog.jooq.org】集成测试存储过程
*   [**> >使用 H2 作为测试数据库产品与 jOOQ**](https://web.archive.org/web/20220826133735/https://blog.jooq.org/using-h2-as-a-test-database-product/)【blog.jooq.org】
*   [**> >探索 CVE-2022-33980:阿帕奇公地配置 RCE 漏洞**](https://web.archive.org/web/20220826133735/https://foojay.io/today/exploring-cve-2022-33980-the-apache-commons-configuration-rce-vulnerability/) [ foojay.io
*   [**> > Spring 授权服务器 1.0 计划 2022 年 11 月**](https://web.archive.org/web/20220826133735/https://www.infoq.com/news/2022/08/spring-authorization-server-1-0/)【infoq.com】
*   [**> >令人困惑的 Java 字符串**](https://web.archive.org/web/20220826133735/https://foojay.io/today/confusing-java-strings/)foojay . io
*   [**> >寻找月食系列终极问题的答案**](https://web.archive.org/web/20220826133735/https://medium.com/javarevisited/finding-answers-to-the-ultimate-questions-about-eclipse-collections-a0f9953b8fa)【medium.com】

**网络研讨会和演示:**

*   [**> >字符串模板、JavaFX 19、反序列化，以及 JavaOne-Inside Java news cast # 32**](https://web.archive.org/web/20220826133735/https://inside.java/2022/08/23/insidejava-newscast-032/)[【inside.java】T4]
*   [**> >一个精彩的播客:Flowable 创始人 Joram Barrez 关于工作流、业务流程管理和更多内容的播客**](https://web.archive.org/web/20220826133735/https://spring.io/blog/2022/08/18/a-bootiful-podcast-flowable-founder-joram-barrez-on-a-bootiful-podcast-on-workflow-business-process-management-and-more) [ spring.io
*   [**>>Java 中的国际化——Java 的 Sip**](https://web.archive.org/web/20220826133735/https://inside.java/2022/08/17/sip064/)【inside.java

**升级时间:**

*   **[>>Spring Boot 2 . 6 . 11](https://web.archive.org/web/20220826133735/https://spring.io/blog/2022/08/17/spring-boot-2-6-11-available-now)[和现在可用的 2 . 7 . 3](https://web.archive.org/web/20220826133735/https://spring.io/blog/2022/08/18/spring-boot-2-7-3-available-now)**spring . io
*   [**>>Spring Shell 2 . 1 . 1 现已推出**](https://web.archive.org/web/20220826133735/https://spring.io/blog/2022/08/19/spring-shell-2-1-1-is-now-available)Spring . io
*   [**> > Payara 平台社区 5 . 2022 . 3**](https://web.archive.org/web/20220826133735/https://docs.payara.fish/community/docs/Release%20Notes/Release%20Notes%205.2022.3.html)Payara . fish
*   [**> >阿帕奇骆驼 3.14.5 发布**](https://web.archive.org/web/20220826133735/https://camel.apache.org/blog/2022/08/RELEASE-3.14.5/)【camel.apache.org】
*   [**> >甲骨文 heli don 3 . 0 . 1**](https://web.archive.org/web/20220826133735/https://github.com/oracle/helidon/releases/tag/3.0.1)【github.com/oracle】
*   [**> > Micronaut 框架 3.6.1 发布！**](https://web.archive.org/web/20220826133735/https://micronaut.io/2022/08/19/micronaut-framework-3-6-1-released/) [ micronaut.io ]
*   [**> >网飞指挥 v 3 . 10 . 7**T3【github.com/Netflix】T4](https://web.archive.org/web/20220826133735/https://github.com/Netflix/conductor/releases/tag/v3.10.7)
*   [**> >夸库斯 2 . 12 . 0 . cr1**](https://web.archive.org/web/20220826133735/https://github.com/quarkusio/quarkus/releases/tag/2.12.0.CR1)【github.com/quarkusio】

## 2。技术&思考

[**> > CloudWatch on AWS:如何应对高安全性需求**](https://web.archive.org/web/20220826133735/https://blog.codecentric.de/en/2022/08/aws-cloudwatch-security-requirements/)[blog . code centric . de]

将 CloudWatch 日志导出到 S3 的实用指南**，以可定制的频率使用密钥轮换进行加密**！

[**>>Kubernetes v 1.25:Pod 安全准入控制器处于稳定状态**](https://web.archive.org/web/20220826133735/https://kubernetes.io/blog/2022/08/25/pod-security-admission-stable/) [ kubernetes.io

通过简单地将标签添加到名称空间`– ` **来方便地实施预定义的 Pod 安全标准符合 Pod 安全准入控制器**。

**同样值得一读:**

*   [**> > #83:实时竞价:在线跟踪如何帮助投放广告**](https://web.archive.org/web/20220826133735/https://nurkiewicz.com/83)【nurkiewicz.com】
*   [**> >在 DynamoDB**](https://web.archive.org/web/20220826133735/https://advancedweb.hu/modeling-common-graphql-patterns-in-dynamodb/)[advanced web . Hu]中建模常见的 GraphQL 模式
*   [**> >固定分区**](https://web.archive.org/web/20220826133735/https://martinfowler.com/articles/patterns-of-distributed-systems/fixed-partitions.html)【martinfowler.com】固定分区
*   [**> >突如其来的领袖**](https://web.archive.org/web/20220826133735/https://martinfowler.com/articles/patterns-of-distributed-systems/emergent-leader.html)【martinfowler.com】
*   [**> >翻新，一个塌实的机器人替代**](https://web.archive.org/web/20220826133735/https://blog.frankel.ch/renovate-alternative-dependabot/)blog . frankel . ch
*   [**> >有效桥接 devo PS–R&D；不牺牲可靠性的差距**T3foojay . io](https://web.archive.org/web/20220826133735/https://foojay.io/today/effectively-bridging-the-devops-rd-gap-without-sacrificing-reliability/)

## 3。漫画

本周我最喜欢的迪尔伯特。

[**> >降低项目智商**](https://web.archive.org/web/20220826133735/https://dilbert.com/strip/2022-08-25)【dilbert.com】

[**> >沃利有三份工作**](https://web.archive.org/web/20220826133735/https://dilbert.com/strip/2022-08-22)【dilbert.com】

[**> >到最后**](https://web.archive.org/web/20220826133735/https://dilbert.com/strip/2022-08-21)【dilbert.com】

## 4。本周精选

**[> >区块链](https://web.archive.org/web/20220826133735/https://calpaterson.com/blockchain.html)**没有那么多用处

**«** Previous[Java Weekly, Issue 451](/web/20220826133735/https://www.baeldung.com/java-weekly-451)