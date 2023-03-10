# Java 周刊，第 463 期

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-weekly-463>

## 1. **Spring 和 Java**

[**> > Java 线程原语弃用**](https://web.archive.org/web/20221111123743/https://docs.oracle.com/en/java/javase/19/docs/api/java.base/java/lang/doc-files/threadPrimitiveDeprecation.html)【docs.oracle.com】

先来说说 Java 中的一些`Thread `方法:`stop, suspend, resume,` 和 `ThreadDeath`的**弃用和替代。**

[**> > Spring 数据 JPA 实体锁定**](https://web.archive.org/web/20221111123743/https://vladmihalcea.com/spring-data-jpa-locking/)【vladmihalcea.com】

关于在 JPA 中获取一个或多个实体时应用共享或排他**行级锁的实用指南。**

[**> > JVM 日志 Java 的 Sip**](https://web.archive.org/web/20221111123743/https://inside.java/2022/11/07/sip071/)【inside.java

看看**统一 JVM 日志记录的作用**:启用日志记录、标记配置、减少开销、运行应用程序等等！

#### 同样值得一读:

*   [**> >暂居 JDK:当前在产**](https://web.archive.org/web/20221111123743/https://www.infoq.com/presentations/java-upgrade-path/)【infoq.com】
*   [**>>Maven Central Search 从 Maven Central**](https://web.archive.org/web/20221111123743/https://www.infoq.com/news/2022/11/maven-central-search/)【infoq.com】检索依赖坐标
*   [**> >用低延迟**](https://web.archive.org/web/20221111123743/https://foojay.io/today/creating-terabyte-sized-queues-with-low-latency/) [ foojay.io ]创建太字节大小的队列
*   [**> >回顾 CVE-2022-42889:Apache Commons Text(Text 4 shell)**](https://web.archive.org/web/20221111123743/https://foojay.io/today/reviewing-cve-2022-42889-arbitrary-code-execution-vulnerability-in-apache-commons-text-text4shell/)[foojay . io]
*   [**> >将安全移入 JVM**](https://web.archive.org/web/20221111123743/https://foojay.io/today/moving-security-into-the-jvm/)foojay . io
*   [**> > Azul 通过推出漏洞检测 SaaS**](https://web.archive.org/web/20221111123743/https://www.infoq.com/news/2022/11/azul-vulnerability-detection/)[【infoq.com】T4]加入到改善供应链安全的努力中
*   [**> > Java 性能:超前与即时**](https://web.archive.org/web/20221111123743/https://foojay.io/today/java-performance-ahead-of-time-versus-just-in-time/) [ foojay.io ]

**网络研讨会和演示:**

*   [**> > Java 17 到 20 模式匹配全教程带记录、Instanceof 和 Switch——JEP 咖啡馆# 14**](https://web.archive.org/web/20221111123743/https://inside.java/2022/11/08/jepcafe14/)[【inside.java】T4
*   [**>>open JDK 中的 GraalVM 和更多 JavaOne 公告——内部 Java 新闻广播# 36**](https://web.archive.org/web/20221111123743/https://inside.java/2022/11/03/newscast-036/)[【inside.java】T4
*   [**> >一个精彩的播客:Java 冠军、传奇人物、多产的开源贡献者安德烈斯·阿尔米瑞**](https://web.archive.org/web/20221111123743/https://spring.io/blog/2022/11/03/a-bootiful-podcast-java-champion-legend-and-prolific-open-source-contributor-andres-almiray) [ spring.io
*   [**> >使用 Gmail API 和 Java 发送电子邮件**T3【blog.sebastian-daschner.com】和](https://web.archive.org/web/20221111123743/https://blog.sebastian-daschner.com/entries/sending-emails-gmail-api-java)

**升级时间:**

*   [**> > Spring 框架 6.0.0-RC4 现已可用**](https://web.archive.org/web/20221111123743/https://spring.io/blog/2022/11/09/spring-framework-6-0-0-rc4-available-now)Spring . io
*   [**> >春安 6.0.0-RC2 现已上市**](https://web.archive.org/web/20221111123743/https://spring.io/blog/2022/11/09/spring-security-6-0-0-rc2-is-available-now)Spring . io
*   **[> >夸尔库斯 2.13.4 .最终发布](https://web.archive.org/web/20221111123743/https://quarkus.io/blog/quarkus-2-13-4-final-released/)** [ 夸尔库斯. io
*   [**> >冬眠 ORM 5 . 6 . 14 . final**T3in . relation to](https://web.archive.org/web/20221111123743/https://in.relation.to/2022/11/04/hibernate-orm-5614/)
*   [**> >阿帕奇蜂巢发布-4 . 0 . 0-阿尔法-2-rc1**T3【github.com/apache】号号](https://web.archive.org/web/20221111123743/https://github.com/apache/hive/releases/tag/release-4.0.0-alpha-2-rc1)
*   [**> >春季数据 2022.0.0-RC2 可用**](https://web.archive.org/web/20221111123743/https://spring.io/blog/2022/11/04/spring-data-2022-0-0-rc2-available)【Spring . io
*   [**> > CVE 报告发布为弹簧工具**](https://web.archive.org/web/20221111123743/https://spring.io/blog/2022/11/03/cve-report-published-for-spring-tools)Spring . io
*   [**>>Spring Modulith 0.1 M2 发布**](https://web.archive.org/web/20221111123743/https://spring.io/blog/2022/11/02/spring-modulith-0-1-m2-released)Spring . io
*   [**> >春季会议 3 . 0 . 0-RC1**](https://web.archive.org/web/20221111123743/https://spring.io/blog/2022/10/26/spring-session-3-0-0-rc1)Spring . io
*   [**> >发布说明——Payara 平台社区 6 . 2022 . 1**](https://web.archive.org/web/20221111123743/https://docs.payara.fish/community/docs/6.2022.1/Release%20Notes/Release%20Notes%206.2022.1.html)[docs . Payara . fish]
*   [**>>elastic search 8 . 5 . 0–最新消息**](https://web.archive.org/web/20221111123743/https://www.elastic.co/guide/en/elasticsearch/reference/8.5/release-highlights.html)【elastic.co】
*   **[> >微档案 6.0-RC3 发布](https://web.archive.org/web/20221111123743/https://github.com/eclipse/microprofile/releases/tag/6.0-RC3)**【github.com/eclipse】

## 2。技术&思考

[**>>Titus 网关中一致的缓存机制**](https://web.archive.org/web/20221111123743/https://netflixtechblog.com/consistent-caching-mechanism-in-titus-gateway-6cb89b9ce296)【netflixtechblog.com】

如何水平扩展作为托管数据的**单一真实来源的组件。**

**同样值得一读:**

*   [**> >再也不会丢失数据——事件源拯救！**](https://web.archive.org/web/20221111123743/https://event-driven.io/en/never_lose_data_with_event_sourcing/) [ 事件驱动. io
*   **[> >架构文档作为代码用 Structurizr&ascii doctor:Part 3](https://web.archive.org/web/20221111123743/https://blog.codecentric.de/architecture-documentation-as-code-with-structurizr-and-asciidoctor-part-3-structurizr)[&4](https://web.archive.org/web/20221111123743/https://blog.codecentric.de/architecture-documentation-as-code-with-structurizr-and-asciidoctor-part4-publishing)**[blog . code centric . de
*   [**> >横向是你的朋友在 SQL 中创建局部列变量**](https://web.archive.org/web/20221111123743/https://blog.jooq.org/lateral-is-your-friend-to-create-local-column-variables-in-sql/)【blog.jooq.org】
*   [**> >命令行上的浏览器书签**](https://web.archive.org/web/20221111123743/https://www.hamvocke.com/blog/lnks-command-line-bookmarks/)【hamvocke.com】
*   **[> >新 AWS 欧洲(苏黎士)地区和 16 年瑞士创新](https://web.archive.org/web/20221111123743/https://www.allthingsdistributed.com/2022/11/aws-launches-europe-zurich-region.html)T3【allthingsdistributed.com】和**

## 3。漫画

[**> > Asok 的工作生活平衡**](https://web.archive.org/web/20221111123743/https://dilbert.com/strip/2022-11-09)【dilbert.com】

[**> >呆伯特长有**](https://web.archive.org/web/20221111123743/https://dilbert.com/strip/2022-11-08)【dilbert.com】

[**> >呆呆地听着**](https://web.archive.org/web/20221111123743/https://dilbert.com/strip/2022-11-06)【dilbert.com】

## 4。本周精选

**[>>CORS 是什么？](https://web.archive.org/web/20221111123743/https://simplelocalize.io/blog/posts/what-is-cors/)**[simple localize . io]

**«** Previous[Java Weekly, Issue 462](/web/20221111123743/https://www.baeldung.com/java-weekly-462)