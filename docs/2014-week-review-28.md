# Baeldung 每周评论 28

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/2014-week-review-28>

在 2014 年初，我决定开始更好地记录我的阅读习惯，并在这里与大家分享。

重点是双重的——通过策展和记录，我的阅读变得更加有目的和多样化。此外，我相信好内容的监管会带来很多价值，帮助人们探索，让最好的东西上升到顶端。

希望您会喜欢我们在 2014 年下半年发布的这些内容。

**开始了……**

## 1。Java

#### [> >编写数据访问代码测试——绿色构建不够好](https://web.archive.org/web/20220707143817/http://www.petrikainulainen.net/programming/testing/writing-tests-for-data-access-code-green-build-is-not-good-enough/)

本周回顾的良好开端——Petri 网回顾了一些测试应用程序的良好实践。关于事务性的第三点尤其重要——不要用不同的事务语义进行测试。

也挺搞笑的:“我们有两个选择:正确的和错误的”。

#### [> >特殊化状态](https://web.archive.org/web/20220707143817/http://cr.openjdk.java.net/~briangoetz/valhalla/specialization.html)

Java 中出现了超越原语的泛型(或许还有具体化)——Brian Goetz 发表了一个非常早期的提议，绝对值得一读。

#### [> >从 JPA 到 Hibernate 的遗留和增强标识符生成器](https://web.archive.org/web/20220707143817/http://vladmihalcea.com/2014/07/15/from-jpa-to-hibernates-legacy-and-enhanced-identifier-generators/)

接下来——关于用 JPA 生成标识符的**——这个系列将是深入研究 Hibernate 和 JPA 持久性的绝佳资源。**

#### [>>Java 编写自定义收集器入门 8](https://web.archive.org/web/20220707143817/http://www.nurkiewicz.com/2014/07/introduction-to-writing-custom.html)

谁说 Java 8 是在贬低番石榴----------------------------------------------------------------------------《收藏家》杂志的第一版第一集第一集第一集第一集就展示了他们可以很好地合作。

#### [> >将递归文件系统遍历转化为流](https://web.archive.org/web/20220707143817/http://www.nurkiewicz.com/2014/07/turning-recursive-file-system-traversal.html)

**优雅地使用流**来展平和遍历文件系统上的目录。

## 2。弹簧

#### [> >我的测试应该是@事务性的吗？](https://web.archive.org/web/20220707143817/https://www.marcobehler.com/2014/06/25/should-my-tests-be-transactional)

这是我现在做事情与几年前相比有很大不同的领域之一。**我对测试和@Transactional 的看法是否定的**

为什么不呢？有几个原因——第一，我发现让测试使用我的系统和我的 API 使用与它们在生产中实际使用的相同的事务语义是有价值的；改变这些将会使事情变得微妙不同——**,而测试中的微妙差异——以我的经验来看——并不好**。

**`no`** 背后的第二个原因是，知道我将有垃圾数据和我的测试套件运行的结束，使我以某种方式编写测试逻辑，真正考虑一些场景，并且总的来说，对测试的灵活性有积极的影响。

但这是我自己的偏好，正如 Marco 在文章开头所说的—**视情况而定**。总而言之，这是一篇值得一读的好文章。

#### [> > SpEL 支持春季数据 JPA @查询定义](https://web.archive.org/web/20220707143817/https://spring.io/blog/2014/07/15/spel-support-in-spring-data-jpa-query-definitions)

nuff 说，春季数据只是变得稍微冷了一点。

#### [> >春天数据休息现在附带阿尔卑斯元数据](https://web.archive.org/web/20220707143817/https://spring.io/blog/2014/07/14/spring-data-rest-now-comes-with-alps-metadata)

**ALPS 元数据对我来说是新的**，这篇文章让它看起来很有趣——也许从超媒体类型标准化的缓慢步伐中向前迈进了一步。

值得一看——并且可能需要一些挖掘来真正了解这种元数据可以为 API 做些什么(我计划在周末进行挖掘)。

#### [> > Spring 工具套件和 Groovy/Grails 工具套件 3.6.0 发布](https://web.archive.org/web/20220707143817/https://spring.io/blog/2014/07/11/spring-tool-suite-and-groovy-grails-tool-suite-3-6-0-released)

**新的 STS 发布了**——因为它是我每天都要使用的工具，所以我很快就升级了；如果你在 Eclipse 上做 Spring——真的没有理由不试一试。

## 3。技术和思考

#### [> > TDD 棋局第九部分:天佑女王](https://web.archive.org/web/20220707143817/http://www.daedtech.com/tdd-chess-game-part-9-god-save-the-queen)

虽然我还没有机会看到这一部分，但我从一开始就在我的每周评论中涵盖了整个系列，所以我毫不犹豫地推荐它，甚至在我这个周末看到它之前。这个系列是`jam packed`——老实说，它可能应该被产品化和销售——但因为它是免费的——浏览它，你会学到很多。

#### [> >面试中不该做的事，第二部分:面试官版](https://web.archive.org/web/20220707143817/http://dandreamsofcoding.com/2014/07/16/interviewing-is-hard-2/)

很好地延续了第一篇面试技巧文章——我参加过面试，我个人发现当面试官(对我来说)要困难得多。要成为一名还算过得去的面试官还有很长很长的路要走——当你处于那个位置时，这是一份需要记住的体面清单。