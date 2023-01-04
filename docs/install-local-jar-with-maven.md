# 用 Maven 安装本地 jar

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/install-local-jar-with-maven>

## 1。问题和选项

Maven 是一个非常通用的工具，其可用的公共库是首屈一指的。然而，总是会有工件**不在任何地方**托管，或者托管它的存储库依赖起来有风险，因为当你需要它时它可能不在。

当这种情况发生时，有几个选择:

*   咬紧牙关，安装一个成熟的**存储库管理**解决方案[，比如 Nexus](https://web.archive.org/web/20221128113625/http://www.sonatype.org/nexus/ "Nexus")
*   尝试将工件上传到一个更好的公共存储库中
*   使用 maven 插件在本地安装工件

 **Nexus 当然是更成熟的解决方案，但它也是**更复杂的**。提供一个实例来运行 Nexus，设置 Nexus 本身，配置和维护它，对于使用单个 jar 这样简单的问题来说可能是多余的。然而，如果这种场景——托管定制工件——很常见，那么存储库管理器就很有意义。

将工件上传到**一个公共库**或者直接上传到 Maven central 也是一个好的解决方案，但是通常[需要很长的时间](https://web.archive.org/web/20221128113625/https://maven.apache.org/guides/mini/guide-central-repository-upload.html "Uploading an artifact to Maven Central")。此外，这个库可能根本不支持 Maven，这使得这个过程更加困难，所以现在能够使用这个工件并不是一个现实的解决方案。

剩下第三个选项——在源代码控制中添加工件并使用 maven 插件——在这种情况下，在构建过程需要它之前,[maven-install-plugin](https://web.archive.org/web/20221128113625/https://maven.apache.org/plugins/maven-install-plugin/ "maven-install-plugin")到**在本地安装它。这是迄今为止最容易和最可靠的选择。**

## 2。用 m `aven-install-plugin` 安装本地 Jar

让我们从将工件安装到我们的本地存储库中所需的完整配置开始:

```java
<plugin>
   <groupId>org.apache.maven.plugins</groupId>
   <artifactId>maven-install-plugin</artifactId>
   <version>2.5.1</version>
   <configuration>
      <groupId>org.somegroup</groupId>
      <artifactId>someartifact</artifactId>
      <version>1.0</version>
      <packaging>jar</packaging>
      <file>${basedir}/dependencies/someartifact-1.0.jar</file>
      <generatePom>true</generatePom>
   </configuration>
   <executions>
      <execution>
         <id>install-jar-lib</id>
         <goals>
            <goal>install-file</goal>
         </goals>
         <phase>validate</phase>
      </execution>
   </executions>
</plugin>
```

现在，我们来分解分析一下这个配置的细节。

### 2.1。神器信息

工件信息被定义为`<configuration>`元素的一部分。实际的语法非常类似于声明依赖关系——一个`groupId`、`artifactId`和`version`元素。

配置的下一部分需要定义工件的`packaging`——这被指定为`jar`。

接下来，我们需要使用 Maven 中可用的 **[属性](https://web.archive.org/web/20221128113625/https://cwiki.apache.org/confluence/display/MAVEN/Maven+Properties+Guide "Maven Properties Guide")提供要安装的实际 jar 文件的**位置**——这可以是绝对文件路径，也可以是相对路径。在这种情况下，`${basedir}`属性表示项目的根，即`pom.xml`文件所在的位置。这意味着`someartifact-1.0.jar`文件需要放在根目录下的`/dependencies/`目录中。**

最后，还有其他几个[可选细节](https://web.archive.org/web/20221128113625/https://maven.apache.org/plugins/maven-install-plugin/install-file-mojo.html "install-file goal")也可以配置。

### 2.2。执行

从标准 Maven [构建生命周期](https://web.archive.org/web/20221128113625/https://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.html "Introduction to the Build Lifecycle")开始， **`install-file`目标**的执行被绑定到 **`validate`阶段**。因此，在尝试编译之前，您需要显式运行验证阶段:

```java
mvn validate
```

在这一步之后，标准编译将开始工作:

```java
mvn clean install
```

一旦编译阶段执行完毕，我们的`someartifact-1.0.jar`就被正确地安装在我们的本地存储库中，就像任何其他可能从 Maven central 本身检索到的工件一样。

### 2.3。生成`POM` vs 提供`POM`

我们是否需要为工件提供一个`pom.xml`文件的问题主要取决于工件本身的**运行时依赖**。简单地说，如果工件在运行时依赖于其他 jar，那么这些 jar 也需要在运行时出现在类路径上**。用一个简单的工件应该不成问题，因为它在运行时可能没有依赖关系(依赖关系图中的一片叶子)。**

`install-file`目标中的`generatePom` 选项应该满足这些类型的工件:

```java
<generatePom>true</generatePom>
```

然而，如果工件更复杂，并且确实有重要的**依赖项**，那么，如果这些依赖项还不在类路径中，就必须添加它们。一种方法是在项目的 pom 文件中手工定义这些新的依赖关系。更好的解决方案是提供一个定制的`pom.xml`文件和已安装的工件:

```java
<generatePom>false</generatePom>
<pomFile>${basedir}/dependencies/someartifact-1.0.pom</pomFile>
```

这将允许 Maven 解析这个定制`pom.xml`中定义的工件的所有依赖项，而不必在项目的主 pom 文件中手工定义它们。

## 3。结论

本文介绍了如何通过使用`maven-install-plugin`在本地安装 jar 来使用不在 Maven 项目中任何地方托管的 jar。**