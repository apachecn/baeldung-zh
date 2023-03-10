# Java 周刊，第 436 期

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-weekly-436>

## 1。Spring 和 Java

[**>>19:425:虚拟线程**](https://web.archive.org/web/20220813065656/https://mail.openjdk.java.net/pipermail/jdk-dev/2022-April/006530.html)【openjdk.java.net】JEP 拟将目标锁定 JDK

最后， **Project Loom 将在 Java 19** 中作为预览功能提供——一个可靠的、新的 Java 并发模型！

[**>>ZGC | JDK 最新动态 18**](https://web.archive.org/web/20220813065656/https://malloc.se/blog/zgc-jdk18) [ malloc.se

字符串重复数据删除支持，不再卸载类，Linux/PowerPC，以及 Java 18 中 ZGC 的更多**增强。**

[**> >为什么写一个空的 finalize()方法？**](https://web.archive.org/web/20220813065656/https://stuartmarks.wordpress.com/2022/04/27/why-write-an-empty-finalize-method/)【stuartmarks.wordpress.com】

另一个奇怪但有时有用的终结怪癖——empty*finalize()*方法**对类实例**禁用终结。

#### 同样值得一读:

*   [**>**](https://web.archive.org/web/20220813065656/https://stuartmarks.wordpress.com/2022/04/27/why-write-an-empty-finalize-method/)**[>云原生 Java 与 Micronaut 框架](https://web.archive.org/web/20220813065656/https://www.infoq.com/articles/native-java-micronaut)**infoq.com
*   **[> >多对多关联的同步方式](https://web.archive.org/web/20220813065656/https://www.jpa-buddy.com/blog/synchronization-methods-for-many-to-many-associations/)**【jpa-buddy.com】
*   [**> >有没有想过在 Spring Data JPA 中重写一个查询？**](https://web.archive.org/web/20220813065656/https://spring.io/blog/2022/05/02/ever-wanted-to-rewrite-a-query-in-spring-data-jpa)spring . io
*   [**> >嵌套在 jOOQ 中的事务**](https://web.archive.org/web/20220813065656/https://blog.jooq.org/nested-transactions-in-jooq/)【blog.jooq.org】
*   [**> >开始使用阿帕奇卡夫卡**T3【infoq.com进行 Quarkus 反应式消息传递](https://web.archive.org/web/20220813065656/https://www.infoq.com/articles/data-with-quarkus-kafka/)
*   [**> >用 Java 调用谷歌云服务**](https://web.archive.org/web/20220813065656/http://www.java-allandsundry.com/2022/04/calling-google-cloud-services-injava.html)【java-allandsundry.com】

**网络研讨会和演示:**

*   [**> >精彩播客:西蒙·里特，Java 冠军，Azul**](https://web.archive.org/web/20220813065656/https://spring.io/blog/2022/04/28/a-bootiful-podcast-simon-ritter-java-champion-and-deputy-cto-at-azul)[spring . io]副 CTO
*   [**> >现代 Java 交付:Java 17、18 和开放 JDK**T3【redmonk.com】和](https://web.archive.org/web/20220813065656/https://redmonk.com/jgovernor/2022/04/28/modern-java-delivery-java-17-18-and-open-jdk/)
*   [**> >垃圾收集中 JDK 8 到 JDK 18:10 个版本，2000+增强**](https://web.archive.org/web/20220813065656/https://inside.java/2022/05/02/odl-jdk8-to-jdk18-gc/)【inside.java】
*   [**> >美文教程——好听的&轻松的**](https://web.archive.org/web/20220813065656/https://www.youtube.com/watch?v=Xatr8AZLOsE)【youtube.com】
*   [**> >用 Java 上 Neo4j 的 8 个技巧和经验**](https://web.archive.org/web/20220813065656/https://blog.sebastian-daschner.com/entries/tips-experiences-on-neo4j-ogm)【blog.sebastian-daschner.com】

**升级时间:**

*   [**> >春云 2021.0.2 已发布**](https://web.archive.org/web/20220813065656/https://spring.io/blog/2022/04/26/spring-cloud-2021-0-2-has-been-released)Spring . io
*   [**> >网飞指挥 v 3 . 8 . 0-RC . 1**T3【github.com/Netflix】](https://web.archive.org/web/20220813065656/https://github.com/Netflix/conductor/releases)
*   [**> >阿帕奇骆驼 3 . 11 . 7**](https://web.archive.org/web/20220813065656/https://camel.apache.org/blog/2022/05/RELEASE-3.11.7/)【camel.apache.org】
*   **[>>Micronaut 3 . 4 . 3](https://web.archive.org/web/20220813065656/https://micronaut.io/2022/04/29/micronaut-3-4-3-released/)**Micronaut . io

## 2。技术&思考

[**> > Kubernetes 1.24:观星者**](https://web.archive.org/web/20220813065656/https://kubernetes.io/blog/2022/05/03/kubernetes-1-24-release-announcement/) [ kubernetes.io

Kubernetes 的一个`interstellar` **版本:Dockershim 移除、gRPC 探测、存储容量跟踪，以及 1.24 版本中的更多功能。**

**[> >工作开发者对称加密入门](https://web.archive.org/web/20220813065656/https://www.geekabyte.io/2022/05/introduction-to-symmetric-encryption.html)** [ geekabyte.io ]

对称加密实用指南:定义、对称性、ECB 与 CBC、AES、填充和初始化向量。

**同样值得一读:**

*   [**> >从****GitHub**](https://web.archive.org/web/20220813065656/https://blog.frankel.ch/authenticate-google-cloud-github/)[frankel . ch]安全认证到 Google Cloud
*   **[> > Dockershim:历史脉络](https://web.archive.org/web/20220813065656/https://kubernetes.io/blog/2022/05/03/dockershim-historical-context/)** [ kubernetes.io ]
*   **[> >如何写一个简单的 systemd 定时器](https://web.archive.org/web/20220813065656/https://advancedweb.hu/how-to-write-a-simple-systemd-timer/)**【advanced web . Hu】
*   **[> > Github 动作测试一个完整的 Tekton CI 安装](https://web.archive.org/web/20220813065656/https://blog.codecentric.de/en/2022/04/github-actions-test-a-full-tekton-ci-installation/)**[blog . code centric . de]
*   [**> >敏捷书友:重构(与马丁·福勒)**](https://web.archive.org/web/20220813065656/https://www.jamesshore.com/v2/books/aoad2/book_club/refactoring)【jamesshore.com】
*   [**> > SoundCloud 编年史公开 API 绞杀者**](https://web.archive.org/web/20220813065656/https://www.infoq.com/news/2022/05/soundcloud-end-strangler/)【infoq.com

## 3。漫画

本周我最喜欢的迪尔伯特。

**[> >确定任何你想要的方式](https://web.archive.org/web/20220813065656/https://dilbert.com/strip/2022-05-03)**【dilbert.com】

**[> >周日幻术管理](https://web.archive.org/web/20220813065656/https://dilbert.com/strip/2022-05-01)**【dilbert.com】

**[> >伦理杀艾](https://web.archive.org/web/20220813065656/https://dilbert.com/strip/2022-04-26)**【dilbert.com】

## 4。本周精选

### [>探索动物群](/web/20220813065656/https://www.baeldung.com/fauna-jw-ww3o)

Next **»**[Java Weekly, Issue 437](/web/20220813065656/https://www.baeldung.com/java-weekly-437)**«** Previous[Java Weekly, Issue 435](/web/20220813065656/https://www.baeldung.com/java-weekly-435)