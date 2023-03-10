# Maven 故障保护插件

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/maven-failsafe-plugin>

[This article is part of a series:](javascript:void(0);)[• Maven Resources Plugin](/web/20220626210859/https://www.baeldung.com/maven-resources-plugin)
[• Maven Compiler Plugin](/web/20220626210859/https://www.baeldung.com/maven-compiler-plugin)
[• Quick Guide to the Maven Install Plugin](/web/20220626210859/https://www.baeldung.com/maven-install-plugin)
• The Maven Failsafe Plugin (current article)[• Quick Guide to the Maven Surefire Plugin](/web/20220626210859/https://www.baeldung.com/maven-surefire-plugin)
[• The Maven Deploy Plugin](/web/20220626210859/https://www.baeldung.com/maven-deploy-plugin)
[• The Maven Clean Plugin](/web/20220626210859/https://www.baeldung.com/maven-clean-plugin)
[• The Maven Verifier Plugin](/web/20220626210859/https://www.baeldung.com/maven-verifier-plugin)
[• The Maven Site Plugin](/web/20220626210859/https://www.baeldung.com/maven-site-plugin)
[• Guide to the Core Maven Plugins](/web/20220626210859/https://www.baeldung.com/core-maven-plugins)

## 1。概述

这篇重点教程描述了`failsafe`插件，Maven 构建工具的核心插件之一。

关于其他核心插件的概述，请参考本文。

## 2。插件目标

**`failsafe`插件用于项目的集成测试。**它有两个目标:

*   `integration-test`–运行集成测试；默认情况下，这个目标被绑定到`integration-test`阶段
*   `verify`–验证集成测试是否通过；默认情况下，这个目标被绑定到`verify`阶段

## 3。目标执行

这个插件运行测试类中的方法，就像`surefire`插件一样。我们可以用相似的方式配置这两个插件。然而，它们之间有一些重要的区别。

首先，与包含在超级`pom.xml`中的`surefire`(参见[这篇文章](/web/20220626210859/https://www.baeldung.com/maven-surefire-plugin))不同，`failsafe`插件及其目标必须在`pom.xml`中明确指定，以成为构建生命周期的一部分:

```java
<plugin>
    <artifactId>maven-failsafe-plugin</artifactId>
    <version>2.21.0</version>
    <executions>
        <execution>
            <goals>
                <goal>integration-test</goal>
                <goal>verify</goal>
            </goals>
            <configuration>
                ...
            </configuration>
        </execution>
    </executions>
</plugin>
```

这个插件的最新版本是[这里是](https://web.archive.org/web/20220626210859/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.apache.maven.plugins%22%20AND%20a%3A%22maven-failsafe-plugin%22)。

第二，`failsafe`插件使用不同的目标运行和验证测试。**在`integration-test`阶段的测试失败不会直接导致构建失败，允许阶段`post-integration-test`执行**，在那里执行清理操作。

失败的测试，如果有的话，只在`verify`阶段报告，在集成测试环境已经被适当地拆除之后。

## 4。结论

在本文中，我们介绍了`failsafe`插件，并将其与另一个用于测试的流行插件`surefire`进行了比较。

本教程的完整源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220626210859/https://github.com/eugenp/tutorials/tree/master/maven-modules/maven-integration-test)

Next **»**[Quick Guide to the Maven Surefire Plugin](/web/20220626210859/https://www.baeldung.com/maven-surefire-plugin)**«** Previous[Quick Guide to the Maven Install Plugin](/web/20220626210859/https://www.baeldung.com/maven-install-plugin)