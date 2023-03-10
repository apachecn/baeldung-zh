# Maven 站点插件

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/maven-site-plugin>

[This article is part of a series:](javascript:void(0);)[• Maven Resources Plugin](/web/20220926200101/https://www.baeldung.com/maven-resources-plugin)
[• Maven Compiler Plugin](/web/20220926200101/https://www.baeldung.com/maven-compiler-plugin)
[• Quick Guide to the Maven Install Plugin](/web/20220926200101/https://www.baeldung.com/maven-install-plugin)
[• The Maven Failsafe Plugin](/web/20220926200101/https://www.baeldung.com/maven-failsafe-plugin)
[• Quick Guide to the Maven Surefire Plugin](/web/20220926200101/https://www.baeldung.com/maven-surefire-plugin)
[• The Maven Deploy Plugin](/web/20220926200101/https://www.baeldung.com/maven-deploy-plugin)
[• The Maven Clean Plugin](/web/20220926200101/https://www.baeldung.com/maven-clean-plugin)
[• The Maven Verifier Plugin](/web/20220926200101/https://www.baeldung.com/maven-verifier-plugin)
• The Maven Site Plugin (current article)[• Guide to the Core Maven Plugins](/web/20220926200101/https://www.baeldung.com/core-maven-plugins)

## 1。概述

本教程介绍了`site`插件，Maven 构建工具的核心插件之一。

关于其他核心插件的概述，请参考本教程。

## 2。插件目标

默认情况下，Maven `site`生命周期有两个阶段绑定到`site`插件的目标:`site`阶段绑定到`site`目标，而`site-deploy`阶段绑定到`deploy`目标。

以下是这些目标的描述:

*   `site`**–**为单个项目生成一个站点；生成的站点只显示关于 POM 中指定的工件的信息
*   `deploy`**–**将生成的站点部署到 POM 的`distributionManagement`元素中指定的 URL

除了`site`和`deploy`之外，`site`插件还有其他几个目标来定制生成文件的内容和控制部署过程。

## 3。目标执行

我们可以使用这个插件，而不用把它添加到 POM 中，因为超级 POM 已经包含了它。

要生成一个站点，只需运行`mvn site:site`或`mvn site`。

要在本地机器上查看生成的站点，运行`mvn site:run`。这个命令将把站点部署到地址为`localhost:8080`的 Jetty web 服务器上。

这个插件的目标并没有隐含地绑定到网站生命周期的某个阶段，因此我们需要直接调用它。

如果我们想停止服务器，我们可以简单地点击`Ctrl + C`。

## 4。结论

本文介绍了`site`插件以及如何实现它的目标。

我们可以在 Maven 网站上找到关于这个插件的更多信息。

Next **»**[Guide to the Core Maven Plugins](/web/20220926200101/https://www.baeldung.com/core-maven-plugins)**«** Previous[The Maven Verifier Plugin](/web/20220926200101/https://www.baeldung.com/maven-verifier-plugin)