# Java 周刊，第 246 期

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-weekly-246>

**开始了……**

## 1。Spring 和 Java

#### [**> > Bootiful GCP:用春云 GCP stack driver Trace(6/8)**](https://web.archive.org/web/20220524031644/https://spring.io/blog/2018/09/06/bootiful-gcp-supporting-observability-with-spring-cloud-gcp-stackdriver-trace-6-8)[Spring . io]

#### [**> > Bootiful GCP:使用 Spring 云 GCP 连接其他 GCP 服务(7/8)**](https://web.archive.org/web/20220524031644/https://spring.io/blog/2018/09/10/bootiful-gcp-use-spring-cloud-gcp-to-connect-to-other-gcp-services-7-8)[Spring . io]

简要看一下使用 Spring Cloud Sleuth 的分布式跟踪，以及如何使用直接 Java SDK 或 REST API 使用另一个 GCP 服务的快速示例。非常酷。

#### [**> >用 StackWalker 和 Stream API 在 Java 中进行 stack walking**](https://web.archive.org/web/20220524031644/https://4comprehension.com/stackwalking-in-java-with-stackwalker-and-stream-api/)【4comprehension.com】

这是对 JEP-259 堆栈审核 API 的一个很好的介绍，它允许您使用 Stream API 来**缓慢地遍历堆栈跟踪。**

#### [**>>JUnit 5.3 新增功能**](https://web.archive.org/web/20220524031644/https://medium.com/@BillyKorando/whats-new-in-junit-5-3-c276eb8507f1)【medium.com】

JUnit 5 中最新特性的概述，包括并行测试执行，最后，**对 maven surefire 和 failsafe 插件的本机支持**。

#### [**> >线程池自诱导死锁**](https://web.archive.org/web/20220524031644/https://www.nurkiewicz.com/2018/09/thread-pool-self-induced-deadlocks.html)【nurkiewicz.com】

一篇关于死锁的可靠文章，外加一个场景，展示了线程池的不正确使用如何容易导致死锁。

#### Java 中的 [**> >基于属性的测试:状态测试**](https://web.archive.org/web/20220524031644/https://blog.johanneslink.net/2018/09/06/stateful-testing/)【blog.johanneslink.net】

还有一种测试应用程序状态的巧妙方法，其中您**将预期行为建模为有限状态机，然后检查不变量和后置条件是否成立**。好东西。

#### 同样值得一读:

*   #### [**>从 Java 8 到 Java 11**](https://web.archive.org/web/20220524031644/https://blog.joda.org/2018/09/from-java-8-to-java-11.html)【blog . joda . org】T5

*   #### [**> > JDK 12:开关语句/表情在行动**](https://web.archive.org/web/20220524031644/https://marxsoftware.blogspot.com/2018/09/jdk12-switch-in-action.html)【Marx software . blogspot . com】T5

*   #### [**>如何将 PostgreSQL inet 类型与作业的装配区（JobPackArea）和冬眠**T3【vladimihalcea . com】和 进行映射](https://web.archive.org/web/20220524031644/https://vladmihalcea.com/postgresql-inet-type-hibernate/)

*   #### [**> > Push the pictures of the workers at Spring Boot Pier 2 to Amazon ECR** T3 [tech.asimio.net]No. T4](https://web.archive.org/web/20220524031644/https://tech.asimio.net/2018/09/05/Pushing-Spring-Boot-2-Docker-images-to-Amazon-ECR.html)

*   #### [**> >科特林支持春云功能**](https://web.archive.org/web/20220524031644/https://spring.io/blog/2018/09/11/kotlin-support-in-spring-cloud-function) 春天。io

*   #### [**> > JEP 342:JVM 与厉鬼**T3【Marx software . blogspot . com】T4](https://web.archive.org/web/20220524031644/https://marxsoftware.blogspot.com/2018/09/jep-342-java-and-spectre.html)

*   #### [**>>**](https://web.archive.org/web/20220524031644/https://medium.com/@jponge/the-graalvm-frenzy-f54257f5932c)【medium . com】

*   #### [**> > Elaborate Java strategy file, practical guide**](https://web.archive.org/web/20220524031644/https://blog.frankel.ch/jvm-security/3/) [ blog.frankel.ch ]

**升级时间:**

*   #### [**> >春天框架 5.1 RC3、5.0.9 和 4.3.19 现已上市**](https://web.archive.org/web/20220524031644/https://spring.io/blog/2018/09/07/spring-framework-5-1-rc3-5-0-9-and-4-3-19-available-now) 春天。io

*   #### [**> >跳马 2.1 跳马 RC**](https://web.archive.org/web/20220524031644/https://spring.io/blog/2018/09/10/spring-vault-2-1-goes-rc) 春天。io 和 [**> >跳马 2.0.2 发布**](https://web.archive.org/web/20220524031644/https://spring.io/blog/2018/09/10/spring-vault-2-0-2-released) 春天。io

*   #### [**> > Spring data released by ingels SR15 and Kai SR10**](https://web.archive.org/web/20220524031644/https://spring.io/blog/2018/09/10/spring-data-ingalls-sr15-and-kay-sr10-released) [ spring.io ]

*   #### [**> >春安 RC2 发布**](https://web.archive.org/web/20220524031644/https://spring.io/blog/2018/09/10/spring-security-5-1-0-rc2-released) 春天。io 和 [**> >春安 5.0.8 和 4.2.8 发布**](https://web.archive.org/web/20220524031644/https://spring.io/blog/2018/09/11/spring-security-5-0-8-and-4-2-8-released) 春天。io

*   #### [**> > Spring Boot 1。5 .16**](https://web.archive.org/web/20220524031644/https://spring.io/blog/2018/09/11/spring-boot-1-5-16) 春天。io

*   #### [**> >托米:托米 7.1 版本概述！**](https://web.archive.org/web/20220524031644/https://www.tomitribe.com/blog/2018/09/tomee-an-overview-of-the-tomee-7-1-release/)【tomi tribe . com】T5

*   #### [**>>DesktopPaneFX 0。11 .0 发布**](https://web.archive.org/web/20220524031644/http://andresalmiray.com/desktoppanefx-0-11-0-released/)【andresalmiray . com】T5

## 2。技术和思考

#### [**> >关于支持连续测试与 FITR 测试自动化(再版)**](https://web.archive.org/web/20220524031644/https://www.ontestautomation.com/on-supporting-continuous-testing-with-fitr-test-automation-republished/)【ontestautomation.com】

如果你想让你的自动化测试成为你的 CI/CD 策略中有价值的一部分，首先要确保它们是集中的、信息丰富的、可信的和可重复的——连续测试的四个支柱。

#### [**> > Keystone 实时流处理平台**](https://web.archive.org/web/20220524031644/https://netflixtechblog.com/keystone-real-time-stream-processing-platform-a3ee651812a)【medium.com】

对网飞 Keystone 平台的架构和设计原则的高度概述，以及在**大规模实施数据管道和 SPaaS 所面临的一些挑战**。

#### [**> >构建一次，随处运行:具体化你的配置**](https://web.archive.org/web/20220524031644/https://reflectoring.io/externalize-configuration/)[reflector ing . io]

一篇吹捧将配置参数从已部署的工件中分离出来的优点的好文章。

**同样值得一读:**

*   #### [**> >托米:升级很容易！**](https://web.archive.org/web/20220524031644/https://www.tomitribe.com/blog/tomee-upgrading-is-easy/)【tomi tribe . com】T5

*   #### [**>用逃犯。vim** [ 高级网页。胡 ]潜入饭桶历史](https://web.archive.org/web/20220524031644/https://advancedweb.hu/2018/09/11/fugitive/)

*   #### [**> > Use the logo**](https://web.archive.org/web/20220524031644/https://vorba.ch/2018/url-security-identicons.html) [ vorba.ch ] to improve URL security

*   #### [**> > IDE 支持脚本**](https://web.archive.org/web/20220524031644/http://blog.code-cop.org/2018/09/ide-support-for-scripts.html)【blog . code-COP . org】T5

*   #### [**> > Round Earth Test Strategy**](https://web.archive.org/web/20220524031644/http://www.satisfice.com/blog/archives/4947) [satisfice.com]

*   #### [**>减轻发布审核负担**](https://web.archive.org/web/20220524031644/https://waynebeaton.wordpress.com/2018/09/11/lightening-the-release-review-burden/)【Wayne beaton . WordPress . com】

*   #### [**> > The illusion of servant leaders and enterprise empowerment**](https://web.archive.org/web/20220524031644/https://daedtech.com/servant-leader/) [daedtech.com]

## 3。漫画

本周我最喜欢的迪尔伯特。

#### [**> >不能从一个萝卜里挤出血**T3【dilbert.com】和](https://web.archive.org/web/20220524031644/http://dilbert.com/strip/2018-09-10)

#### [**> >感谢你的坦诚**](https://web.archive.org/web/20220524031644/http://dilbert.com/strip/2018-09-06)【dilbert.com】

#### [**> >老大哥正在看**](https://web.archive.org/web/20220524031644/http://dilbert.com/strip/2011-05-27)【dilbert.com】

## 4。本周精选

#### [**> >实功 vs 虚功**](https://web.archive.org/web/20220524031644/https://m.signalvnoise.com/real-work-vs-imaginary-work-8bdb84a7d1da)【m.signalvnoise.com】

下一期[Java 周刊，第 247 期](/web/20220524031644/https://www.baeldung.com/java-weekly-247)上一期[Java 周刊，第 245 期](/web/20220524031644/https://www.baeldung.com/java-weekly-245)