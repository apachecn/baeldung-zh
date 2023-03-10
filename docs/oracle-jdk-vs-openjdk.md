# Oracle JDK 和 OpenJDK 的区别

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/oracle-jdk-vs-openjdk>

## 1。简介

在本教程中，我们将探索 [Oracle Java 开发工具包](https://web.archive.org/web/20221011105944/https://www.oracle.com/technetwork/java/index.html)和 [OpenJDK](https://web.archive.org/web/20221011105944/https://openjdk.java.net/) 之间的区别。首先，我们将仔细看看它们中的每一个，然后我们将比较它们。最后，我们将列出其他 JDK 实现。

## 延伸阅读:

## [在 Ubuntu 上安装 Java](/web/20221011105944/https://www.baeldung.com/ubuntu-install-jdk)

Learn how to install OpenJDK and Oracle JDK versions 8 through 11 on Ubuntu systems.[Read more](/web/20221011105944/https://www.baeldung.com/ubuntu-install-jdk) →

## [JVM、JRE 和 JDK 的区别](/web/20221011105944/https://www.baeldung.com/jvm-vs-jre-vs-jdk)

A guide to understanding the difference between JVM, JRE, and JDK in Java.[Read more](/web/20221011105944/https://www.baeldung.com/jvm-vs-jre-vs-jdk) →

## 2.Oracle JDK 和 Java SE 历史

JDK (Java Development Kit)是一个用于 Java 平台编程的软件开发环境。它包含一个完整的 Java 运行时环境，即所谓的私有运行时。这样命名是因为它包含了比独立的 JRE 更多的工具，以及开发 Java 应用程序所需的其他组件。

**Oracle 强烈建议使用术语 JDK 来指代 Java SE(标准版)开发套件**(还有企业版和微版平台)。

让我们来看看 Java SE 的历史:

*   `JDK Beta – 1995`
*   `JDK 1.0 – January 1996`
*   `JDK 1.1 – February 1997`
*   `J2SE 1.2 – December 1998`
*   `J2SE 1.3 – May 2000`
*   `J2SE 1.4 – February 2002`
*   `J2SE 5.0 – September 2004`
*   `Java SE 6 – December 2006`
*   `Java SE 7 – July 2011`
*   **Java SE 8(LTS)–2014 年 3 月**
*   `Java SE 9 – September 2017`
*   `Java SE 10 (18.3) – March 2018`
*   **Java 参见 11(18.9 lts)–2018 年 9 月**
*   Java SE 12(19.3)-2019 年 3 月

注意:不再支持斜体版本。

我们可以看到，在 Java SE 7 之前，Java SE 的主要版本大约每两年发布一次。从 Java SE 6 升级到 Java SE 8 花了五年时间，之后又花了三年时间。

从 Java SE 10 开始，我们每六个月就会有新的发布。但是，并非所有版本都是长期支持(LTS)版本。由于甲骨文的发布计划，LTS 产品将每三年发布一次。

Java SE 11 是最新的 LTS 版本，Java SE 8 将在 2020 年 12 月之前免费获得公共更新，用于非商业用途。

这个开发套件在 2010 年甲骨文收购 Sun Microsystems 后有了现在的名字。在此之前，它的名字叫孙 JDK，是 Java 编程语言的正式实现。

## 3.OpenJDK

OpenJDK 是 Java SE 平台版的免费开源实现。它最初于 2007 年发布，是 Sun Microsystems 在 2006 年开始开发的结果。

我们应该强调的是 **OpenJDK 是自版本 SE 7** 以来 Java 标准版的官方参考实现。

最初，它只基于 JDK 7，但从 Java 10 开始，Java SE 平台的开源参考实现由 JDK 项目 T3 负责。而且，就像甲骨文一样，JDK 项目也将每六个月发布一次新功能。

我们应该注意到，在这个长期运行的项目之前，有一些 JDK 发布项目发布了一个特性，然后就停止了。

现在让我们来看看 OpenJDK 的版本:

*   `OpenJDK 6 project – based on JDK 7, but modified to provide an open-source version of Java 6`
*   `OpenJDK 7 project – 28 July 2011`
*   `OpenJDK 7u project – this project develops updates to Java Development Kit 7`
*   `OpenJDK 8 project – 18 March 2014`
*   `OpenJDK 8u project – this project develops updates to Java Development Kit 8`
*   `OpenJDK 9 project – 21 September 2017`
*   `JDK project release 10 – 20 March 2018`
*   **JDK 项目发布 2018 年 9 月 11 日至 25 日**
*   JDK 项目第 12 版—[稳定阶段](https://web.archive.org/web/20221011105944/https://openjdk.java.net/jeps/3)

## 4.甲骨文 JDK vs. OpenJDK

在本节中，我们将重点关注 Oracle JDK 和 OpenJDK 之间的主要区别。

### 4.1.发布时间表

正如我们提到的， **Oracle 将每三年发布一次版本，**而 **OpenJDK 将每六个月发布一次**。

Oracle 为其版本提供长期支持。另一方面，OpenJDK 只支持对一个版本的修改，直到下一个版本发布。

### 4.2.执照

**甲骨文 JDK 公司根据甲骨文二进制代码许可协议**获得许可，而 **OpenJDK 拥有 GNU 通用公共许可证(GNU GPL)版本 2，带有链接例外**。

使用 Oracle 平台时会涉及一些许可问题。正如甲骨文[宣布](https://web.archive.org/web/20221011105944/https://java.com/en/download/release_notice.jsp)的那样，2019 年 1 月后发布的甲骨文 Java SE 8 公共更新将无法在没有商业许可的情况下用于商业、商业或生产用途。不过 OpenJDK 是完全开源的，可以免费使用。

### 4.3.表演

两者之间没有真正的技术差异，因为 Oracle JDK 的构建过程是基于 OpenJDK 的。

谈到性能， **Oracle 在响应能力和 JVM 性能方面要好得多**。由于稳定性对企业客户的重要性，它更加关注稳定性。

相比之下，OpenJDK 发布版本的频率更高。因此，我们可能会遇到不稳定的问题。根据[社区反馈](https://web.archive.org/web/20221011105944/https://www.reddit.com/r/java/comments/6g86p9/openjdk_vs_oraclejdk_which_are_you_using/)，我们知道一些 OpenJDK 用户遇到了性能问题。

### 4.4.特征

如果我们比较功能和选项，我们会发现 **Oracle 产品有飞行记录器、Java 任务控制和应用程序类——数据共享**、T2 功能，而 **OpenJDK 有字体渲染器功能**。

另外， **Oracle 有更多的垃圾收集选项和更好的呈现器。**

### 4.5.发展和普及

**甲骨文 JDK 完全由甲骨文公司开发，**而 **OpenJDK 由甲骨文、OpenJDK 和 Java 社区开发**。然而，像 Red Hat、Azul Systems、IBM、Apple Inc .和 SAP AG 这样的顶级公司也积极参与其开发。

正如我们从上一小节的链接中看到的，当谈到在工具中使用 Java 开发工具包的顶级公司的**受欢迎程度时，**如 Android Studio 或 IntelliJ IDEA， **Oracle JDK 过去是更受欢迎的，但这两家公司都转向了基于 OpenJDK 的 JetBrains [构建](https://web.archive.org/web/20221011105944/https://bintray.com/jetbrains/intellij-jdk/)。**

此外，主要的 Linux 发行版(Fedora、Ubuntu、Red Hat Enterprise Linux)都提供 OpenJDK 作为缺省的 Java SE 实现。

## 5.自 Java 11 以来的变化

正如我们在 [Oracle 的博客文章](https://web.archive.org/web/20221011105944/https://blogs.oracle.com/java-platform-group/oracle-jdk-releases-for-java-11-and-later)中看到的，从 Java 11 开始有一些重要的变化。

首先，**当使用 Oracle JDK 作为 Oracle 产品或服务的一部分时，或者当开源软件不受欢迎时，Oracle 将把它的历史“ [BCL](https://web.archive.org/web/20221011105944/https://www.oracle.com/technetwork/java/javase/terms/license/index.html) 许可证改为开源 [GNU 通用公共许可证 v2(带有类路径例外(GPL v2+CPE)】](https://web.archive.org/web/20221011105944/https://openjdk.java.net/legal/gplv2+ce.html)和商业许可证**的组合。

每个许可证将有不同的版本，但它们在功能上是相同的，只有一些外观和包装上的不同。

此外，传统的“商业特性”，如飞行记录器、Java 任务控制、应用程序类数据共享以及 Z 垃圾收集器，现在都可以在 OpenJDK 中使用。因此，**从 Java 11 开始**，Oracle JDK 和 OpenJDK 的版本本质上是相同的。

让我们来看看主要的区别:

*   Oracle 的 Java 11 工具包在使用`-XX:+UnlockCommercialFeatures`选项时会发出警告，而在 OpenJDK 构建中，该选项会导致错误
*   Oracle JDK 提供了一种配置来为“高级管理控制台”工具提供使用日志数据
*   Oracle 一直要求第三方加密提供者通过已知证书进行签名，而 OpenJDK 中的加密框架具有开放的加密接口，这意味着对于可以使用哪些提供者没有限制
*   Oracle JDK 11 将继续包括安装程序、品牌和 JRE 打包，而 OpenJDK 版本目前以`zip`和`tar.gz`文件的形式提供
*   由于 Oracle 版本中存在一些额外的模块,`javac –release`命令对于 Java 9 和 Java 10 目标的行为有所不同
*   `java –version`和`java -fullversion`命令的输出将区分 Oracle 的构建和 OpenJDK 的构建

## 6.其他 JDK 实现

现在让我们快速地看一下其他活跃的 Java 开发工具包实现。

### 6.1.免费和开源

以下按字母顺序列出的实现是开源的，可以免费使用:

*   AdoptOpenJDK
*   亚马逊闭路电视
*   Azul 祖鲁
*   Bck2Brwsr
*   可可豆
*   代号一
*   DoppioJVM
*   Eclipse OpenJ9
*   葛拉姆 CE
*   HaikuVM
*   热点
*   杰米加
*   詹姆姆
*   JVM 板
*   吉克斯 RVM(吉克斯研究虚拟机)
*   JVM.go
*   利比里亚 jdk
*   很远很远
*   玛克辛（f.）
*   多操作系统引擎
*   RopeVM
*   uJVM

### 6.2.专有实现

也有受版权保护的实现:

*   Azul Zing JVM
*   中欧和东欧 J
*   精益喷射
*   GraalVM EE
*   Imsys AB 公司
*   牙买加(AIC)
*   混合物(Aplix)
*   微 JVM(IS2T——工业智能软件技术)
*   OJVM
*   PTC 分钟
*   SAP JVM
*   Waratek CloudVM for Java

除了上面列出的[活动实现](https://web.archive.org/web/20221011105944/https://en.wikipedia.org/wiki/List_of_Java_virtual_machines#Active)之外，我们还可以看到一个[非活动实现](https://web.archive.org/web/20221011105944/https://en.wikipedia.org/wiki/List_of_Java_virtual_machines#Inactive)的列表以及每个实现的简短描述。

## 7.结论

在本文中，我们关注了当今最流行的两个 Java 开发工具包。

首先，我们描述了它们中的每一个，然后我们强调了它们之间的差异。我们还特别关注了自 Java 11 以来的变化和差异。最后，我们列出了目前可用的其他活动实现。