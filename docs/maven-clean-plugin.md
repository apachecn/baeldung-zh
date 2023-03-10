# Maven Clean 插件

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/maven-clean-plugin>

[This article is part of a series:](javascript:void(0);)[• Maven Resources Plugin](/web/20220627172926/https://www.baeldung.com/maven-resources-plugin)
[• Maven Compiler Plugin](/web/20220627172926/https://www.baeldung.com/maven-compiler-plugin)
[• Quick Guide to the Maven Install Plugin](/web/20220627172926/https://www.baeldung.com/maven-install-plugin)
[• The Maven Failsafe Plugin](/web/20220627172926/https://www.baeldung.com/maven-failsafe-plugin)
[• Quick Guide to the Maven Surefire Plugin](/web/20220627172926/https://www.baeldung.com/maven-surefire-plugin)
[• The Maven Deploy Plugin](/web/20220627172926/https://www.baeldung.com/maven-deploy-plugin)
• The Maven Clean Plugin (current article)[• The Maven Verifier Plugin](/web/20220627172926/https://www.baeldung.com/maven-verifier-plugin)
[• The Maven Site Plugin](/web/20220627172926/https://www.baeldung.com/maven-site-plugin)
[• Guide to the Core Maven Plugins](/web/20220627172926/https://www.baeldung.com/core-maven-plugins)

## 1。概述

这个快速教程描述了`clean`插件，Maven 构建工具的核心插件之一。

关于其他核心插件的概述，请参考本文。

## 2。插件目标

`clean`生命周期只有一个名为`clean`的阶段，它被自动绑定到同名插件的唯一目标。因此，这个目标可以通过命令`mvn clean`来执行。

超级 POM 中已经包含了`clean`插件，因此我们无需在项目的 POM 中指定任何东西就可以使用它。

这个插件，顾名思义，**清理前一次构建时生成的文件和目录。**默认情况下，插件会删除`target`目录。

## 3。配置

我们可以使用`filesets`参数添加要清理的目录:

```java
<plugin>
    <artifactId>maven-clean-plugin</artifactId>
    <version>3.0.0</version>
    <configuration>
        <filesets>
            <fileset>
                <directory>output-resources</directory>
            </fileset>
        </filesets>
    </configuration>
</plugin>
```

这个插件的最新版本在这里列出[。](https://web.archive.org/web/20220627172926/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.apache.maven.plugins%22%20AND%20a%3A%22maven-clean-plugin%22)

如果`output-resources`目录包含一些生成的资源，默认设置下不能删除。我们刚刚做的更改指示`clean`插件删除除默认目录之外的那个目录。

## 4。结论

在本文中，我们讨论了`clean`插件，并说明了如何定制它。

本教程的完整源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220627172926/https://github.com/eugenp/tutorials/tree/master/maven-modules/maven-plugins)

Next **»**[The Maven Verifier Plugin](/web/20220627172926/https://www.baeldung.com/maven-verifier-plugin)**«** Previous[The Maven Deploy Plugin](/web/20220627172926/https://www.baeldung.com/maven-deploy-plugin)