# Java 周刊，第 196 期

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-weekly-196>

本周有很多关于 Java 9 的有趣文章。

**开始了……**

## 1。Spring 和 Java

#### [**> > Java 9 和 IntelliJ IDEA**](https://web.archive.org/web/20220524021714/https://blog.jetbrains.com/idea/2017/09/java-9-and-intellij-idea/)【blog.jetbrains.com】

很高兴看到工具适应新版本的速度非常快🙂

#### [**>>Java 中的 Lambda 表达式偷偷抛出异常**](https://web.archive.org/web/20220524021714/http://4comprehension.com/sneakily-throwing-exceptions-in-lambda-expressions-in-java/)【4comprehension.com】

Java 8 也对类型推断做了一些小的修改——类型推断**可以用来将检查异常伪装成运行时异常。**

知道这一点当然很好，但是在使用时要小心。

#### [**>>JVM 着火——用火焰图分析性能**](https://web.archive.org/web/20220524021714/https://blog.codecentric.de/en/2017/09/jvm-fire-using-flame-graphs-analyse-performance/)[blog . code centric . de]

看起来火焰图可以成为标准剖析视图的更具可读性的替代品。

#### [**> >测试微服务—Java&Spring Boot**[hamvocke.com](https://web.archive.org/web/20220524021714/http://www.hamvocke.com/blog/testing-java-microservices/)

Spring Boot 微服务测试综合指南。

#### [**>>Java Web 应用安全登录**](https://web.archive.org/web/20220524021714/https://techblog.bozho.net/securelogin-java-web-applications/)【techblog.bozho.net

SecureLogin 协议是保护 web 应用程序的一个很有前途的新选项。

#### [**> > RebelLabs 开发者生产力报告 2017:你为什么使用你所使用的 Java 工具？**](https://web.archive.org/web/20220524021714/https://zeroturnaround.com/rebellabs/developer-productivity-report-2017-why-do-you-use-java-tools-you-use/)【zeroturnaround.com】

了解我们实际使用的工具的用法总是非常有趣的。

#### 也值得再 **阿丁:**

*   #### [> >冬眠类开源项目诞生](https://web.archive.org/web/20220524021714/https://vladmihalcea.com/2017/09/25/the-hibernate-types-open-source-project-is-born/)

*   [ Brian Goetz interviewed InfoQ [infoq.com about pattern matching of Java.
*   #### [> > Java 9 支持 Eclipse IDE，氧气版](https://web.archive.org/web/20220524021714/https://waynebeaton.wordpress.com/2017/09/22/java-9-support-for-eclipse-ide-oxygen-edition/)【Wayne beaton . WordPress . com】

*   #### [> >兰达斯和洁码](https://web.archive.org/web/20220524021714/https://blog.frankel.ch/lambdas-clean-code/#gsc.tab=0)博客。弗兰克尔。ch

*   #### [**> > Everything you need to know about Java 9**](https://web.archive.org/web/20220524021714/http://blog.takipi.com/everything-you-need-to-know-about-java-9/) [blog.takipi.com]

*   #### [**> >爪哇命令行接口（第十六部分):JArgp**](https://web.archive.org/web/20220524021714/https://marxsoftware.blogspot.com/2017/09/jargp.html)[【Marx software . blogspot . com】T4

*   #### [**>在 Java 9 中用-Xlog 选项统一登录**](https://web.archive.org/web/20220524021714/https://blog.codefx.org/java/unified-logging-with-the-xlog-option/)【blog . codefx . org】

*   #### [**> > Keep your code base healthy**](https://web.archive.org/web/20220524021714/http://andresalmiray.com/keeping-your-codebase-healthy/) [andresalmiray.com

**网络研讨会和演示:**

*   #### [> > Live Webinar: Real World Java 9](https://web.archive.org/web/20220524021714/https://blog.jetbrains.com/idea/2017/09/live-webinar-real-world-java-9/) [blog.jetbrains.com

*   #### [**>网络直播：无功泉**【【blog.jetbrains.com】T4](https://web.archive.org/web/20220524021714/https://blog.jetbrains.com/idea/2017/09/live-webinar-reactive-spring/)

*   #### [**> > Spring tip: reactive web sockets with spring frame 5**](https://web.archive.org/web/20220524021714/https://spring.io/blog/2017/09/27/spring-tips-reactive-websockets-with-spring-framework-5) spring.io

*   #### [**> > Java 9 专家内幕**](https://web.archive.org/web/20220524021714/https://www.oracle.com/java/java9-screencasts.html) 【oracle.com】

*   #### [**> > JavaOne 2017:你不想错过的 12 强会议**](https://web.archive.org/web/20220524021714/http://blog.takipi.com/javaone-2017-the-top-12-sessions-you-dont-want-to-miss/)【blog . taki pi . com】

**升级时间:**

*   #### [**> > Java platform, standard version of Oracle Bone Inscriptions new function JDK9** T3 [docs.oracle.com] T5]](https://web.archive.org/web/20220524021714/https://docs.oracle.com/javase/9/whatsnew/toc.htm#JSNEW-GUID-C23AFD78-C777-460B-8ACE-58BE5EA681F6)

*   #### [**> > JDK 9 日发布**](https://web.archive.org/web/20220524021714/https://marxsoftware.blogspot.com/2017/09/jdk-9-released-today.html)【Marx software . blogspot . com】T5

*   #### [**>春云任务发布现已发售**](https://web.archive.org/web/20220524021714/https://spring.io/blog/2017/09/25/spring-cloud-task-1-2-2-release-is-now-available) 春天。io

*   #### [**> >春天 AMQP 2.0 发布候选 2 可用**](https://web.archive.org/web/20220524021714/https://spring.io/blog/2017/09/27/spring-amqp-2-0-release-candidate-2-available) 春天。io

## 2。技术

#### [**> >礼貌的 HTTP API 设计——“使用头，卢克！”**](https://web.archive.org/web/20220524021714/https://blog.codecentric.de/en/2017/09/polite-http-api-design-use-the-headers-luke/)[blog . code centric . de]

按照规范的意图，使用正确的 HTTP 头总是一个好主意🙂

**同样值得一读:**

*   #### [**>不复杂 TimeDivisionDuplex 时分双工和 BDD** T3【testerstories.com】和](https://web.archive.org/web/20220524021714/http://testerstories.com/2017/09/uncomplicate-tdd-and-bdd/)

## 3。沉思

#### [**> >低风险单体向微服役演进第一部分**](https://web.archive.org/web/20220524021714/http://blog.christianposta.com/microservices/low-risk-monolith-to-microservice-evolution/)【blog.christianposta.com】

一篇很酷的关于**将你的 monolith 重构到微服务架构时的风险最小化的文章。**

**同样值得一读:**

*   #### [>>As-Salaam-Alaikum:云到中东！](https://web.archive.org/web/20220524021714/http://www.allthingsdistributed.com/2017/09/aws-region-middle-east.html)【all things distributed . com】

*   #### [>动词代替名词](https://web.archive.org/web/20220524021714/http://blog.code-cop.org/2017/09/verbs-instead-of-nouns.html)【blog . code-COP . org】T3

*   #### [**>塞拉朱丽叶狐步舞**](https://web.archive.org/web/20220524021714/http://blog.cleancoder.com/uncle-bob/2017/09/26/SierraJulietFoxtrot.html)blog . clean coder . com】T4

*   #### [**> > How many unit tests are enough?**](https://web.archive.org/web/20220524021714/https://www.daedtech.com/unit-testing-enough/) 【 daedtech.com】

*   #### [**> > Ten Elements of Excellent API Documents**](https://web.archive.org/web/20220524021714/https://alistapart.com/article/the-ten-essentials-for-good-api-documentation) [alistapart.com

## 4。漫画

本周我最喜欢的迪尔伯特。

#### [**> >杀魂任务**](https://web.archive.org/web/20220524021714/http://dilbert.com/strip/2017-09-01)【dilbert.com】

#### [**> >估计时间线**](https://web.archive.org/web/20220524021714/http://dilbert.com/strip/2017-08-19)【dilbert.com】

#### [**> >理想客户**](https://web.archive.org/web/20220524021714/http://dilbert.com/strip/2017-08-17)【dilbert.com】

## 5。本周精选

#### [> >在效率时代创业](https://web.archive.org/web/20220524021714/https://www.daedtech.com/starting-tech-firm-efficiencer/)【daedtech.com】