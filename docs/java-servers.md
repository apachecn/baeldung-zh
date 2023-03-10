# 面向 Java 的 Web 和应用服务器

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-servers>

## 1。概述

在这篇简短的文章中，我们将描述 Java 开发中不同的流行服务器。

## 2。网络与应用服务器

我们将了解 web 和应用服务器之间的区别，以及它们支持哪些 Java EE 规范。

简而言之，核心区别在于应用服务器完全支持 Java EE 规范，而 web 服务器只支持该功能的一小部分:

[![javaee spec supp](img/9db98c79507b18d211e266d67eb0b905.png)](/web/20220911005431/https://www.baeldung.com/wp-content/uploads/2017/11/javaee-spec-supp-2-1.png)

## 3。阿帕奇雄猫

Java 生态系统中最受欢迎的 web 服务器之一是 Apache Tomcat。

您可以在项目网站上查看 Apache Tomcat 的最新版本和支持的 Java 版本。

这里有一个漂亮的表格，上面有每个版本中 Tomcat 支持的具体规格。

你也可以在这里为项目[做贡献。](https://web.archive.org/web/20220911005431/https://tomcat.apache.org/getinvolved.html)

## 4。码头

Jetty web 服务器是在 T2 Eclipse 基金会下开发的。

因为它非常轻量级，所以可以很容易地嵌入到设备、框架和应用服务器中。使用 Jetty 的产品有 [Apache ActiveMQ](https://web.archive.org/web/20220911005431/http://activemq.apache.org/) 、 [Eclipse](https://web.archive.org/web/20220911005431/https://www.eclipse.org/) 、 [Google App Engine](https://web.archive.org/web/20220911005431/https://cloud.google.com/appengine/) 、 [Apache Hadoop](https://web.archive.org/web/20220911005431/https://hadoop.apache.org/) 和 [Atlassian 吉拉](https://web.archive.org/web/20220911005431/https://www.atlassian.com/software/jira)。

自然，这个项目是开源的，你可以在这里贡献自己的力量。

现在让我们从 web 服务器转移到应用服务器。

## 5。阿帕奇托米

Apache TomEE 是建立在标准 Apache Tomcat 之上的完整应用服务器，主要由 T2 tomi tribe T3 支持。你可以点击查看网站[获取最新版本。](https://web.archive.org/web/20220911005431/https://tomee.apache.org/download-ng.html)

TomEE 使我们能够使用 Tomcat 不支持的 Java EE 的一些特性。

顾名思义，这个应用服务器是在 Apache 基金会的保护伞下。

您可以在此为项目[做贡献。](https://web.archive.org/web/20220911005431/https://tomee.apache.org/community/index.html)

## 6。Oracle WebLogic

WebLogic 12 也值得一提，因为它是来自 [Oracle](https://web.archive.org/web/20220911005431/https://www.oracle.com/index.html) 的主要应用服务器产品。

最新发布和支持的 Java 版本可以在[这里](https://web.archive.org/web/20220911005431/https://www.oracle.com/middleware/technologies/fusionmiddleware-downloads.html)找到。

## 7。WebSphere

IBM 也开发了自己的应用服务器，叫做 WebSphere。最新发布和支持的 Java 版本可以在[这里](https://web.archive.org/web/20220911005431/https://www.ibm.com/cloud/websphere-application-server?lnk=STW_US_STESCH&lnk2=trial_WASCloud&pexp=def&psrc=none&mhsrc=ibmsearch_a&mhq=webshpere)找到。

WebSphere 不是一个开源项目，但是它将 WebSphere Liberty 应用程序交给了 Eclipse——这使得 WebSphere 的一些基本代码对开发人员开放，供他们使用和贡献。

你可以在这里为那个项目[做贡献。](https://web.archive.org/web/20220911005431/https://openliberty.io/contribute/)

## 8 .野猫〔t1〕

[Wildfly](https://web.archive.org/web/20220911005431/http://wildfly.org/) 是一个开源的 Java 应用服务器，由[红帽](https://web.archive.org/web/20220911005431/https://www.redhat.com/en)开发。

Wildfly 在 Java EE 应用程序中越来越受欢迎，最新发布的版本可以在这里找到。

你也可以在这里为项目[做贡献。](https://web.archive.org/web/20220911005431/https://www.wildfly.org/contribute/)

## 9。Apache Geronimo

[Apache Geronimo](https://web.archive.org/web/20220911005431/http://geronimo.apache.org/) 是由 [Apache 软件基金会](https://web.archive.org/web/20220911005431/https://www.apache.org/)在 [Apache 许可](https://web.archive.org/web/20220911005431/https://www.apache.org/licenses/LICENSE-2.0)下开发的，这使得它成为一个开源项目，因此我们也可以做出贡献，就像以前的应用服务器一样。

可用的最新版本可在[这里](https://web.archive.org/web/20220911005431/https://github.com/apache/geronimo-specs)找到。

您可以在此为项目[做贡献。](https://web.archive.org/web/20220911005431/http://geronimo.apache.org/get-involved.html)

## 10。GlassFish

Glassfish 是一款开源应用服务器，也是由甲骨文赞助的。可用的最新版本可以在[这里](https://web.archive.org/web/20220911005431/https://javaee.github.io/glassfish/download)找到。

您可以在此为项目[做贡献。](https://web.archive.org/web/20220911005431/https://javaee.github.io/glassfish/CONTRIBUTING)

## 11。结论

在这篇简短的列表式文章中，我们对 Java 生态系统中的 web 和应用服务器前景进行了高度概括。