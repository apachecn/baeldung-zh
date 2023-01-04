# Baeldung 每周评论 44

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/2014-week-review-44>

2014 年初，我决定开始追踪我的阅读习惯，并在 Baeldung 上分享最好的东西。

管理我的阅读使它更有目的和多样化——我也希望通过让本周最好的内容上升到顶部来为你提供价值。

**开始了……**

## 1。Java

#### [> >可选 Java SE 8](https://web.archive.org/web/20220521214956/http://blog.joda.org/2014/11/optional-in-java-se-8.html)

这是一个有用的指南，指导你如何使用新的`Optional`类，当它被引入到语言中时，它意味着。和其他东西一样，有一个很好的使用方法，然后还有其他所有的方法。

#### [>>Java 中更好的 null 10？](https://web.archive.org/web/20220521214956/http://blog.joda.org/2014/11/better-nulls-in-java-10.html)

JDK 10 号还有很长的路要走，所以现在是进行思想实验的时候了。这是其中之一——很有用的一点是，它清楚地表明了 Java 中`null`的语义绝对可以改进并且应该改进。

#### [> >关于 Java 你不知道的 10 件事](https://web.archive.org/web/20220521214956/http://blog.jooq.org/2014/11/03/10-things-you-didnt-know-about-java/)

Java 中很酷的角落案例和惊喜列表——即使你已经从事 Java 多年，这也是一本有趣的读物。

#### [> >关于 Java 泛型和擦除](https://web.archive.org/web/20220521214956/http://techblog.bozho.net/on-java-generics-and-erasure/)

Java 中类型擦除如何工作的快速教育探索。

#### [> >超越线程池:Java 并发没有你想象的那么差](https://web.archive.org/web/20220521214956/http://www.takipiblog.com/beyond-thread-pools-java-concurrency-is-not-as-bad-as-you-think/)

Java 并发生态系统的一个非常高层次的图表—**一些可用的解决方案和范例，帮助您开始**并了解您的选择。

#### [> >冬眠收藏乐观锁定](https://web.archive.org/web/20220521214956/http://vladmihalcea.com/2014/11/01/hibernate-collections-optimistic-locking/)

对 Hibernate 的又一次深入探讨——这一次向**展示了父子关联的建模对于**并发和可靠地访问数据的重要性。

还有一些其他的**版本和公告**，我很兴奋，或者在更广泛的 Java 生态系统中值得注意:

*   #### [弹性搜索 1.4.0 和 1.3.5 发布](https://web.archive.org/web/20220521214956/http://www.elasticsearch.org/blog/elasticsearch-1-4-0-released/)

*   #### [Elastic Search Shield: You know, for safety (coming soon)](https://web.archive.org/web/20220521214956/http://www.elasticsearch.org/blog/shield-know-security-coming-soon/)

*   #### [IntelliJ 理念 14 发布！](https://web.archive.org/web/20220521214956/http://blog.jetbrains.com/idea/2014/11/intellij-idea-14-is-released/)

## 2。弹簧

#### [> >使用 Logstash、Elasticsearch 和 Kibana](https://web.archive.org/web/20220521214956/https://blog.codecentric.de/en/2014/10/log-management-spring-boot-applications-logstash-elastichsearch-kibana/) 对 Spring Boot 应用程序进行日志管理

**麋鹿是个美丽的东西**。我已经用了一段时间了，它很棒。

#### [> > Spring 缓存抽象和 Google 番石榴缓存](https://web.archive.org/web/20220521214956/http://www.java-allandsundry.com/2014/10/spring-caching-abstraction-and-google.html)

使用番石榴缓存为 Spring 应用中的缓存提供动力是非常有意义的。这是怎么回事。

#### [> > A 质量@限定词](https://web.archive.org/web/20220521214956/https://spring.io/blog/2014/11/04/a-quality-qualifier)

我主要使用`@Qualifier`作为面试问题，但有时它可以将一个棘手的情况变成一个优雅的解决方案。正如乔希指出的那样，这种情况已经持续了多年。

#### [>](https://web.archive.org/web/20220521214956/http://www.petrikainulainen.net/programming/spring-framework/spring-from-the-trenches-resetting-auto-increment-columns-before-each-test-method/)

一个有趣的深入研究**使用一个大型集成测试套件**——如何确保您的结果是正确的和可重复的，并且您的测试是等幂的。

最后——春季的一些很酷的**发布和网络研讨会:**

*   #### [> > Spring data Evans SR1 released](https://web.archive.org/web/20220521214956/https://spring.io/blog/2014/11/03/spring-data-evans-sr1-released)

*   #### [> > Chunyun 1.0.0.M2 has been listed](https://web.archive.org/web/20220521214956/https://spring.io/blog/2014/11/05/spring-cloud-1-0-0-m2-available-now)

*   #### [> > Webinar playback: Build a "vibrant" user interface with Spring Boot and Wading](https://web.archive.org/web/20220521214956/https://spring.io/blog/2014/11/04/webinar-replay-building-bootful-uis-with-spring-boot-and-vaadin)

*   #### [> > Webinar playback: Web and mobile applications supported by content using Spring, Groovy and Crafter](https://web.archive.org/web/20220521214956/https://spring.io/blog/2014/10/30/webinar-replay-content-enabled-web-and-mobile-applications-with-spring-groovy-and-crafter)

## 3。技术和思考

#### [> >空行是码闻](https://web.archive.org/web/20220521214956/http://www.yegor256.com/2014/11/03/empty-line-code-smell.html)

`**“A method should do one thing”**`。过了很久，我才真正内化了这个事实，开始在自己的设计中积极寻找。

所以我在这种背景下读了这篇文章，重点是改进我自己的设计。这也是我在这里分享的方式。

#### [> >如何让你的公司停止杀猫](https://web.archive.org/web/20220521214956/http://www.daedtech.com/how-to-get-your-company-to-stop-killing-cats)

我们都有自己的战争故事。但是让一群人去改变是一件很糟糕的事情——这就是为什么当事情变得更好的时候，我觉得很酷。

虽然不常发生。

#### [> >收集管道](https://web.archive.org/web/20220521214956/http://martinfowler.com/articles/collection-pipeline/)

一个非常好的关于功率和宽度收集流水线操作的拼凑。让我今天想做点 Clojure。

## 4。漫画

现在你实际上正在阅读我的每周评论——XKCD:

#### [> >星级](https://web.archive.org/web/20220521214956/https://xkcd.com/1098/)

#### [> >转到](https://web.archive.org/web/20220521214956/https://xkcd.com/292/)

#### [> >墓地](https://web.archive.org/web/20220521214956/https://xkcd.com/736/)

## 5。本周精选

我最近在我的“每周评论”中介绍了“本周精选”部分。有趣的是，这完全是我的电子邮件列表订阅者的专属。

所以——如果你是从我的邮件列表中看到这篇文章的，你已经有选择了——希望你喜欢。如果没有，请随意订阅，你会得到下一个。