# Maven 部署插件

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/maven-deploy-plugin>

[This article is part of a series:](javascript:void(0);)[• Maven Resources Plugin](/web/20221126214958/https://www.baeldung.com/maven-resources-plugin)
[• Maven Compiler Plugin](/web/20221126214958/https://www.baeldung.com/maven-compiler-plugin)
[• Quick Guide to the Maven Install Plugin](/web/20221126214958/https://www.baeldung.com/maven-install-plugin)
[• The Maven Failsafe Plugin](/web/20221126214958/https://www.baeldung.com/maven-failsafe-plugin)
[• Quick Guide to the Maven Surefire Plugin](/web/20221126214958/https://www.baeldung.com/maven-surefire-plugin)
• The Maven Deploy Plugin (current article)[• The Maven Clean Plugin](/web/20221126214958/https://www.baeldung.com/maven-clean-plugin)
[• The Maven Verifier Plugin](/web/20221126214958/https://www.baeldung.com/maven-verifier-plugin)
[• The Maven Site Plugin](/web/20221126214958/https://www.baeldung.com/maven-site-plugin)
[• Guide to the Core Maven Plugins](/web/20221126214958/https://www.baeldung.com/core-maven-plugins)

## 1。概述

本教程介绍了`deploy`插件，Maven 构建工具的核心插件之一。

要快速浏览其他核心插件，请参考本文。

## 2。插件目标

我们在`deploy`阶段使用`deploy`插件，**将工件推送到远程存储库与其他开发人员共享**。

除了工件本身，这个插件还确保所有相关的信息，比如 POM、元数据或哈希值，都是正确的。

超级 POM 中指定了`deploy`插件，因此没有必要将这个插件添加到 POM 中。

这个插件最引人注目的目标是`deploy`，默认绑定到`deploy`阶段。[本教程](/web/20221126214958/https://www.baeldung.com/maven-deploy-nexus)详细介绍了`deploy`插件，因此我们不再赘述。

`deploy`插件有另一个名为`deploy-file`的目标，在远程存储库中部署工件。不过这个目标并不常用。

## 3。结论

本文简要描述了`deploy`插件。

我们可以在 Maven 网站上找到关于这个插件的更多信息。

Next **»**[The Maven Clean Plugin](/web/20221126214958/https://www.baeldung.com/maven-clean-plugin)**«** Previous[Quick Guide to the Maven Surefire Plugin](/web/20221126214958/https://www.baeldung.com/maven-surefire-plugin)