# Baeldung 每周评论 49

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/2014-week-review-49>

2014 年初，我决定跟踪我的阅读习惯，并在 Baeldung 上分享最好的东西。

**开始了……**

## 1。弹簧

让我们从一些激动人心的消息开始回顾——本周有许多春季发布:

*   #### [>>Spring Boot 1 . 2 . 0 发布](https://web.archive.org/web/20220521212015/https://spring.io/blog/2014/12/11/spring-boot-1-2-0-released)

*   #### [> > Spring Boot 1。1 .10 发布](https://web.archive.org/web/20220521212015/https://spring.io/blog/2014/12/11/spring-boot-1-1-10-released)

*   #### [> > Spring Frame 4.1.3 was released](https://web.archive.org/web/20220521212015/https://spring.io/blog/2014/12/09/spring-framework-4-1-3-released)

*   #### [> > Chunan 4.0.0.RC1 released](https://web.archive.org/web/20220521212015/https://spring.io/blog/2014/12/11/spring-security-4-0-0-rc1-released)

当然，还有更多来自 SpringOne 的**录音——大部分是关于 Spring XD 的:**

*   #### [> > Easily develop powerful big data application](https://web.archive.org/web/20220521212015/https://spring.io/blog/2014/12/09/springone2gx-2014-replay-develop-powerful-big-data-applications-easily-with-spring-xd)

    with Spring XD
*   #### [> > Spring XD performs real-time Hadoop workload analysis](https://web.archive.org/web/20220521212015/https://spring.io/blog/2014/12/09/springone2gx-2014-replay-spring-xd-for-real-time-hadoop-workload-analysis)

*   #### [>用春季 XD](https://web.archive.org/web/20220521212015/https://spring.io/blog/2014/12/09/springone2gx-2014-replay-implementing-the-lambda-architecture-with-spring-xd) 实现希腊字母的第 11 个架构

*   #### [> > XD—— in spring-guide](https://web.archive.org/web/20220521212015/https://spring.io/blog/2014/12/09/springone2gx-2014-replay-spring-xd-a-guided-tour)

*   #### [> > Asynchronous/non-blocking micro-service using reactor](https://web.archive.org/web/20220521212015/https://spring.io/blog/2014/12/11/webinar-replay-using-reactor-for-asynch-non-blocking-microservices)

#### [> > Spring Data JPA 教程:获取所需依赖项](https://web.archive.org/web/20220521212015/http://www.petrikainulainen.net/programming/spring-framework/spring-data-jpa-tutorial-getting-the-required-dependencies/)

这是一篇非常有用且切中要点的文章，讲述了如何将正确的 Maven 依赖关系整合在一起以使用 Spring Data JPA。

#### [> >用 Spring Boot 和 MongoDB](https://web.archive.org/web/20220521212015/http://www.petrikainulainen.net/programming/spring-framework/creating-a-rest-api-with-spring-boot-and-mongodb/) 创建 REST API

以及用 Boot 和 MongoDB 构建 REST API 的坚实介绍。

#### [> >启动自己的基础设施——分五步扩展 Spring Boot](https://web.archive.org/web/20220521212015/https://blog.codecentric.de/en/2014/11/extending-spring-boot-five-steps-writing-spring-boot-starter/)

如果你找不到一个已经满足你需求的现有软件，这是一篇关于**打造你自己的 Spring Boot 入门软件**的基础设施层面的详细文章。谁知道呢——也许有一天它会成为官方文件。

#### [> >在@配置中避免条件逻辑](https://web.archive.org/web/20220521212015/http://blog.frankel.ch/avoid-conditional-logic-in-configuration)

当`@Profile`在春天推出时，它对我们控制配置的方式产生了相当大的影响。**我们可以用概要文件做在**之前根本不可能做的事情，至少不干净——这最终是你在一个好的抽象中所寻找的。

Spring 4.0 通过**引入`@Conditional`** 对`@Profile`进行了改进——这是一个更高层次的抽象注解，基本上允许你基于任意数量的条件来控制你的配置，而不仅仅是配置文件。

这篇文章详细介绍了如何使用 Spring Boot 提供的这些条件注释。

## 2。Java

#### [> >一个 Beans v2.0 规范可能包含什么？](https://web.archive.org/web/20220521212015/http://blog.joda.org/2014/12/what-might-beans-v20-spec-contain.html)

一个很酷的实验，关于在新的 Java Beans 2.0 规范中什么是有意义的。但不仅仅如此——一个实际的、早期的实现，你可以检验(并为之做出贡献)。

#### [> >堆上 vs 堆下内存使用量](https://web.archive.org/web/20220521212015/https://vanillajava.blogspot.com/2014/12/on-heap-vs-off-heap-memory-usage.html?view=classic)

JVM 的堆外内存是你在书上读到过的东西之一，但是你可能从来没有真正使用过。尽管如此，这是我喜欢读的一篇文章。

#### [> >别“自作聪明”:双花括号反花样](https://web.archive.org/web/20220521212015/http://blog.jooq.org/2014/12/08/dont-be-clever-the-double-curly-braces-anti-pattern/)

一个老掉牙的好东西——双花括号实例化。是的—**不是个好主意**。

#### [> >版本的不利面——不太乐观锁定](https://web.archive.org/web/20220521212015/http://vladmihalcea.com/2014/12/08/the-downside-of-version-less-optimistic-locking/)

一篇关于**版本不乐观锁**的文章——一个我不知道的很酷的 Hibernate 特性。

#### [> >验证中用通知替换抛出异常](https://web.archive.org/web/20220521212015/http://martinfowler.com/articles/replaceThrowWithNotification.html)

必须阅读**正确处理验证**以及如何重构以获得更好的基于通知的解决方案。

#### [> >弹性搜索提示:插入与更新你的索引](https://web.archive.org/web/20220521212015/https://blog.codecentric.de/en/2014/12/elasticsearch-usage-tipps-transform-update-heavy-index-heavy-indexing/)

我最近用了很多弹性搜索，这很有帮助。

## 3。技术和思考

#### [> >克里斯·理查森通过 Docker](https://web.archive.org/web/20220521212015/http://www.infoq.com/interviews/cqrs-docker-richardson) 讨论 CQRS 和事件采购

克里斯·理查森关于 CQRS 建筑的 15 分钟访谈。

如果你正在建立微服务(或者正在考虑)micro 和事件采购是一个不错的选择。

#### [> >灵活 vs 简单？为什么不两者都要？](https://web.archive.org/web/20220521212015/http://www.daedtech.com/flexibility-vs-simplicity-why-not-both)

关于**批判和重新评估你的信念**的务实文章，甚至(或特别是)行业接受了你以前认为理所当然的信念。如果你换个角度看问题，也许会有更好的方法。

## 4。漫画

本周和呆伯特一起:

#### [> >追捕并杀死我方数据](https://web.archive.org/web/20220521212015/http://www.dilbert.com/strips/comic/2009-08-30/)

#### [> >编造数字](https://web.archive.org/web/20220521212015/http://www.dilbert.com/strips/comic/2008-05-08/)

#### [> >重装你的 OS](https://web.archive.org/web/20220521212015/http://www.dilbert.com/strips/comic/2014-12-01/)

## 5。本周精选

我最近在我的“每周评论”中介绍了“本周精选”部分。**如果你已经在我的电子邮件列表中，你已经得到了选择**，希望你喜欢它。

如果没有，您可以**分享评论**并在此解锁:

[社会标签 id = ' 5073 ']

#### [>>](https://web.archive.org/web/20220521212015/http://www.catb.org/esr/writings/cathedral-bazaar/cathedral-bazaar/)大教堂和集市

可以说是埃里克·s·雷蒙德最有影响力的作品之一，也是一本引人入胜的读物。如果你想清理原来的 XHTML，这里有一个更漂亮的版本🙂

[/sociallocker]