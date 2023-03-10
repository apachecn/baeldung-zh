# Baeldung 每周评论 14

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/2014-week-review-14>

## Java

#### [> > Java 8 星期五:不再需要 ORM](https://web.archive.org/web/20220521223409/http://blog.jooq.org/2014/04/11/java-8-friday-no-more-need-for-orms/)

这加入了以前的讨论(我需要 ORM 吗？)使用新的有前途的工具——看看代码示例，会惊讶地发现这确实是 Java 代码。非常酷。

#### [>>JUnit 中处理异常的另一种方式:捕捉异常](https://web.archive.org/web/20220521223409/http://blog.codeleak.pl/2014/04/yet-another-way-to-handle-exceptions-in.html)

处理 JUnit 测试中**异常的可靠方法——BDD`catch-exception`库。我现在正在尝试——因为它看起来确实很酷。**

#### [> >面向高吞吐量低延迟 Java 应用的垃圾收集优化](https://web.archive.org/web/20220521223409/https://engineering.linkedin.com/garbage-collection/garbage-collection-optimization-high-throughput-and-low-latency-java-applications)

好好复习一下**为真实生产环境调优 Java 垃圾收集**。回顾了大多数 GC 概念，但是是以一种务实的、注重优化的方式进行的，这是对现有标准 GC 文章的一个很好的改变。

#### [> > Maven Git 流插件更好的版本](https://web.archive.org/web/20220521223409/https://blogs.atlassian.com/2013/05/maven-git-flow-plugin-for-better-releases/)

有趣的是，已经建立的 Maven 发布插件，使得 git 的所有好东西都成为一流公民。

#### [>Java 8 语言变化](https://web.archive.org/web/20220521223409/https://www.ibm.com/developerworks/java/library/j-java8lambdas/index.html)

#### [> > JVM 并发:Java 8 并发基础知识](https://web.archive.org/web/20220521223409/https://www.ibm.com/developerworks/java/library/j-jvmc2/index.html)

IBM Developerworks 有两篇关于 Java 8 的文章——语言变化和并发性。

#### [> >阿帕奇雄猫 8 预告](https://web.archive.org/web/20220521223409/http://www.java-tv.com/2014/04/07/apache-tomcat-8-preview/)

最后，一场精彩的网络研讨会介绍了即将推出的 Tomcat 8 服务器。如果您正在使用 Tomcat，这是一个很好的资源，可以让您及时了解即将发生的事情。

## 春天

#### [> >萨根项目:零停机部署](https://web.archive.org/web/20220521223409/https://spring.io/blog/2014/04/04/project-sagan-zero-downtime-deployments)

project Sagan——新的 reference Spring 应用程序——看起来越来越有趣，尤其是考虑到它正在为`spring.io`提供动力。这篇文章展示了代码是如何部署的——这是一个非常好的系列，我将密切关注它。

#### [> >跟踪异常——第四部分——春季邮件发送者](https://web.archive.org/web/20220521223409/http://www.captaindebug.com/2014/04/tracking-exceptions-part-4-springs-mail.html)

如果你读过我最近的几篇每周评论，你会看到这个系列的早期文章。这种应用程序——理解日志文件数据——是任何大型项目都应该解决的问题。

#### [> > CSRF 保护春天 MVC，百里叶，春天安全应用](https://web.archive.org/web/20220521223409/http://blog.codeleak.pl/2014/04/csrf-protectrion-in-spring-mvc.html)

简明扼要地说明了 CSRF 攻击是如何工作的，以及如何用 Spring Security (3.2+)来防范它。非常好。

## 技术

### [> > TDD 棋局第三部分:磕磕绊绊与重构](https://web.archive.org/web/20220521223409/http://www.daedtech.com/tdd-chess-game-part-3-stumbling-and-refactoring)

我以前谈论过这个系列，但是，也许并不奇怪，它又是本周的阅读(或观看)了。为什么我一直选它？简单——这是为数不多的能让你克服实施 TDD 的最初阻力的事情之一。这对我来说花了几年时间，所以我知道这不容易——但结果是巨大的。

所以——开门见山——如果你这周只打算读一篇文章，那就去读这篇吧(实际上，读第[首](https://web.archive.org/web/20220521223409/http://www.daedtech.com/tdd-and-modeling-a-chess-game) [和第二](https://web.archive.org/web/20220521223409/http://www.daedtech.com/tdd-chess-game-part-2)部分会更好)。

#### [>MongoDB 2.6 是$ out](https://web.archive.org/web/20220521223409/http://vladmihalcea.com/2014/04/08/mongodb-2-6-is-out/)

Vlad 对 MongoDB 2.6 中的新功能进行了有用的概述。

#### [>Web API 和 n+1 问题](https://web.archive.org/web/20220521223409/https://byterot.blogspot.com.es/2014/04/web-apis-and-n-plus-1-problem-web-api-rest-cache-mongodb-soa-microsoervice-timeout-retry-circuit-breaker-layered-caching-nosql.html)

臭名昭著的 **n+1 问题**也存在于 Web APIs 中。这篇文章有很多信息需要消化，所以不要着急。

#### [>安全:心脏出血漏洞](https://web.archive.org/web/20220521223409/https://github.com/blog/1818-security-heartbleed-vulnerability)

是的，Heartbleed 是本周披露的，是的，有许多文章涉及它，但我假设你已经阅读了其中的一些，所以我只包括许多文章中的一篇 github 报告。

## 沉思

#### [> >上下文切换的代价](https://web.archive.org/web/20220521223409/http://www.petrikainulainen.net/software-development/processes/the-cost-of-context-switching/)

我认为我们都高估了我们的上下文切换能力——我们越早接受切换是开发人员的克星——我们就能越早做些什么。这篇文章很好地提醒了这一事实。