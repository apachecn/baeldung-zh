# Java 周刊，第 239 期

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-weekly-239>

**开始了……**

## 1。Spring 和 Java

#### [**> >炼制功能性泉水**](https://web.archive.org/web/20220926201311/https://blog.frankel.ch/refining-functional-spring/)blog . frankel . ch

在这个令人兴奋的**Spring Boot**的新功能方法中，快速写一篇关于编写处理程序和路线的一些细微差别的文章。

#### [**> >利用这些先进的 GC 技术提高应用程序性能**](https://web.archive.org/web/20220926201311/https://blog.takipi.com/improve-your-application-performance-with-garbage-collection-optimization/)blog.takipi.com

关于 JVM 中垃圾收集如何工作的坚实基础，以及可以用来提高应用程序性能的一些技巧。好东西。

#### [**> >当所有子代都必须匹配过滤条件时如何查询父行用 SQL 和 Hibernate**[【vladmihalcea.com】T4](https://web.archive.org/web/20220926201311/https://vladmihalcea.com/sql-query-parent-rows-all-children-match-filtering-criteria/)

这是一个很好的教程，它逐步构建了这个问题的最佳解决方案，首先是一个原生 SQL 查询，然后是一个基于 JPQL 标准的查询。非常酷。

#### [**> >只修改了詹金斯**中的文件](https://web.archive.org/web/20220926201311/http://blog.code-cop.org/2018/07/only-modified-files-in-jenkins.html)【blog.code-cop.org】中的

还有一种有趣的方法，它使用一个 Groovy 脚本来识别自上次绿色构建以来发生变化的所有文件。

#### 同样值得一读:

*   #### [**>将自动警报系统参数存储与春云**【【blog.trifork.com】T4](https://web.archive.org/web/20220926201311/https://blog.trifork.com/2018/07/20/integrating-the-aws-parameter-store-with-spring-cloud/)

*   #### [**> >可选。isempty()可在 JDK 11 个电子艺界游戏公司构建**T3【Marx software . blogspot . com】和](https://web.archive.org/web/20220926201311/https://marxsoftware.blogspot.com/2018/07/optional-isempty-jdk-11.html)

*   #### [> > lazy evaluation by Lambda expression](https://web.archive.org/web/20220926201311/http://4comprehension.com/leveraging-lambda-expressions-for-lazy-evaluation-in-java/) [4comprehension.com]

*   #### [**> > IntelliJ 理念中的绝妙的改进 2018.2**](https://web.archive.org/web/20220926201311/https://blog.jetbrains.com/idea/2018/07/groovy-improvements-in-intellij-idea-2018-2/)【blog . jetbrains . com】T5

*   #### [**> >冬眠提示：如何在查询中使用@元素集合条目**](https://web.archive.org/web/20220926201311/https://www.thoughts-on-java.org/hibernate-tips-query-elementcollection/)

*   #### [**> > K Native: a powerful building module of portable functional platform**](https://web.archive.org/web/20220926201311/https://content.pivotal.io/blog/knative-powerful-building-blocks-for-a-portable-function-platform) [ content.pivotal.io

**网络研讨会和演示:**

*   #### [> > Valid Java, 3rd Edition-Keep it valid](https://web.archive.org/web/20220926201311/https://www.infoq.com/presentations/effective-java-third-edition) [infoq.com]

*   #### [**> > Java 11-Keep the Java release train on the right track**](https://web.archive.org/web/20220926201311/https://www.infoq.com/presentations/java-10-11) [infoq.com]

*   #### [**>样本课:测试容器简介**](https://web.archive.org/web/20220926201311/https://www.petrikainulainen.net/programming/testing/sample-lesson-introduction-to-testcontainers/)

**升级时间:**

*   #### [**> > Spring Cloud Data Stream 1.6 RC1 was released**](https://web.archive.org/web/20220926201311/https://spring.io/blog/2018/07/23/spring-cloud-data-flow-1-6-rc1-released) Spring.io

*   #### [**> > Spring Festival travel rush documents 2.0.2\. Release**](https://web.archive.org/web/20220926201311/https://spring.io/blog/2018/07/19/spring-rest-docs-2-0-2-release) spring.io and [**> > Spring Festival travel rush documents 1.2.5.release**](https://web.archive.org/web/20220926201311/https://spring.io/blog/2018/07/19/spring-rest-docs-1-2-5-release)

*   #### [**> >休眠验证器 6。0 .11 .最终发布**](https://web.archive.org/web/20220926201311/http://in.relation.to/2018/07/18/hibernate-validator-6011-final-out/)

*   #### [**> > dependency management plug-in 1.0.6\. Release**](https://web.archive.org/web/20220926201311/https://spring.io/blog/2018/07/18/dependency-management-plugin-1-0-6-release) spring.io

*   #### [**>加特林 JDBC 1 号。0 .0 版本**](https://web.archive.org/web/20220926201311/https://blog.codecentric.de/en/2018/07/gatling-jdbc-release-1-0-0/)博客。代码中心。德

*   #### [**> > Dormancy Search Third Maintenance Release 5.10**](https://web.archive.org/web/20220926201311/http://in.relation.to/2018/07/25/hibernate-search-5-10-3-Final/)

*   #### [**> > Hibernation ORM 5.3.3\. Finally released**](https://web.archive.org/web/20220926201311/http://in.relation.to/2018/07/24/hibernate-orm-533-final-out/)

*   #### [**> > Kafka 1.0.1\. Release**](https://web.archive.org/web/20220926201311/https://github.com/reactor/reactor-kafka/releases/tag/v1.0.1.RELEASE) [github.com]

## 2。技术和思考

#### [**> >在不停机的情况下更新您的数据库模式**](https://web.archive.org/web/20220926201311/https://www.thoughts-on-java.org/update-database-schema-without-downtime/)【thoughts-on-java.org】

如果你绝对不能承受停机时间，这里有一些**伟大的策略，当推出非向后兼容的模式更新**与应用程序更新。

#### [**>>web assembly 的未来——看看即将推出的功能和提案**](https://web.archive.org/web/20220926201311/https://blog.scottlogic.com/2018/07/20/wasm-future.html)【blog.scottlogic.com】

看起来这个基于浏览器的 VM 很快会有一些重要的增强，包括引用类型、异常处理和垃圾收集。

#### 同样值得一读:

*   #### [>教练接战三法则](https://web.archive.org/web/20220926201311/http://blog.code-cop.org/2018/07/three-rules-of-coaching-engagement.html)【blog . code-COP . org】T5

*   #### [**>黑盒子记——阿拉戈克** T3、【codemonkey ism . co . uk】T4](https://web.archive.org/web/20220926201311/https://codemonkeyism.co.uk/htb-aragog/)

## 3。漫画

本周我最喜欢的迪尔伯特。

#### [**> >如何成为工程师**](https://web.archive.org/web/20220926201311/http://dilbert.com/strip/2018-07-21/)【dilbert.com】

#### [**> >呆伯特加入门萨**](https://web.archive.org/web/20220926201311/http://dilbert.com/strip/1992-02-05/)【dilbert.com

#### [**> >升级会有风险**](https://web.archive.org/web/20220926201311/http://dilbert.com/strip/1999-07-03/)【dilbert.com】

## 4。本周精选

#### [**> >为什么 Json 不是一个好的配置语言**](https://web.archive.org/web/20220926201311/https://www.lucidchart.com/techblog/2018/07/16/why-json-isnt-a-good-configuration-language/)【lucidchart.com】

下一期[Java 周刊，第 240 期](/web/20220926201311/https://www.baeldung.com/java-weekly-240)上一期[Java 周刊，第 238 期](/web/20220926201311/https://www.baeldung.com/java-weekly-238)