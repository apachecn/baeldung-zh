# Java 周刊，第 244 期

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-weekly-244>

**开始了……**

## 1。Spring 和 Java

#### [**> >乐观锁定 JPA/休眠**T3【blog.arnoldgalovics.com】乐观锁定](https://web.archive.org/web/20220703155125/https://blog.arnoldgalovics.com/optimistic-locking-in-jpa-hibernate/)

很好地回顾了**丢失更新异常以及如何使用版本化和非版本化乐观数据库锁检测它们**。

#### [**> > Bootiful GCP:与 Spring Cloud 的关系数据访问 GCP(2/8)**](https://web.archive.org/web/20220703155125/https://spring.io/blog/2018/08/23/bootiful-gcp-relational-data-access-with-spring-cloud-gcp-2-8)Spring . io

在 Spring Boot 和谷歌云平台系列的这一期中，我们看到**连接到托管 MySQL 数据库**并执行一些基本查询是多么容易。

#### [**> > Bootiful GCP:使用扳手进行全局一致的数据访问(3/8)**](https://web.archive.org/web/20220703155125/https://spring.io/blog/2018/08/27/bootiful-gcp-globally-consistent-data-access-with-spanner-3-8) [ spring.io

再来一次，本教程将带领我们通过使用 Spring 数据集成到 Google Spanner】。非常酷。

#### [**> >休眠提示:将 1 个实体属性映射到 2 列**](https://web.archive.org/web/20220703155125/https://www.thoughts-on-java.org/hibernate-tips-map-1-attribute-2-columns/)【thoughts-on-java.org】

一篇精彩的文章为我们提供了一种方法，通过使用一个瞬态属性和两个内部属性来解决遗留数据库代码中常见的问题。

#### [**> > Hibernate 数据库模式多租户**](https://web.archive.org/web/20220703155125/https://vladmihalcea.com/hibernate-database-schema-multitenancy/)【vladmihalcea.com

在上周关于基于目录的多租户文章的后续文章中，我们看到了**基于模式的多租户如何适用于明确区分目录和模式的数据库**，例如 PostgreSQL。

#### [**> >在 Azure 虚拟机**](https://web.archive.org/web/20220703155125/https://www.infoq.com/articles/azure-wildfly-spring-boot-app)【infoq.com】上构建一个运行在 WildFly 上的 MySQL Spring Boot 应用

关于使用 Azure Database for MySQL 创建 MySQL 服务器并从基于 Wildfly 的 Spring Boot 应用程序连接到它的很好的教程。好东西。

#### [**> > Java 依然零成本**](https://web.archive.org/web/20220703155125/http://blog.joda.org/2018/08/java-is-still-available-at-zero-cost.html)【blog.joda.org】

尽管有传言说 Java 仍然是免费的，但是如果你坚持使用旧版本并且仍然依赖 Oracle 的支持，请准备好支付大笔费用。

#### [**> >将 Maven 项目迁移到 Java 11**](https://web.archive.org/web/20220703155125/https://winterbe.com/posts/2018/08/29/migrate-maven-projects-to-java-11-jigsaw/)【winterbe.com】

如果您只是想在 JDK 11 上运行您的应用程序，但不关心新的 Jigsaw 模块系统，这是一个可靠的资源。

#### 同样值得一读:

*   #### [**> >弦。(对象)与对象的值。tostring(Object)**](https://web.archive.org/web/20220703155125/https://marxsoftware.blogspot.com/2018/08/null-safe-to-string.html)【Marx software . blogspot . com】T5

*   #### [**> > How to fix Hibernate's "JDBC type has no dialect mapping"**](https://web.archive.org/web/20220703155125/https://vladmihalcea.com/hibernate-no-dialect-mapping-for-jdbc-type/) [vladmihalcea.com]

*   #### [**> > Jib, Java container image builder**](https://web.archive.org/web/20220703155125/https://www.infoq.com/news/2018/08/jib)

    from Google [infoq.com
*   #### [**> > Uber open source JVM Profiler is used to track distributed JVM**](https://web.archive.org/web/20220703155125/https://www.infoq.com/news/2018/08/uber-jvm-profiler) [infoq.com]

*   #### [**> >托米:使用 JCache 配合 CDI**](https://web.archive.org/web/20220703155125/https://www.tomitribe.com/blog/using-jcache-with-cdi-2/)【tomi tribe . com】

*   #### [**> > QCon New York 2018: Better developer experience of Netflix: Multilingualism and containers**](https://web.archive.org/web/20220703155125/https://www.infoq.com/news/2018/08/better-devex-at-netflix) infoq.com

**升级时间:**

*   #### [**> >休眠验证器 6。0 .13 .最终发布**](https://web.archive.org/web/20220703155125/http://in.relation.to/2018/08/23/hibernate-validator-6013-final-out/)

*   #### [**>>Spring Security oauth 2 Boot 自动配置 2。0 .4 & 2。1 .0 .货币供应量之二发布**](https://web.archive.org/web/20220703155125/https://spring.io/blog/2018/08/29/spring-security-oauth2-boot-auto-config-2-0-4-2-1-0-m2-released) 春天。io

*   #### [**> >月食发布了微简介**](https://web.archive.org/web/20220703155125/https://www.infoq.com/news/2018/08/microprofile-1.4-and-2.0)【infoq . com】1.4 和 2.0 版本

*   #### [**> > JDK 11:首发候选人**T4【邮件】open JDK . Java . net](https://web.archive.org/web/20220703155125/http://mail.openjdk.java.net/pipermail/jdk-dev/2018-August/001844.html)

## 2。技术和思考

#### [**> >如何从一个庞然大物中提取数据丰富的服务**](https://web.archive.org/web/20220703155125/https://martinfowler.com/articles/extract-data-rich-service.html)【martinfowler.com】

另一个有希望的系列——这一期将该任务的模式描述为一系列步骤，旨在最大限度地减少对服务消费者的干扰。一个伟大的方法。

#### [**> >返璞归真:依赖注入**](https://web.archive.org/web/20220703155125/https://blog.frankel.ch/basics-dependency-injection/) [ blog.frankel.ch

一个快速复习课程吹捧直接投资的优点，即使许多人因为错误的信息而质疑它的价值。

#### [**> >为什么证书钉住 HPKP 不是个好主意**](https://web.archive.org/web/20220703155125/https://advancedweb.hu/2018/08/28/hpkp/)[advanced web . Hu]

对 HTTP 公钥锁定的研究— **乍听起来可能不错，但伴随着不可接受的风险**。避开。

#### [**> >橙色代号**](https://web.archive.org/web/20220703155125/https://michaelfeathers.silvrback.com/orange-code)【michaelfeathers.silvrback.com】

一个颇有见地的类比将苹果比作橙子，苹果是整体的方法，橙子是它们精心制作的等价物，通过方法提取实现。

#### [**>>2018 年敏捷软件的状态**](https://web.archive.org/web/20220703155125/https://martinfowler.com/articles/agile-aus-2018.html)【martinfowler.com】

一篇深思熟虑的文章概述了敏捷必须克服的一些挑战，比如“伪敏捷”和“敏捷工业综合体”。

#### [**> >将低价值的程序员证书转化为高价值的身份证书**](https://web.archive.org/web/20220703155125/https://daedtech.com/transmuting-low-value-programmer-cred-into-high-value-status-illegibility/)【daedtech.com】

对影响程序员招聘实践的动态的有趣观察。

**同样值得一读:**

*   #### [**> > Set up network data monitoring with relaxation alarm**](https://web.archive.org/web/20220703155125/https://blog.arnoldgalovics.com/setting-up-netdata-monitoring-with-slack-alarms/) [blog.arnoldgalovics.com]

*   #### [**> > Experience and lessons after serving thousands of concurrent users in a devops team for one year**](https://web.archive.org/web/20220703155125/https://vanwilgenburg.wordpress.com/2018/08/22/lessons-learned-after-serving-thousands-of-concurrent-users-in-a-devops-team-for-a-year/) [vanwilgenburg.wordpress.com]

*   #### [**>设定期望**](https://web.archive.org/web/20220703155125/https://dandreamsofcoding.com/2018/08/22/setting-expectations/)【dandreamsofcoding . com】T5

*   #### [>了解物联网（上)](https://web.archive.org/web/20220703155125/https://blog.codecentric.de/en/2018/08/understandig-iot-part-1/)博客。代码为中心。德

*   #### [**>了解物联网（下)**](https://web.archive.org/web/20220703155125/https://blog.codecentric.de/en/2018/08/understanding-iot-part-2/)博客。代码为中心。德

*   #### [**> > Part II: [ T3, [medium.com] T4 Netflix schedule book**](https://web.archive.org/web/20220703155125/https://netflixtechblog.com/scheduling-notebooks-348e6c14cfd6)

*   #### [**> > The future of software delivery is code. And here** T3 [the-composition.com] T3 T5]](https://web.archive.org/web/20220703155125/https://the-composition.com/the-future-of-software-delivery-is-code-and-its-here-a2601759d99b)

## 3。漫画

本周我最喜欢的迪尔伯特。

#### [**> >根据道格伯特**](https://web.archive.org/web/20220703155125/http://dilbert.com/strip/2018-08-29)【dilbert.com进行时间管理

#### [**> >沃利作为导师**](https://web.archive.org/web/20220703155125/http://dilbert.com/strip/2018-08-27)【dilbert.com】

#### [**> >我们喜欢数据库**](https://web.archive.org/web/20220703155125/http://dilbert.com/strip/1996-02-27)【dilbert.com】

## 4。本周精选

#### 

本周，我终于宣布了即将在我的春季课程中推出的新内容——都与 Spring Boot 2 和春季 5.1 有关(以及即将到来的价格变化):

#### [>>`REST With Spring`](/web/20220703155125/https://www.baeldung.com/rest-with-spring-course#new-modules)即将推出新模块

下一期[Java 周刊，第 245 期](/web/20220703155125/https://www.baeldung.com/java-weekly-245)上一期[Java 周刊，第 243 期](/web/20220703155125/https://www.baeldung.com/java-weekly-243)