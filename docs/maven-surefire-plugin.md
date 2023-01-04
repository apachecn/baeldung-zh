# Maven Surefire 插件快速指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/maven-surefire-plugin>

[This article is part of a series:](javascript:void(0);)[• Maven Resources Plugin](/web/20220525000901/https://www.baeldung.com/maven-resources-plugin)
[• Maven Compiler Plugin](/web/20220525000901/https://www.baeldung.com/maven-compiler-plugin)
[• Quick Guide to the Maven Install Plugin](/web/20220525000901/https://www.baeldung.com/maven-install-plugin)
[• The Maven Failsafe Plugin](/web/20220525000901/https://www.baeldung.com/maven-failsafe-plugin)
• Quick Guide to the Maven Surefire Plugin (current article)[• The Maven Deploy Plugin](/web/20220525000901/https://www.baeldung.com/maven-deploy-plugin)
[• The Maven Clean Plugin](/web/20220525000901/https://www.baeldung.com/maven-clean-plugin)
[• The Maven Verifier Plugin](/web/20220525000901/https://www.baeldung.com/maven-verifier-plugin)
[• The Maven Site Plugin](/web/20220525000901/https://www.baeldung.com/maven-site-plugin)
[• Guide to the Core Maven Plugins](/web/20220525000901/https://www.baeldung.com/core-maven-plugins)

## 1。概述

本教程演示了`surefire`插件，Maven 构建工具的核心插件之一。对于其他核心插件的概述，请参考[这篇文章](/web/20220525000901/https://www.baeldung.com/core-maven-plugins)。

## 2。插件目标

**我们可以使用`surefire`插件运行项目的测试。**默认情况下，这个插件在目录`target/surefire-reports`中生成 XML 报告。

这个插件只有一个目标，`test`。这个目标被绑定到默认构建生命周期的`test`阶段，命令`mvn test `将执行它。

## 3。配置

`surefire`插件可以与测试框架 JUnit 和 TestNG 一起工作。无论我们使用哪个框架，`surefire`的行为都是一样的。

默认情况下，`surefire`自动包含所有名称以`Test`开头，或者以`Test`、`Tests`或者`TestCase`结尾的测试类。

然而，我们可以使用`excludes`和`includes`参数来改变这种配置:

```java
<plugin>
    <artifactId>maven-surefire-plugin</artifactId>
    <version>2.22.2</version>
    <configuration>
        <excludes>
            <exclude>DataTest.java</exclude>
        </excludes>
        <includes>
            <include>DataCheck.java</include>
        </includes>
    </configuration>
</plugin>
```

使用这种配置，`DataCheck`类中的测试用例被执行，而`DataTest`类中的测试用例没有被执行。

我们可以在这里找到最新版本的插件[。](https://web.archive.org/web/20220525000901/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.apache.maven.plugins%22%20AND%20a%3A%22maven-surefire-plugin%22)

## 4。结论

在这篇简短的文章中，我们浏览了`surefire`插件，描述了它唯一的目标以及如何配置它。

一如既往，本教程的完整源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220525000901/https://github.com/eugenp/tutorials/tree/master/maven-modules/maven-integration-test)

Next **»**[The Maven Deploy Plugin](/web/20220525000901/https://www.baeldung.com/maven-deploy-plugin)**«** Previous[The Maven Failsafe Plugin](/web/20220525000901/https://www.baeldung.com/maven-failsafe-plugin)