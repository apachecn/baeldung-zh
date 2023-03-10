# Maven 验证器插件

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/maven-verifier-plugin>

[This article is part of a series:](javascript:void(0);)[• Maven Resources Plugin](/web/20221126233205/https://www.baeldung.com/maven-resources-plugin)
[• Maven Compiler Plugin](/web/20221126233205/https://www.baeldung.com/maven-compiler-plugin)
[• Quick Guide to the Maven Install Plugin](/web/20221126233205/https://www.baeldung.com/maven-install-plugin)
[• The Maven Failsafe Plugin](/web/20221126233205/https://www.baeldung.com/maven-failsafe-plugin)
[• Quick Guide to the Maven Surefire Plugin](/web/20221126233205/https://www.baeldung.com/maven-surefire-plugin)
[• The Maven Deploy Plugin](/web/20221126233205/https://www.baeldung.com/maven-deploy-plugin)
[• The Maven Clean Plugin](/web/20221126233205/https://www.baeldung.com/maven-clean-plugin)
• The Maven Verifier Plugin (current article)[• The Maven Site Plugin](/web/20221126233205/https://www.baeldung.com/maven-site-plugin)
[• Guide to the Core Maven Plugins](/web/20221126233205/https://www.baeldung.com/core-maven-plugins)

## 1。概述

本教程介绍了`verifier`插件，Maven 构建工具的核心插件之一。

关于其他核心插件的概述，请参考[这篇概述文章](/web/20221126233205/https://www.baeldung.com/core-maven-plugins)。

## 2。插件目标

`verifier`插件只有一个目标——`verify`。**该目标验证文件和目录是否存在**，根据正则表达式检查文件内容。

尽管名字如此，`verify`目标默认绑定到`integration-test`阶段，而不是`verify`阶段。

## 3。配置

只有当`verifier`插件被显式添加到`pom.xml`时，它才会被触发:

```java
<plugin>
    <artifactId>maven-verifier-plugin</artifactId>
    <version>1.1</version>
    <configuration>
        <verificationFile>input-resources/verifications.xml</verificationFile>
    </configuration>
    <executions>
        <execution>
            <goals>
                <goal>verify</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

[这个链接](https://web.archive.org/web/20221126233205/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.apache.maven.plugins%22%20AND%20a%3A%22maven-verifier-plugin%22)显示了插件的最新版本。

**验证文件的默认位置是`src/test/verifier/verifications.xml`** 。如果我们想使用另一个文件，我们必须为参数`verificationFile`设置一个值。

以下是给定配置中显示的验证文件的内容:

```java
<verifications 

  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/verifications/1.0.0 
  http://maven.apache.org/xsd/verifications-1.0.0.xsd">
    <files>
        <file>
            <location>input-resources/baeldung.txt</location>
            <contains>Welcome</contains>
        </file>
    </files>
</verifications>
```

该验证文件确认名为`input-resources/baeldung.txt`的文件存在，并且它包含单词`Welcome`。我们之前已经添加了这样一个文件，因此目标执行成功。

## 4。结论

在本文中，我们浏览了一下`verifier`插件，并描述了如何定制它。

本教程的完整源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221126233205/https://github.com/eugenp/tutorials/tree/master/maven-modules/maven-plugins)

Next **»**[The Maven Site Plugin](/web/20221126233205/https://www.baeldung.com/maven-site-plugin)**«** Previous[The Maven Clean Plugin](/web/20221126233205/https://www.baeldung.com/maven-clean-plugin)