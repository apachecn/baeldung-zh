# Maven 安装插件快速指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/maven-install-plugin>

[This article is part of a series:](javascript:void(0);)[• Maven Resources Plugin](/web/20221208143832/https://www.baeldung.com/maven-resources-plugin)
[• Maven Compiler Plugin](/web/20221208143832/https://www.baeldung.com/maven-compiler-plugin)
• Quick Guide to the Maven Install Plugin (current article)[• The Maven Failsafe Plugin](/web/20221208143832/https://www.baeldung.com/maven-failsafe-plugin)
[• Quick Guide to the Maven Surefire Plugin](/web/20221208143832/https://www.baeldung.com/maven-surefire-plugin)
[• The Maven Deploy Plugin](/web/20221208143832/https://www.baeldung.com/maven-deploy-plugin)
[• The Maven Clean Plugin](/web/20221208143832/https://www.baeldung.com/maven-clean-plugin)
[• The Maven Verifier Plugin](/web/20221208143832/https://www.baeldung.com/maven-verifier-plugin)
[• The Maven Site Plugin](/web/20221208143832/https://www.baeldung.com/maven-site-plugin)
[• Guide to the Core Maven Plugins](/web/20221208143832/https://www.baeldung.com/core-maven-plugins)

## 1。概述

本文描述了`install`插件，它是 Maven 构建工具的核心插件之一。

关于其他核心插件的概述，请参考本文。

## 2。插件目标

**我们使用`install`插件将工件添加到本地存储库中。**这个插件包含在超级 POM 中，因此 POM 不需要显式包含它。

这个插件最值得注意的目标是`install`，默认绑定到`install`阶段。

其他目标是`install-file`用于自动将外部工件安装到本地存储库中，以及`help`显示关于插件本身的信息。

在大多数情况下，`install`插件不需要任何定制配置。这就是为什么我们不会深入这个插件的原因。

## 3。结论

这篇文章简单介绍了一下`install`插件。

我们可以在 Maven 网站上找到关于这个插件的更多信息。

Next **»**[The Maven Failsafe Plugin](/web/20221208143832/https://www.baeldung.com/maven-failsafe-plugin)**«** Previous[Maven Compiler Plugin](/web/20221208143832/https://www.baeldung.com/maven-compiler-plugin)