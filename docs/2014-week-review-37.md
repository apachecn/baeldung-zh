# Baeldung 每周评论 37

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/2014-week-review-37>

2014 年初，我决定开始追踪我的阅读习惯，并在 Baeldung 上分享最好的东西。

管理我的阅读使它更有目的和多样化——我也希望通过让本周最好的内容上升到顶部来为你提供价值。

现在——这一周有点不寻常，仅仅是因为**过去几天里出现了大量好文章**。不确定这是什么——可能是假期后创造力的爆发，也可能只是夏天的结束，但数量的上升是相当了不起的。

**开始了……**

## 1。Java

#### [> >数据库锁定和丢失更新现象入门](https://web.archive.org/web/20220521224045/http://vladmihalcea.com/2014/09/14/a-beginners-guide-to-database-locking-and-the-lost-update-phenomena/)

让我们从一篇关于**数据库锁定和“丢失更新”**的文章开始回顾吧，这是一篇很好的研究文章，有很多内容需要了解。

我个人一直关注这个系列，学到了很多东西——我们一直在这里跟踪整个进展，也在每周的评论中。虽然它们都很好，但这是整个系列中最好的一个。

#### [> >为什么不应该实施分层架构](https://web.archive.org/web/20220521224045/http://blog.jooq.org/2014/09/12/why-you-should-not-implement-layered-architecture/)

这篇文章将会引起争议(现在可能已经引起了争议)。就我个人而言——已经看到许多系统的架构带有许多不必要的复杂性(为什么我们不在那里放一个消息队列呢？)–我当然能理解这篇文章的想法。

话虽如此——这不是一件非黑即白的事情——而且**你可能会倾向于过度简化。举一个文章中的例子——你可能不需要十几种保险，但是为你的房子和你的健康投保仍然是一个好主意。**

#### [> >这是最后的讨论！](https://web.archive.org/web/20220521224045/http://blog.jooq.org/2014/09/15/this-is-the-final-discussion/)

**好好讨论一下`final` (关键词)**。如果最终是默认的，哦，人类的苦难会减少。玩笑归玩笑——如果你在编码的时候还没有给`final`很多思考，去读一下这篇文章。

#### [>>λs 和副作用](https://web.archive.org/web/20220521224045/https://vanillajava.blogspot.ro/2014/09/lambdas-and-side-effects.html)

一些有趣的兰姆达斯案件。

最后 JavaZone 会议的所有**视频**都出来了——其中有一些很棒的视频:

#### [>>JavaZone 2014–90 场演讲，60 场闪电会谈](https://web.archive.org/web/20220521224045/http://2014.javazone.no/program.html)

## 2。弹簧

#### [>预览 Spring 安全 WebSocket 支持&会话](https://web.archive.org/web/20220521224045/https://spring.io/blog/2014/09/16/preview-spring-security-websocket-support-sessions)

直到我读了这篇文章，我才意识到 Spring Session 的努力(T1)——这是一个广泛的努力，基本上是用一个新的 Session 实现来完全取代容器管理的 Session(T3)。这是一个目标——看起来它有一些非常有趣的实际优势，至少对一类问题来说是这样。

#### [> >在 Spring Boot 使用@ configuration properties](https://web.archive.org/web/20220521224045/http://blog.codeleak.pl/2014/09/using-configurationproperties-in-spring.html)

非常酷的**Spring 的标准属性处理方式的替代方式**——我绝对能看到这个 Spring Boot 选项是如何派上用场的。

#### [> >在 Spring Boot 测试邮件代码](https://web.archive.org/web/20220521224045/http://blog.codeleak.pl/2014/09/testing-mail-code-in-spring-boot.html)

Spring Boot 探索，特别是如何**设置你的电子邮件逻辑，并使用一些有趣的邮件工具来测试它**,这些工具看起来很方便独立的单元测试——不错。

#### [> >用 Spring Boot 和 Spring MVC](https://web.archive.org/web/20220521224045/http://www.java-allandsundry.com/2014/09/customizing-httpmessageconverters-with.html) 定制 HttpMessageConverters

Spring Boot 让事情变得更简单的另一种方式是—**在系统中配置 Http 消息转换器**。我早就想在春天做这件事了——现在终于有可能了，这太好了。

#### [>](https://web.archive.org/web/20220521224045/http://www.infoq.com/interviews/Juergen-Hoeller-QConNY-2014-Interview)

对 Juergen ho eller(Spring 的联合创始人)的一次很好的采访经历了许多引人入胜的问题，有些可能是你意想不到的。如果你决定观看面试，期待一个有趣的条件配置解释。

#### [> >网络研讨会回放:用 Spring Boot](https://web.archive.org/web/20220521224045/https://spring.io/blog/2014/09/17/webinar-replay-building-bootiful-microservices-with-spring-boot) 打造“Bootiful”微服务

最后，我将在周末观看一个关于微服务的网络研讨会。如果你一直在关注我的每周评论，你已经知道微服务并不容易实现——当系统不再琐碎时，会有很多潜在的陷阱。

## 3。技术和思考

#### [> >负载测试指南](https://web.archive.org/web/20220521224045/http://techblog.bozho.net/?p=1535)

关于负载测试实践和考虑的精彩文章。即使你已经这样做了一段时间——读一下这篇文章也是一个好主意。

#### [> >“我喜欢嘲笑，但我不信任间谍”](https://web.archive.org/web/20220521224045/http://blog.thecodewhisperer.com/2014/09/15/i-like-mocks/)

深入探究**新手开发者如何使用**间谍，以及这些实践如何随着开发者的经验水平而改变以及应该如何改变。我计划从这篇文章中获得一些有用的见解，并应用到我自己的实践中。

#### [> >速射手艺提示](https://web.archive.org/web/20220521224045/http://www.daedtech.com/rapid-fire-craftsmanship-tips)

船长明显在这里—**改善你的工艺是一个持续的事情**——没有“到来”。这里有一套关于如何做到这一点的好建议。

#### [> >一阶段提交——内存缓存快速事务](https://web.archive.org/web/20220521224045/https://gridgain.blogspot.ro/2014/09/one-phase-commit-fast-transactions-for.html)

从“两阶段提交”到轻度的“一阶段提交”操作——这当然带来了很好的速度优势——如果你在使用内存缓存的话，这是另一个有趣的阅读**。**

我个人现在没有这样做，但我过去有过，如果设置正确，它们会非常有用，如果不正确，它们会非常烦人。

## 4。漫画

人们会厌倦 XKCD 的优点吗:

#### [>篝火](https://web.archive.org/web/20220521224045/https://xkcd.com/742/)

#### [> >旅行推销员问题](https://web.archive.org/web/20220521224045/https://xkcd.com/399/)

#### [>密码重用](https://web.archive.org/web/20220521224045/https://xkcd.com/792/)

我说“T0”。

## 5。本周精选

本周，我想我们应该尝试一些新的东西——在我的“每周评论”中，我引入了一个新的“每周精选”部分。有趣的是，我打算把它只提供给我的电子邮件列表订阅者。

所以——如果你是从我的邮件列表中看到这篇文章的，你已经有了选择——希望你喜欢🙂

如果没有，你当然可以订阅这个列表来获得下一个。

干杯。