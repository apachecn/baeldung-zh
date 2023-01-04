# Java 周刊，第 272 期

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-weekly-272>

**开始了……**

## 1。Spring 和 Java

#### [>>JVM 的内存足迹](https://web.archive.org/web/20220926194044/https://spring.io/blog/2019/03/11/memory-footprint-of-the-jvm)spring . io

一篇关于堆内存与非堆内存、本机内存区域、调整 JVM 的挑战以及 Spring 如何最小化自己的内存占用的文章。

#### [**>>Spring DevTools with Jib and IntelliJ IDEA**](https://web.archive.org/web/20220926194044/https://blog.frankel.ch/spring-devtools-jib-intellij-idea/)[blog . frankel . ch]

快速看一下如何**利用 Spring DevTools，而不将其包含在使用 Maven Jib 插件构建的生产 docker 映像**中。非常酷。

#### [**> >有条件的豆跟 Spring Boot**](https://web.archive.org/web/20220926194044/https://reflectoring.io/spring-boot-conditionals/)reflector ing . io

概述了可用于**的注释，这些注释指定了 bean 应该加载到应用程序上下文**中的条件，以及每个注释的示例用例。

#### [**> >为什么要用冬眠**](https://web.archive.org/web/20220926194044/https://vladmihalcea.com/hibernate-extra-lazy-collections/)【vladmihalcea.com】来避免额外的懒惰集合

深入观察有序列表的额外惰性映射，会发现 **N+1 查询问题，这可能会很快导致性能问题**。

#### 同样值得一读:

*   #### [> > Hayden V1 brings API stability and MicroProfile 1.2 support](https://web.archive.org/web/20220926194044/https://www.infoq.com/news/2019/03/helidon-v1-api-stability?utm_campaign=infoq_content&utm_source=infoq&utm_medium=feed&utm_term=global) [infoq.com]

*   #### [**> > Why is Kwakus**](https://web.archive.org/web/20220926194044/http://in.relation.to/2019/03/08/why-quarkus/)

**网络研讨会和演示:**

*   #### [**> > A wonderful podcast: Matt Laibl and james ward at Dev Nexus 2019**](https://web.archive.org/web/20220926194044/https://spring.io/blog/2019/03/08/a-bootiful-podcast-matt-raible-and-james-ward-at-devnexus-2019) [ Spring.io ]

*   #### [**>利用可视化工作室代码**](https://web.archive.org/web/20220926194044/https://www.infoq.com/presentations/spring-boot-studio-visual-code?utm_campaign=infoq_content&utm_source=infoq&utm_medium=feed&utm_term=global)【infoq . com】黑客攻击 Spring Boot 应用

*   #### [**> > Cotrim: Write it once and run around (actually)**](https://web.archive.org/web/20220926194044/https://www.infoq.com/presentations/kotlin-run-anywhere?utm_campaign=infoq_content&utm_source=infoq&utm_medium=feed&utm_term=global) [infoq.com]

*   #### [**> > The global event stream is made of the spring cloud stream & Cloud Pub/sub**](https://web.archive.org/web/20220926194044/https://www.infoq.com/presentations/google-cloud-pub-sub-spring?utm_campaign=infoq_content&utm_source=infoq&utm_medium=feed&utm_term=global) [infoq.com]

*   #### [**> > Using Istio and Kubernetes**](https://web.archive.org/web/20220926194044/https://www.infoq.com/presentations/istio-microservices?utm_campaign=infoq_content&utm_source=infoq&utm_medium=feed&utm_term=global) [infoq.com] to reduce the complexity of microservice architecture

*   #### [**> > Ethics and artificial intelligence: identifying and preventing the deviation in the prediction model**](https://web.archive.org/web/20220926194044/https://www.infoq.com/presentations/ai-bias-discrimination-model?utm_campaign=infoq_content&utm_source=infoq&utm_medium=feed&utm_term=global) [infoq.com]

*   #### [**> > Successful iteration: a case study of remote pair programming, the evolution of a dream and the international turning point**](https://web.archive.org/web/20220926194044/https://www.infoq.com/presentations/corelogic-iteration-success?utm_campaign=infoq_content&utm_source=infoq&utm_medium=feed&utm_term=global) [[infoq.com] t4

*   #### [**> > What can Jason bocks and Paul Johnston technical experts do for climate change**](https://web.archive.org/web/20220926194044/https://www.infoq.com/podcasts/technologists-initiative-on-climate?utm_campaign=infoq_content&utm_source=infoq&utm_medium=feed&utm_term=global) [infoq.com]

#### 升级时间:

*   #### [**>春云数据流及 Skipper 2.0 GA 发布**](https://web.archive.org/web/20220926194044/https://spring.io/blog/2019/03/06/spring-cloud-data-flow-and-skipper-2-0-ga-released) 春天。io

*   #### [**> > Chunyun Greenwich. SR1 is now listed**](https://web.archive.org/web/20220926194044/https://spring.io/blog/2019/03/07/spring-cloud-greenwich-sr1-is-now-available) [ spring.io ]

*   #### [**>>Spring Boot 2.2 M1**](https://web.archive.org/web/20220926194044/https://spring.io/blog/2019/03/08/spring-boot-2-2-m1)春天。io

*   #### [**> >牧场主实验室发布面向边缘、物联网和电讯公司平台的轻量级库伯内特斯发行版【k3s】**](https://web.archive.org/web/20220926194044/https://www.infoq.com/news/2019/03/rancher-labs-k3s-kubernetes?utm_campaign=infoq_content&utm_source=infoq&utm_medium=feed&utm_term=global)【infoq . com】

## 2。技术和思考

#### [**>>**](https://web.archive.org/web/20220926194044/https://blog.codecentric.de/en/2019/03/walkthrough-dvc/)[blog . codecentric . de]

很好的介绍了 DVC，一个用于机器学习项目的**开源版本控制系统。**

#### [**> >伟大的工程师需要文科**](https://web.archive.org/web/20220926194044/https://www.infoq.com/articles/great-engineer-needs-liberal-arts?utm_campaign=infoq_content&utm_source=infoq&utm_medium=feed&utm_term=global)【infoq.com】

还有一篇关于文科教育的好处以及它的课程如何帮助我们创造伟大的软件的好文章。

#### 同样值得一读:

*   #### [**> > medium.com**](https://web.archive.org/web/20220926194044/https://medium.com/netflix-techblog/design-principles-for-mathematical-engineering-in-experimentation-platform-15b3ea143b1f) Mathematical Engineering Design Principles of Netflix Experimental Platform

*   #### [**> > Convolutional neural network for damage detection**](https://web.archive.org/web/20220926194044/https://blog.codecentric.de/en/2019/03/convolutional-neural-networks-damage-detection/) [ blog.codecentric.de

*   #### [**> > Open source is beneficial to innovation and organizational agility**](https://web.archive.org/web/20220926194044/https://www.infoq.com/news/2019/03/open-source-benefits?utm_campaign=infoq_content&utm_source=infoq&utm_medium=feed&utm_term=global) [infoq.com]

## 3。漫画

#### 本周我最喜欢的迪尔伯特。

#### [**> >营销谎言**](https://web.archive.org/web/20220926194044/https://dilbert.com/strip/2019-03-11)【dilbert.com

#### [**> >员工敬业度上升**](https://web.archive.org/web/20220926194044/https://dilbert.com/strip/2019-03-06)【dilbert.com】

#### [**> >呆呆的不知所措**](https://web.archive.org/web/20220926194044/https://dilbert.com/strip/2019-03-09)【dilbert.com】

## 4。本周精选

#### [> >别耍小聪明用登录形式](https://web.archive.org/web/20220926194044/http://bradfrost.com/blog/post/dont-get-clever-with-login-forms/)【bradfrost.com】

下一期[Java 周刊，第 273 期](/web/20220926194044/https://www.baeldung.com/java-weekly-273)上一期[Java 周刊，第 271 期](/web/20220926194044/https://www.baeldung.com/java-weekly-271)