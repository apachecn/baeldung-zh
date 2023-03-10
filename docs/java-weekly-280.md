# Java 周刊，第 280 期

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-weekly-280>

**开始了……**

## 1。Spring 和 Java

#### [**> >多缓存配置带咖啡因和 Spring Boot**T3【techblog.bozho.net】和](https://web.archive.org/web/20220629011814/https://techblog.bozho.net/multiple-cache-configurations-with-caffeine-and-spring-boot/)

一个新颖的`CaffeineCacheManager`扩展让你**配置不同规格的缓存，全部由同一个`CacheManager`** 管理。非常酷。

#### [**> >运行科特林测试与格拉德**](https://web.archive.org/web/20220629011814/https://www.petrikainulainen.net/programming/testing/running-kotlin-tests-with-gradle/)【petrikainulainen.net

通过一点配置，您可以在一个 Gradle 构建期间在 Kotlin 中运行单元测试和集成测试——或者单独运行。

#### [**> > Eclipse 和 Oracle 无法就 javax 包名称空间和商标的条款达成一致**](https://web.archive.org/web/20220629011814/https://www.infoq.com/news/2019/05/end-of-javax-package)【infoq.com

一个令人难以置信的决定导致**明显背离了 Java SE 和 EE 兼容性的悠久历史**。还有[关于发展情况的一些常见问题](https://web.archive.org/web/20220629011814/https://eclipse-foundation.blog/2019/05/08/jakarta-ee-8-faq/)。

#### 同样值得一读:

*   #### [**>有感于雅加达电子工程师包改名**](https://web.archive.org/web/20220629011814/https://blog.sebastian-daschner.com/entries/thoughts-on-jakarta-package-name)

*   #### [**> > Chunyun introduces pluggable circuit breaker interface**](https://web.archive.org/web/20220629011814/https://www.infoq.com/news/2019/05/spring-cloud-pluggable-circuit) [infoq.com]

*   #### [**> > How to get the exposure dormancy statistics**](https://web.archive.org/web/20220629011814/https://vladmihalcea.com/hibernate-statistics-jmx/)

    through JMX [ vladmihalcea.com
*   #### [**> > 5 分钟以内：带有（同 JavaMessageService）Java 消息服务队列和主题的 ActiveMQ**](https://web.archive.org/web/20220629011814/https://tomitribe4.wpengine.com/blog/5-minutes-or-less-activemq-with-jms-queues-and-topics/)【tomi tribe 4 . wpengine . com】T5

*   #### [**> >创意教育工具:直接在你的集成驱动电子设备中学习**](https://web.archive.org/web/20220629011814/https://www.vojtechruzicka.com/idea-edu-tools/)【vojtechruzicka . com】T5

*   #### [**> > NetBeans 晋级顶级街头流氓项目**](https://web.archive.org/web/20220629011814/https://www.infoq.com/news/2019/05/apache-netbeans?utm_campaign=infoq_content&utm_source=infoq&utm_medium=feed&utm_term=Java)【infoq . com】T5

#### 网络研讨会和演示:

*   #### [**>超越 Java 语言(一种计算机语言，尤用于创建网站)的生活 8**T3【infoq . com】T5T6](https://web.archive.org/web/20220629011814/https://www.infoq.com/presentations/java-8-plus)

*   #### [**> > Spring tip: the number of page views of reactive power**](https://web.archive.org/web/20220629011814/https://spring.io/blog/2019/05/08/spring-tips-reactive-web-views) [ spring.io ]

*   #### [**>一个精彩的播客:春云工程师奥尔加·马恰泽克-夏尔马**](https://web.archive.org/web/20220629011814/https://spring.io/blog/2019/05/03/a-bootiful-podcast-spring-cloud-engineer-olga-maciaszek-sharma)春天。io

*   #### [**> > Extraordinary Java:**](https://web.archive.org/web/20220629011814/https://www.infoq.com/presentations/java-science-aerospace) [infoq.com]

    to the moon and beyond
*   #### [**> > Graal:不仅仅是 JVM**](https://web.archive.org/web/20220629011814/https://www.infoq.com/presentations/graal-jit-c2)infoq . com的新 JIT

*   #### [**> > Yuga Byte DB-a planetary database**](https://web.archive.org/web/20220629011814/https://www.infoq.com/presentations/yugabytedb) [infoq.com]

    for low-latency trading applications
*   #### [**> > Many DevSecOps tools just apply lipstick to an old pig**](https://web.archive.org/web/20220629011814/https://www.infoq.com/presentations/evaluate-devsecops-tools) [infoq.com]

#### 升级时间:

*   #### [**>适用于阿帕奇晶洞的 Spring Boot &关键的宝石 1。0 .0 .释放；排放；发布现已发布！**](https://web.archive.org/web/20220629011814/https://spring.io/blog/2019/05/07/spring-boot-for-apache-geode-pivotal-gemfire-1-0-0-release-available) 春天。io

*   #### [**> >科特林 1.3.30 带来了科特林语/本土语和 KAPT 的改进,以及更多的**](https://web.archive.org/web/20220629011814/https://www.infoq.com/news/2019/04/kotlin-1.3.30)【infoq . com】T5

*   #### [**>宣布 OCI-grad le-插件版本 0 .1 .0**](https://web.archive.org/web/20220629011814/http://andresalmiray.com/announcing-oci-gradle-plugin-version-0-1-0/)【andresalmiray . com】T5

## 2。技术和思考

#### [**> >幸存开源漏洞的频率**](https://web.archive.org/web/20220629011814/https://www.tomitribe.com/blog/surviving-the-frequency-of-open-source-vulnerabilities/)【tomitribe.com】

据估计，所有网站中有一半存在严重的安全漏洞，没有一家公司能够免受网络攻击。

#### [**>>cloud formation CLI 工作流程**](https://web.archive.org/web/20220629011814/https://advancedweb.hu/2019/05/07/cf_workflows/)[advanced web . Hu]

尽管通过控制台管理堆栈是乏味的，但是一些基本的工具和脚本可以减轻一些痛苦。

#### 同样值得一读:

*   #### [> >用空手道】](https://web.archive.org/web/20220629011814/https://vanwilgenburg.wordpress.com/2019/05/03/writing-integration-tests-for-cors-headers-with-karate/)

*   #### [> > Write testable code](https://web.archive.org/web/20220629011814/https://medium.com/feedzaitech/writing-testable-code-b3201d4538eb) [medium.com]

*   #### [**> > Multiple given, time and time**](https://web.archive.org/web/20220629011814/https://lizkeogh.com/2019/05/06/on-multiple-givens-whens-and-thens/) [lizkeogh.com]

*   #### [>谷歌宣布云代码：将 IntelliJ 和可视化工作室代码扩展到 kubernetes Apps](https://web.archive.org/web/20220629011814/https://www.infoq.com/news/2019/05/google-cloud-code)【infoq.comT4

*   #### [**>>**](https://web.archive.org/web/20220629011814/https://blog.frankel.ch/cherry-pick-automation-bash/)博客。弗兰克尔。ch

*   #### [**> > Let's replace the term "technical debt" with**](https://web.archive.org/web/20220629011814/https://morethancoding.com/2019/05/08/lets-replace-the-term-technical-debt/) [morethancoding.com]

*   #### [**> > End-to-end testing of Web applications: painless way**](https://web.archive.org/web/20220629011814/https://mtlynch.io/painless-web-app-testing/) [ mtlynch.io

## 3。漫画

本周我最喜欢的迪尔伯特。

#### [>>](https://web.archive.org/web/20220629011814/https://dilbert.com/strip/2019-05-08)【dilbert.com】

#### [> >杜伯特叙述](https://web.archive.org/web/20220629011814/https://dilbert.com/strip/2019-05-07)【dilbert.com】

#### [> >工程师不会说谎](https://web.archive.org/web/20220629011814/https://dilbert.com/strip/2019-05-04)【dilbert.com】

## 4。本周精选

#### [> >保护自己免遭身份盗窃](https://web.archive.org/web/20220629011814/https://www.schneier.com/blog/archives/2019/05/protecting_your_2.html)【schneier.com】

下一期[Java 周刊，第 281 期](/web/20220629011814/https://www.baeldung.com/java-weekly-281)上一期[Java 周刊，第 279 期](/web/20220629011814/https://www.baeldung.com/java-weekly-279)