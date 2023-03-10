# Java 周刊，第 455 期

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-weekly-455>

## 1. **Spring 和 Java**

[**> > Helidon Níma —虚拟线程上的 heli don**](https://web.archive.org/web/20220915125407/https://medium.com/helidon/helidon-n%C3%ADma-helidon-on-virtual-threads-130bb2ea2088)【medium.com

**Helidon 遇见 Loom** `– `一个使用廉价虚拟线程处理 HTTP 请求的并发模型。

[**> > Java 很快，如果不创建很多对象**](https://web.archive.org/web/20220915125407/http://blog.vanillajava.blog/2022/09/java-is-very-fast-if-you-dont-create.html)[blog . vanilla Java . blog]

**避免对象分配和 GC 暂停**，同时通过 TCP 传递 40 亿个事件/秒。让我们看看这怎么可能。

[**>>JEP 429:Extent-Java 中促进不变性的局部变量**](https://web.archive.org/web/20220915125407/https://www.infoq.com/news/2022/09/extent-local-variables-java/)【infoq.com】

和一种轻量级方法来使用范围局部变量在线程之间共享不可变数据。好东西。

#### 同样值得一读:

*   [**> > JDK 19、JDK 20:目前已知的**](https://web.archive.org/web/20220915125407/https://www.infoq.com/news/2022/09/java-19-so-far/)【infoq.com】
*   [**> >使用 jOOQ 的隐式连接从内部连接..关于条款**](https://web.archive.org/web/20220915125407/https://blog.jooq.org/using-jooqs-implicit-join-from-within-the-join-on-clause/)【blog.jooq.org】条款
*   [**>>Lucene 简介|如何构建自己的搜索引擎**](https://web.archive.org/web/20220915125407/https://advancedweb.hu/intro-to-lucene/)[advanced web . Hu]
*   [**> >一个 Java 17 原生内存泄露的故事**](https://web.archive.org/web/20220915125407/https://foojay.io/today/the-story-of-a-java-17-native-memory-leak/)foojay . io
*   [**> >春季数据 findAll 反模式**](https://web.archive.org/web/20220915125407/https://vladmihalcea.com/spring-data-findall-anti-pattern/)【vladmihalcea.com】
*   [**> >回顾:面向 Java 开发者的 Devops 工具**](https://web.archive.org/web/20220915125407/https://info.michael-simons.eu/2022/09/13/review-devops-tools-for-java-developers/)[info . Michael-Simons . eu]
*   [**> >用反向外壳攻击控制你的服务器**](https://web.archive.org/web/20220915125407/https://foojay.io/today/controlling-your-server-with-a-reverse-shell-attack/)foojay . io
*   [**>>**](https://web.archive.org/web/20220915125407/https://andresalmiray.com/a-pom-by-any-other-name/)【andresalmiray.com】

**网络研讨会和演示:**

*   [**> >爪哇未来**](https://web.archive.org/web/20220915125407/https://inside.java/2022/09/14/java-to-the-future/)【inside.java
*   [**> >行动中的 Java 19——Java 新闻播报# 33**](https://web.archive.org/web/20220915125407/https://inside.java/2022/09/08/insidejava-newscast-033/)【inside.java
*   [**> >如何用 Java 构建命令行文本编辑器(第一部分)**](https://web.archive.org/web/20220915125407/https://youtu.be/kT4JYQi9w4w)【youtube.com】
*   [**> >用夸库斯**](https://web.archive.org/web/20220915125407/https://blog.sebastian-daschner.com/entries/uploading-files-quarkus)【blog.sebastian-daschner.com上传文件
*   [**> >一个精彩的播客:Hashicorp 的 Rosemary Wang 用 hashi corp Vault**](https://web.archive.org/web/20220915125407/https://spring.io/blog/2022/09/08/a-bootiful-podcast-hashicorp-s-rosemary-wang-on-securing-the-intersection-of-apps-and-ops-with-hashicorp-vault)[spring . io]保护应用和运营的交集
*   [**> > OpenJDK:魔法发生的地方**](https://web.archive.org/web/20220915125407/https://inside.java/2022/09/12/change-the-future-of-java/)【inside.java

**升级时间:**

*   [**> >冬眠 ORM 6.1.3 .最终发布**](https://web.archive.org/web/20220915125407/https://in.relation.to/2022/09/08/hibernate-orm-613-final/)
*   [**> >春云 2021.0.4(代号禧年)已经上映**](https://web.archive.org/web/20220915125407/https://spring.io/blog/2022/09/07/spring-cloud-2021-0-4-codename-jubilee-has-been-released)Spring . io
*   [**> >甲骨文/海利登 2 . 5 . 3**](https://web.archive.org/web/20220915125407/https://newreleases.io/project/github/oracle/helidon/release/2.5.3)new releases . io
*   [**> > Micronaut 框架 3.6.3 发布！**](https://web.archive.org/web/20220915125407/https://micronaut.io/2022/09/09/micronaut-framework-3-6-3/) [ micronaut.io ]
*   [**> >网飞指挥 v 3 . 11 . 2**T3【github.com/Netflix】](https://web.archive.org/web/20220915125407/https://github.com/Netflix/conductor/releases)
*   [**> >夸尔库斯 2.12.1 .最终发布**](https://web.archive.org/web/20220915125407/https://quarkus.io/blog/quarkus-2-12-1-final-released/) [ 夸尔库斯. io

## 2。技术&思考

[**>>YugabyteDB**【vladmihalcea.com](https://web.archive.org/web/20220915125407/https://vladmihalcea.com/yugabytedb/)

**认识 yugabytdb**:兼容 PostgreSQL 的开源分布式 SQL 数据库。

**同样值得一读:**

*   [**> >事件版本控制与马腾**](https://web.archive.org/web/20220915125407/https://event-driven.io/en/event_versioning_with_marten/)事件驱动. io
*   [**> >平台工程——入门**](https://web.archive.org/web/20220915125407/https://blog.codecentric.de/en/2022/09/platform-engineering-a-primer/)[blog . codecentric . de]
*   [**> >请求等候名单**](https://web.archive.org/web/20220915125407/https://martinfowler.com/articles/patterns-of-distributed-systems/request-waiting-list.html)【martinfowler.com】
*   [**>>Python 依赖管理的迷宫**](https://web.archive.org/web/20220915125407/https://blog.frankel.ch/maze-python-dependency-management/)blog . frankel . ch
*   [**> >新系列:用机器学习创造媒体**](https://web.archive.org/web/20220915125407/https://netflixtechblog.medium.com/new-series-creating-media-with-machine-learning-5067ac110bcd)【netflixtechblog.medium.com】
*   [**> >你不能这么做:用类型推理抽象出 Rust 中的所有权(而 GATs 无济于事)**](https://web.archive.org/web/20220915125407/https://lucumr.pocoo.org/2022/9/8/abstracting-over-ownership/)【lucumr.pocoo.org】

## 3。漫画

本周我最喜欢的迪尔伯特。

[**> >超出预期**](https://web.archive.org/web/20220915125407/https://dilbert.com/strip/2022-09-11)**dilbert.com**

 **[**> >最后一次重构**](https://web.archive.org/web/20220915125407/https://dilbert.com/strip/2022-09-10)**dilbert.com**

 **[**> >做不可能的事**](https://web.archive.org/web/20220915125407/https://dilbert.com/strip/2022-09-08)**dilbert.com**

 **## 4。本周精选

最后，上周我谈到了来自 vFunction 的新托管的**技术债务平台**:

**[> >考核枢纽快车](/web/20220915125407/https://www.baeldung.com/vfunction-assessment-hub-express-d3u8)**

简单地说，这是一个了解你的技术债务实际上是什么样子的可靠方法。

**«** Previous[Java Weekly, Issue 454](/web/20220915125407/https://www.baeldung.com/java-weekly-454)******