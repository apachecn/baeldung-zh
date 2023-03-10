# Java 周刊，第 392 期

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-weekly-392>

## 1。Spring 和 Java

[**> >内部的 JDK 元素强烈地封装在 JDK 17 号**T3【infoq.com】号号里](https://web.archive.org/web/20220626113202/https://www.infoq.com/news/2021/06/internals-encapsulated-jdk17/)

**从 Java 17 开始，非法访问内部 API**不再是一个选项——必须知道我们是否计划使用 Java 17！

[**> >冬眠物理命名策略**](https://web.archive.org/web/20220626113202/https://vladmihalcea.com/hibernate-physical-naming-strategy/)【vladmihalcea.com】

让我们看看 **Hibernate 5 如何将实体属性映射到数据库标识符**——了解事情如何在幕后工作总是一个好主意！

#### 同样值得一读:

*   [**> >用 ElastiCache 缓存 Redis 和 Spring Cloud AWS**](https://web.archive.org/web/20220626113202/https://reflectoring.io/spring-cloud-aws-redis/)[reflector ing . io]
*   [**> > Hibernate 的查询计划缓存——如何工作以及如何调优**](https://web.archive.org/web/20220626113202/https://thorben-janssen.com/hibernate-query-plan-cache/)【thorben-janssen.com】
*   [**> > Scala 3 大修语言，为开发者带来更好的体验**](https://web.archive.org/web/20220626113202/https://www.infoq.com/news/2021/06/scala-3-overhaul/)【infoq.com】
*   [**> >赶紧用 Jbang 试出 jOOQ！**](https://web.archive.org/web/20220626113202/https://blog.jooq.org/2021/06/24/quickly-trying-out-jooq-with-jbang/)【blog.jooq.org】
*   [**> >用亚马逊 SES 和春云 AWS 发送邮件**](https://web.archive.org/web/20220626113202/https://reflectoring.io/spring-cloud-aws-ses/)[reflector ing . io]
*   [**> >科特林:带适配器的类型转换**](https://web.archive.org/web/20220626113202/https://www.mscharhag.com/kotlin/type-conversion-with-adapters)【mscharhag.com】

**网络研讨会和演示:**

*   [**> >第十八集《Java 稳步迈向强封装》与艾伦·贝特曼**](https://web.archive.org/web/20220626113202/https://inside.java/2021/06/29/podcast-018/)【inside.java
*   [**> >一个精彩的播客:webassembly，IoT，数据科学，还有 Java 大师布莱恩·斯莱顿**](https://web.archive.org/web/20220626113202/https://spring.io/blog/2021/06/24/a-bootiful-podcast-webassembly-iot-data-science-and-java-guru-brian-sletten) [ spring.io
*   [**> >春天提示:Kubernetes 原生 Java**](https://web.archive.org/web/20220626113202/https://spring.io/blog/2021/06/23/spring-tips-kubernetes-native-java)【Spring . io】
*   [**>>【jep 咖啡# 1】**](https://web.archive.org/web/20220626113202/https://inside.java/2021/06/23/jepcafe/)[inside . Java

**升级时间:**

*   [**> > Kotlin 1.5.20 发布！**](https://web.archive.org/web/20220626113202/https://blog.jetbrains.com/kotlin/2021/06/kotlin-1-5-20-released/)【blog.jetbrains.com】
*   **[>>Spring Boot 2 . 4 . 8](https://web.archive.org/web/20220626113202/https://spring.io/blog/2021/06/24/spring-boot-2-4-8-is-now-available)[和 2.5.2 现已上市](https://web.archive.org/web/20220626113202/https://spring.io/blog/2021/06/24/spring-boot-2-5-2-is-now-available)**spring . io
*   [**>>IntelliJ IDEA 2021 . 1 . 3 可用**](https://web.archive.org/web/20220626113202/https://blog.jetbrains.com/idea/2021/06/intellij-idea-2021-1-3/)【blog.jetbrains.com】
*   [**>>Spring Security 5 . 5 . 1、5.4.7、5.3.10 和 5.2.11 发布**](https://web.archive.org/web/20220626113202/https://spring.io/blog/2021/06/22/spring-security-5-5-1-5-4-7-5-3-10-and-5-2-11-released)Spring . io
*   [**> >冬眠 ORM 5.5.3 .最终发布**](https://web.archive.org/web/20220626113202/https://in.relation.to/2021/06/23/hibernate-orm-553-release/)in . relation . io
*   [**>>Spring Native 0 . 10 . 0 现已可用**](https://web.archive.org/web/20220626113202/https://spring.io/blog/2021/06/14/spring-native-0-10-0-available-now)Spring . io
*   [**> > CVE 报道发表为春**](https://web.archive.org/web/20220626113202/https://spring.io/blog/2021/06/28/cve-report-published-for-spring-security)Spring . io
*   [**> >春季数据 2021.0.2 和 2020.0.10 发布**](https://web.archive.org/web/20220626113202/https://spring.io/blog/2021/06/22/spring-data-2021-0-2-and-2020-0-10-released)Spring . io
*   [**>>Spring Integration Zip 2 . 0 . 0 可用**](https://web.archive.org/web/20220626113202/https://spring.io/blog/2021/06/25/spring-integration-zip-2-0-0-available)Spring . io
*   [**> >冬眠搜索 6.0.5 .最终发布**](https://web.archive.org/web/20220626113202/https://in.relation.to/2021/06/23/hibernate-search-6-0-5-Final/)in . relation . io

## 2。技术

**[> >时间和分布式系统:版本矢量、](https://web.archive.org/web/20220626113202/https://martinfowler.com/articles/patterns-of-distributed-systems/version-vector.html) [混合时钟、](https://web.archive.org/web/20220626113202/https://martinfowler.com/articles/patterns-of-distributed-systems/hybrid-clock.html) [Lamport 时钟](https://web.archive.org/web/20220626113202/https://martinfowler.com/articles/patterns-of-distributed-systems/lamport-clock.html)**martinfowler.com

分布式系统中同步时钟的幻觉——在分布式系统中维护**历史修订或因果关系**的一些模式。

[**> >用 Istio**](https://web.archive.org/web/20220626113202/https://www.infoq.com/articles/microservicilities-istio/)【infoq.com】实现微服务器

通过 Istio 处理微服务中的大多数**交叉问题——断路器、跟踪、监控等。有趣的读物。**

**同样值得一读:**

*   [**> >为 Pod 标签编写控制器**](https://web.archive.org/web/20220626113202/https://kubernetes.io/blog/2021/06/21/writing-a-controller-for-pod-labels/)kubernetes . io
*   [**> >下一层互联网将如何标准化**](https://web.archive.org/web/20220626113202/https://www.mnot.net/blog/2021/06/21/standards-competition-governance)【mnot.net
*   [**> >探索数据@网飞**](https://web.archive.org/web/20220626113202/https://netflixtechblog.com/exploring-data-netflix-9d87e20072e3)【netflixtechblog.com
*   [**> >在云端构建无服务器应用**](https://web.archive.org/web/20220626113202/https://blog.codecentric.de/en/2021/06/structuring-serverless-applications-in-the-cloud/)[blog . code centric . de]
*   [**> > Lambda 开发工具箱:CLI 日志观察器**](https://web.archive.org/web/20220626113202/https://advancedweb.hu/lambda-development-toolbox-cli-logs-watcher/)[advanced web . Hu]
*   [**> >每个序列化框架都应该有自己的瞬态注释**](https://web.archive.org/web/20220626113202/https://techblog.bozho.net/every-serialization-framework-should-have-its-own-transient-annotation/)【techblog.bozho.net】
*   [**> >为什么后端和前端使用单个域**](https://web.archive.org/web/20220626113202/https://advancedweb.hu/why-use-a-single-domain-for-the-backend-and-the-frontend/)[advanced web . Hu]

## 3。沉思

[**> >开始清洗！**](https://web.archive.org/web/20220626113202/https://reflectoring.io/start-clean/)[reflector ing . io]

想睡个好觉吗？**对代码库中可以控制的事情负责任地行动**——记录决策、解释架构、模块化等等！

**同样值得一读:**

*   [**> >介绍网飞时文创作血统**](https://web.archive.org/web/20220626113202/https://netflixtechblog.com/introducing-netflix-timed-text-authoring-lineage-6fb57b72ad41)【netflixtechblog.com】
*   [**> >介绍 AWS BugBust**](https://web.archive.org/web/20220626113202/https://www.allthingsdistributed.com/2021/06/introducing-aws-bugbust.html)【allthingsdistributed.com】

## 4。漫画

本周我最喜欢的迪尔伯特。

[**> >基于事实！**](https://web.archive.org/web/20220626113202/https://dilbert.com/strip/2021-06-29)【dilbert.com】

[**> >失败者探测器**](https://web.archive.org/web/20220626113202/https://dilbert.com/strip/2021-06-27)【dilbert.com】

[**> >远程射击！**](https://web.archive.org/web/20220626113202/https://dilbert.com/strip/2021-06-28)【dilbert.com】

## 5。本周精选

**[> >如何努力](https://web.archive.org/web/20220626113202/http://www.paulgraham.com/hwh.html)**【paulgraham.com】

Next **»**[Java Weekly, Issue 393](/web/20220626113202/https://www.baeldung.com/java-weekly-393)**«** Previous[Java Weekly, Issue 391](/web/20220626113202/https://www.baeldung.com/java-weekly-391)