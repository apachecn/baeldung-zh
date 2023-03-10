# Java 周刊，第 184 期

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-weekly-184>

本周有很多关于 Java 9 的有趣文章。

**开始了……**

## 1。Spring 和 Java

#### [**>>Java 8 流真的很懒吗？不完全！**](https://web.archive.org/web/20220525134903/https://blog.jooq.org/2017/07/03/are-java-8-streams-truly-lazy-not-completely/)【blog.jooq.org】

事实证明，Java 8 **Streams API 并不像你想象的那样懒惰**—`flatmap()`操作急切地评估内部`Stream`——当使用 [Scala](https://web.archive.org/web/20220525134903/https://www.scala-lang.org/) 或 [Vavr 时，情况并非如此。](https://web.archive.org/web/20220525134903/http://www.vavr.io/)

#### [**> >简单的 Spring Boot 管理员设置**](https://web.archive.org/web/20220525134903/https://techblog.bozho.net/simple-spring-boot-admin-setup/)【techblog.bozho.net】

酷 Spring Boot 管理仪表板设置可能有点不直观-这里是如何设置它的一个很好的概述。

#### [**>>JPA 2.2 中的新特性—`Stream`一次`Query`执行的结果**](https://web.archive.org/web/20220525134903/https://vladmihalcea.com/2017/07/04/whats-new-in-jpa-2-2-stream-the-result-of-a-query-execution/)vladmihalcea.com

JPA 2.2 的新增功能——将`Query`结果返回为`Stream –`是一个有趣的功能，但仍然不如分页的 `ResultSet.`有效

#### [**> >为什么要避免 CascadeType。去除对多关联，改为做什么**](https://web.archive.org/web/20220525134903/https://www.thoughts-on-java.org/avoid-cascadetype-delete-many-assocations/)【thoughts-on-java.org】

使用`CascadeType.REMOVE`可能相当危险——除了**生成太多的查询**之外，它还可能删除超出预期的内容。

#### 同样值得一读:

*   #### [**> > Java module platform system (JSR 376) passed the public review and reconsideration vote**](https://web.archive.org/web/20220525134903/https://www.infoq.com/news/2017/07/jsr-376-approved?utm_campaign=infoq_content&utm_source=infoq&utm_medium=feed&utm_term=Java) [infoq.com]

*   #### [**> > Why JVM is a good choice for serverless computing: John Chapin discusses AWS Lambda 【T5] on QConny**](https://web.archive.org/web/20220525134903/https://www.infoq.com/news/2017/06/fearless-aws-lambda?utm_campaign=infoq_content&utm_source=infoq&utm_medium=feed&utm_term=Java) [infoq.com] T4]

*   #### [**> >码头工人监控:Docker**](https://web.archive.org/web/20220525134903/http://blog.takipi.com/docker-monitoring-5-methods-for-monitoring-java-applications-in-docker/)【blog . taki pi . com】中监控 Java 语言(一种计算机语言，尤用于创建网站)应用的 5 种方法

*   #### [**> >爪哇命令行接口（第四部分):命令行**](https://web.archive.org/web/20220525134903/https://marxsoftware.blogspot.com/2017/06/Commandline.html)【Marx software . blogspot . com】T5

*   #### [**> >爪哇命令行接口（第五部分):JewelCli**](https://web.archive.org/web/20220525134903/https://marxsoftware.blogspot.com/2017/06/jewelcli.html)【Marx software . blogspot . com】

*   #### [>为 CXF 和招摇](https://web.archive.org/web/20220525134903/http://tech.asimio.net/2017/06/29/Implementing-a-custom-SpringBoot-starter-for-CXF-and-Swagger.html)实现自定义 Spring Boot 首发

**网络研讨会和演示:**

*   #### [**> > Parasitic programming language**](https://web.archive.org/web/20220525134903/https://www.infoq.com/presentations/language-runtime) [infoq.com]

**升级时间:**

*   #### [**> >休眠验证器 6。0 .0 .cr1 出带 Bean 验证 2.0.0.CR1 支持**](https://web.archive.org/web/20220525134903/http://in.relation.to/2017/06/29/hibernate-validator-600-cr1-out/) 在。关系。到

*   #### [**> > Spring Cloud Data Stream 1.2.2 Release**](https://web.archive.org/web/20220525134903/https://spring.io/blog/2017/06/29/spring-cloud-data-flow-1-2-2-released) Spring.io

*   #### [**>新发布的 SSL/TLS 部署最佳实践**](https://web.archive.org/web/20220525134903/https://blog.ivanristic.com/2016/06/new-release-of-ssl-tls-best-practices.html)【blog . ivanristic . com】

## 2。技术

#### [**> >一个基本的编程模式:先过滤，后贴图**](https://web.archive.org/web/20220525134903/https://blog.jooq.org/2017/06/29/a-basic-programming-pattern-filter-first-map-later/)【jooq.org】

为了利用`Stream` API 的惰性并降低操作的复杂性，尽可能依赖适当的限制是很重要的——尽管这可能不会在所有场景中强制执行惰性。

#### [>>ORM 应该更新“改变”的值，而不仅仅是“修改”那些](https://web.archive.org/web/20220525134903/https://blog.jooq.org/2017/06/28/orms-should-update-changed-values-not-just-modified-ones/)【jooq.org】

许多 ORM 更新被“触动”但不一定改变的值——这并不理想。阅读整篇文章，深入了解问题和一些可能的解决方案。

## 3。沉思

#### [**> >一看 5 NoSQL 方案**](https://web.archive.org/web/20220525134903/http://www.daedtech.com/look-5-nosql-solutions/)【daedtech.com】

NoSQL 和最流行的解决方案的快速实用介绍。

#### [**> >停止等待完美，从错误中吸取教训**](https://web.archive.org/web/20220525134903/http://www.allthingsdistributed.com/2017/06/stop-waiting-for-perfection.html)【allthingsdistributed.com】

错误/失误时有发生，我们需要学会如何接受它们，以便改进和创新，因为它们是过程的一部分。

**同样值得一读:**

*   #### [> >艾与机器语言(Machine Language)的区别](https://web.archive.org/web/20220525134903/https://horicky.blogspot.com/2017/07/how-ai-differs-from-ml.html)【horicky . blogspot . com】

*   #### [**> > The key to becoming a software consultant**](https://web.archive.org/web/20220525134903/http://www.daedtech.com/key-becoming-software-consultant/) [daedtech.com]

*   #### [**> > Clarify your values and expectations**](https://web.archive.org/web/20220525134903/http://www.mehdi-khalili.com/be-clear-and-explicit-about-your-values-and-expectations) [mehdi-khalili.com]

*   #### [>如何从优秀走向伟大？](https://web.archive.org/web/20220525134903/http://www.ontestautomation.com/how-to-move-up-from-being-good-to-being-great/)【ontestautomation . com】

*   #### [**> > Developer-oriented essentialism**](https://web.archive.org/web/20220525134903/https://blog.codecentric.de/en/2017/07/essentialism-for-developers/) [ blog.codecentric.de ]

*   #### [**> > Free programming without marketing sense**](https://web.archive.org/web/20220525134903/http://www.daedtech.com/freelance-programming-without-marketing/) [daedtech.com]

## 4。漫画

本周我最喜欢的迪尔伯特。

#### [> >你看低人](https://web.archive.org/web/20220525134903/http://dilbert.com/strip/2011-03-30)【dilbert.com】

#### [> >更新我的好友资源](https://web.archive.org/web/20220525134903/http://dilbert.com/strip/2010-10-22)【dilbert.com】

## 5。本周精选

#### [> >在这 7 种情况下说谢谢让你的生活更美好](https://web.archive.org/web/20220525134903/http://jamesclear.com/say-thank-you)【jamesclear.com】