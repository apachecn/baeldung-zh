# Maven 离线模式

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/maven-offline>

## 1.概观

有时，出于各种原因，我们可能需要明确要求 Maven 不要从它的存储库中下载任何东西。

在这个简短的教程中，我们将看到如何在 Maven 中启用离线模式。

## 2.准备的

在进入离线模式之前，必须下载必要的工件。否则，我们可能无法有效地使用这种模式。

**为了准备离线模式，我们可以使用`go-offline `目标从 [`maven-dependency-plugin`](https://web.archive.org/web/20220813020625/https://maven.apache.org/plugins/maven-dependency-plugin/go-offline-mojo.html)** :

```java
mvn dependency:go-offline
```

这个目标解决了所有的项目依赖——包括插件和报告以及它们的依赖。运行这个目标后，我们就可以安全地在离线模式下工作了。

## 3.离线模式

**要在离线模式下执行 [Maven 目标和阶段](/web/20220813020625/https://www.baeldung.com/maven-goals-phases)，我们只需使用`-o `或`–offline `选项**。例如，为了在离线模式下运行集成测试:

```java
mvn -o verify
```

如果我们已经下载了所有需要的工件，这个命令将成功地执行所有的测试。否则就会失败。

还可以通过设置`~/.m2/settings.xml`文件中的`offline `属性来全局配置离线模式:

```java
<settings 
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0
                      https://maven.apache.org/xsd/settings-1.0.0.xsd">
  <offline>true</offline>
</settings>
```

这个设置将应用于所有 Maven 项目。默认情况下，`offline `属性被设置为`false`。所以，当我们使用`-o `选项时，它会在命令执行期间暂时覆盖默认设置。

## 4.结论

在这个快速教程中，我们看到了如何使用`maven-dependency-plugin`为 Maven 离线模式做准备。此外，我们熟悉了命令行方法和基于设置的方法来启用离线模式。