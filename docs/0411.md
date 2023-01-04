# Java 周刊，第 438 期

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-weekly-438>

## 1。Spring 和 Java

[**> > JEP 428:结构化并发(孵化器)**](https://web.archive.org/web/20220811165416/https://openjdk.java.net/jeps/428)【openjdk.java.net】

多线程代码具有更好的**可靠性、可观察性、**T2，避免了常见的取消和关闭风险。

[**> > JEP 405:记录模式(预览)**](https://web.archive.org/web/20220811165416/https://openjdk.java.net/jeps/405)【openjdk.java.net】

**满足记录的析构模式模式匹配**-一种在模式匹配中从记录中提取信息的酷方法。

[**> >调试 LinkedIn 的 JVM 崩溃——第二部分**](https://web.archive.org/web/20220811165416/https://devblogs.microsoft.com/java/debugging-a-jvm-crash-for-linkedin-part-2/)【devblogs.microsoft.com】

并且，深入到 JVM 内部的兔子洞——这一次，**让我们检查 JVM 的核心转储，以便针对编译的方法进行反汇编**。

#### 同样值得一读:

*   [**> >爪哇生态系统的状态来自新遗迹**](https://web.archive.org/web/20220811165416/https://www.infoq.com/news/2022/05/java-ecosystem-report-2022)【infoq.com】
*   [**> >配置休眠方言的最佳方式**](https://web.archive.org/web/20220811165416/https://vladmihalcea.com/hibernate-dialect/)【vladmihalcea.com】
*   [**>>jOOQ**](https://web.archive.org/web/20220811165416/https://blog.jooq.org/the-many-different-ways-to-fetch-data-in-jooq/)【blog.jooq.org】中获取数据的多种不同方式
*   [**> >玩弄柯特林的语境接收者**](https://web.archive.org/web/20220811165416/https://blog.frankel.ch/kotlin-context-receivers/)frankel . ch
*   [**>>CVE-2022-22976:BCrypt 跳过盐轮为工作因子 31**](https://web.archive.org/web/20220811165416/https://spring.io/blog/2022/05/15/cve-2022-22976-bcrypt-skips-salt-rounds-for-work-factor-of-31) [ spring.io ]
*   [**>>CVE-2022-22978:RegexRequestMatcher**](https://web.archive.org/web/20220811165416/https://spring.io/blog/2022/05/15/cve-2022-22978-authorization-bypass-in-regexrequestmatcher)[spring . io]中的授权旁路
*   [**> >用 ORM 6**](https://web.archive.org/web/20220811165416/https://in.relation.to/2022/05/12/orm-uuid-mapping/)映射 UUID 值与
*   [**> >如何修复 Hibernate 的警告“用集合 fetch 指定的 first result/max results”**](https://web.archive.org/web/20220811165416/https://thorben-janssen.com/hibernate-warning-firstresult-maxresults/)[【thorben-janssen.com】]
*   [**>>Mark Little 在 Devoxx UK 22 上看到的 Java 的未来:原生 Java、Adoptium 和更快的步伐**](https://web.archive.org/web/20220811165416/https://www.infoq.com/news/2022/05/future-java-may22/)[【infoq.com】T4
*   [**> >面向 Java 应用的谷歌云结构化日志**](https://web.archive.org/web/20220811165416/http://www.java-allandsundry.com/2022/05/google-cloud-structured-logging-for.html)【java-allandsundry.com

**网络研讨会和演示:**

*   [**> >一个精彩的播客:EasyMock 贡献者，Java 冠军，Java 杰出人物亨利·特伦布莱**](https://web.archive.org/web/20220811165416/https://spring.io/blog/2022/05/12/a-bootiful-podcast-easymock-contributor-java-champion-and-java-luminary-henri-tremblay) [ spring.io
*   [**>>J Spring 框架 6: HTTP 接口**](https://web.archive.org/web/20220811165416/https://www.youtube.com/watch?v=A1V71peRNn0)【youtube.com】
*   [**> > JFR 事件流——啜饮爪哇**](https://web.archive.org/web/20220811165416/https://inside.java/2022/05/12/sip49/)【inside.java
*   [**> >爪哇接下来——从琥珀到织机，从巴拿马到瓦尔哈拉**T3、【inside.java】T4](https://web.archive.org/web/20220811165416/https://inside.java/2022/05/09/java-next/)
*   [**> > JUnit 5 教程——好听的&轻松的**](https://web.archive.org/web/20220811165416/https://www.youtube.com/watch?v=6uSnF6IuWIw)【youtube.com】

**升级时间:**

*   [**>>Spring Security 5 . 7 . 0、5.6.4、5.5.7 发布——修复了 CVE-2022-22975&CVE-2022-22976**](https://web.archive.org/web/20220811165416/https://spring.io/blog/2022/05/15/spring-security-5-7-0-5-6-4-5-5-7-released-fixes-cve-2022-22975-cve-2022-22976)[Spring . io]
*   [**>>Spring Security 6 . 0 . 0-M5 现已推出**](https://web.archive.org/web/20220811165416/https://spring.io/blog/2022/05/18/spring-security-6-0-0-m5-available-now)Spring . io
*   [**> >春季数据 2021.2 和 2022.0 M4 发布**](https://web.archive.org/web/20220811165416/https://spring.io/blog/2022/05/13/spring-data-2021-2-and-2022-0-m4-released)Spring . io
*   [**> > Spring 框架 6.0.0-M4 现已可用**](https://web.archive.org/web/20220811165416/https://spring.io/blog/2022/05/12/spring-framework-6-0-0-m4-available-now)Spring . io
*   [**>>Hibernate ORM 5 . 6 . 9 . final 和 Hibernate Reactive 1 . 1 . 5 . final**](https://web.archive.org/web/20220811165416/https://in.relation.to/2022/05/16/hibernate-orm-569-and-reactive-115/)in . relation to
*   [**> >夸尔库斯 2.9.0 .最终发布**](https://web.archive.org/web/20220811165416/https://quarkus.io/blog/quarkus-2-9-0-final-released/)夸尔库斯. io

## 2。技术&思考

[**>>Kubernetes 1.24:gRPC 容器探针内测**](https://web.archive.org/web/20220811165416/https://kubernetes.io/blog/2022/05/13/grpc-probes-now-in-beta/) [ kubernetes.io ]

现在我们可以**连接到 gRPC 端点进行启动、活动和就绪探测**——不再有愚蠢的变通办法。

PostgreSQL: 4 中的 [**> >查询。索引扫描**](https://web.archive.org/web/20220811165416/https://postgrespro.com/blog/pgsql/5969493)【postgrespro.com】

Postgres 中**不同索引扫描方法的深入探讨:普通索引、仅索引扫描和可见性映射、位图扫描和成本估算。**

**同样值得一读:**

*   [**> > Kubernetes 1.24:卷填充器升级到 Beta**](https://web.archive.org/web/20220811165416/https://kubernetes.io/blog/2022/05/16/volume-populators-beta/)[Kubernetes . io
*   [**> >如何保存 Bash**](https://web.archive.org/web/20220811165416/https://advancedweb.hu/how-to-save-entered-values-in-bash/)[advanced web . Hu]中输入的值
*   [**> > GitLab 安全扫描–第三部分:Kubernetes 部署**](https://web.archive.org/web/20220811165416/https://blog.codecentric.de/en/2022/05/gitlab-security-scanning-part-3-kubernetes-deployments/)[blog . code centric . de]
*   [**> >这叫依赖“注入”是有原因的**](https://web.archive.org/web/20220811165416/https://www.beust.com/weblog/its-called-dependency-injection-for-a-reason/)【beust.com】
*   [**> >软件复活节彩蛋万岁！**](https://web.archive.org/web/20220811165416/https://queue.acm.org/detail.cfm?id=3534857)【queue.acm.org】
*   [**> >键盘锁。x，但是安全——没有易受攻击的库**](https://web.archive.org/web/20220811165416/https://blog.codecentric.de/en/2022/05/keycloak-x-but-secure-without-vulnerable-libraries/)[blog . codecentric . de]
*   [**> >与 Postgres 的未竟之业**](https://web.archive.org/web/20220811165416/https://www.craigkerstiens.com/2022/05/18/unfinished-business-with-postgres/)【craigkerstiens.com】

## 3。漫画

本周我最喜欢的迪尔伯特。

[**> >焦虑平手**](https://web.archive.org/web/20220811165416/https://dilbert.com/strip/2022-05-19)【dilbert.com】

[**> >厌烦或焦虑**](https://web.archive.org/web/20220811165416/https://dilbert.com/strip/2022-05-16)【dilbert.com】

[**> >工作更聪明**](https://web.archive.org/web/20220811165416/https://dilbert.com/strip/2022-05-14)【dilbert.com】

## 4。本周精选

基于实际代码库创建**应用程序现代化计划**的有趣工具:

#### [> > vFunction 评估枢纽](/web/20220811165416/https://www.baeldung.com/vfunction-free-trial-2a3k)【vfunction.com】

Next **»**[Java Weekly, Issue 439](/web/20220811165416/https://www.baeldung.com/java-weekly-439)**«** Previous[Java Weekly, Issue 437](/web/20220811165416/https://www.baeldung.com/java-weekly-437)