# 处理 Maven 无效 LOC 头错误

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/maven-invalid-loc-header-error>

## 1.介绍

有时，当我们的本地 [Maven](/web/20221206144407/https://www.baeldung.com/maven) repo 中的一个 jar 损坏时，我们会看到错误:`Invalid LOC Header`。

在本教程中，我们将学习这种情况何时发生，以及如何处理和预防。

## 2.什么时候会出现“无效的 LOC 标题”？

Maven 将一个项目的依赖项下载到我们文件系统上的一个已知位置，称为[本地存储库](/web/20221206144407/https://www.baeldung.com/maven-local-repository)。Maven 下载的每个工件都附带有 SHA1 和 MD5 校验和文件:

[![localrepo 1](img/ac71a3236e7431221f2add3e62cedfa6.png)](/web/20221206144407/https://www.baeldung.com/wp-content/uploads/2019/06/localrepo-1.png)

这些校验和的目的是确保相关工件的完整性。由于**网络和文件系统可能会失败，**就像任何其他事情一样，有时工件会被损坏，使得工件内容与签名不匹配。

在这些情况下，Maven 构建会抛出“无效的 LOC 头”错误。

解决方案是从存储库中删除损坏的 jar。让我们来看看几种方法。

## 3.删除本地存储库

对该错误的快速修复是**删除整个 Maven 本地存储库并重新构建项目:**

```java
rm -rf ${LOCAL_REPOSITORY}
```

这将清除本地缓存并重新下载所有项目依赖关系—**效率不高。**

请注意，默认的本地存储库位于`${user.home}/.m2/repository`，除非我们在`[settings.xml](https://web.archive.org/web/20221206144407/https://maven.apache.org/guides/mini/guide-configuring-maven.html) <localRepository>`标签中指定了它。我们也可以通过命令`mvn help:evaluate -Dexpression=settings.localRepository`找到本地存储库

## 4.找到腐败的罐子

另一个解决方案是**识别特定的损坏 jar 并将其从本地存储库**中删除。

当我们使用 Maven 输出 stack trace 命令时，它将包含处理失败时损坏的 jar 细节。

我们可以通过在 build 命令中添加-X 来启用调试级别的日志记录:

```java
mvn -X package
```

产生的堆栈跟踪将在日志末尾指出损坏的 jar。在识别损坏的 jar 之后，我们可以在本地存储库中找到它并删除它。现在在构建时，Maven 将重试下载 jar。

此外，我们可以使用`zip -T`命令测试归档的完整性:

```java
find ${LOCAL_REPOSITORY} -name "*.jar" | xargs -L 1 zip -T | grep error
```

## 5.验证校验和

前面提到的两种解决方案只会迫使 Maven 重新下载 jar。当然，这个问题可能会在以后的下载中再次出现。我们可以通过配置 Maven 在从远程存储库下载工件时验证校验和来防止这种情况。

我们可以在 Maven 命令中添加`–strict-checksums or -C`选项。如果计算出的校验和与校验和文件中的值不匹配，这将导致 Maven 构建失败。

有两个选项，如果校验和不匹配，要么`fail`构建:

```java
-C,--strict-checksums
```

或默认选项`warn`:

```java
-c,--lax-checksums
```

今天，Maven 在将工件上传到中央存储库时需要[签名文件](https://web.archive.org/web/20221206144407/https://central.sonatype.org/pages/requirements.html#sign-files-with-gpgpgp)。但是**在中央储存库中可能有没有签名文件**的工件，尤其是历史文件。这就是为什么默认选项是`warn`。

为了更永久的解决方案，我们可以在 Maven 的 [`settings.xml`](https://web.archive.org/web/20221206144407/https://maven.apache.org/ref/current/maven-settings/settings.html) 文件中配置`checksumPolicy`。此属性指定项目校验和验证失败时的行为。为了避免将来出现问题，让我们编辑我们的`settings.xml`文件，以便在校验和失败时使下载失败:

```java
<profiles>
    <profile>
        <repositories>
            <repository>
                <id>codehausSnapshots</id>
                <name>Codehaus Snapshots</name>
                <releases>
                    <enabled>false</enabled>
                    <updatePolicy>always</updatePolicy>
                    <checksumPolicy>fail</checksumPolicy>
                </releases>
            </repository>
        </repository>
    </profile>
</profiles>
```

当然，我们需要为每个已配置的存储库做这件事。

## 6.结论

在这篇快速的文章中，我们看到了何时会出现无效的 LOC 头错误，以及如何处理它的选项。