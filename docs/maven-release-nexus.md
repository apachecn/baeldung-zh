# Maven 发布到 Nexus

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/maven-release-nexus>

## 1。概述

在本系列的前一篇文章中，我们用 Maven 到 Nexus 设置了一个 [**部署流程。在本文中，我们将使用 Maven** 配置**发布流程——在项目的`pom` 和 Jenkins 作业中都是如此。**](/web/20220706103857/https://www.baeldung.com/maven-deploy-nexus "Maven Deploy to Nexus")

## 2。`pom`中的知识库

为了让 Maven 能够发布到 Nexus 存储库服务器，我们需要**通过`distributionManagement`元素定义存储库**信息:

```java
<distributionManagement>
   <repository>
      <id>nexus-releases</id>
      <url>http://localhost:8081/nexus/content/repositories/releases</url>
   </repository>
</distributionManagement>
```

托管发布库在 Nexus 上是现成的，所以不需要显式地创建它。

## 3。单片机中的美文`pom`

发布过程将与项目的源代码控制交互——这意味着我们首先需要在我们的`pom.xml`中定义`<scm>`元素:

```java
<scm>
   <connection>scm:git:https://github.com/user/project.git</connection>
   <url>http://github.com/user/project</url>
   <developerConnection>scm:git:https://github.com/user/project.git</developerConnection>
</scm>
```

或者，使用 git 协议:

```java
<scm>
   <connection>scm:git:[[email protected]](/web/20220706103857/https://www.baeldung.com/cdn-cgi/l/email-protection):user/project.git</connection>
   <url>scm:git:[[email protected]](/web/20220706103857/https://www.baeldung.com/cdn-cgi/l/email-protection):user/project.git</url>
   <developerConnection>scm:git:[[email protected]](/web/20220706103857/https://www.baeldung.com/cdn-cgi/l/email-protection):user/project.git</developerConnection>
</scm>
```

## 4。发布插件

发布过程使用的标准 Maven 插件是`maven-release-plugin`——这个插件的配置是最小的:

```java
<plugin>
   <groupId>org.apache.maven.plugins</groupId>
   <artifactId>maven-release-plugin</artifactId>
   <version>2.4.2</version>
   <configuration>
      <tagNameFormat>[[email protected]](/web/20220706103857/https://www.baeldung.com/cdn-cgi/l/email-protection){project.version}</tagNameFormat>
      <autoVersionSubmodules>true</autoVersionSubmodules>
      <releaseProfiles>releases</releaseProfiles>
   </configuration>
</plugin>
```

这里重要的是,`releaseProfiles`配置实际上会强制 Maven 概要文件—`releases`概要文件在发布过程中激活。

在这个过程中，`nexus-staging-maven-plugin`被用来执行到`nexus-releases` Nexus 存储库的部署:

```java
<profiles>
   <profile>
      <id>releases</id>
      <build>
         <plugins>
            <plugin>
               <groupId>org.sonatype.plugins</groupId>
               <artifactId>nexus-staging-maven-plugin</artifactId>
               <version>1.5.1</version>
               <executions>
                  <execution>
                     <id>default-deploy</id>
                     <phase>deploy</phase>
                     <goals>
                        <goal>deploy</goal>
                     </goals>
                  </execution>
               </executions>
               <configuration>
                  <serverId>nexus-releases</serverId>
                  <nexusUrl>http://localhost:8081/nexus/</nexusUrl>
                  <skipStaging>true</skipStaging>
               </configuration>
            </plugin>
         </plugins>
      </build>
   </profile>
</profiles>
```

该插件被配置为执行发布过程**而没有暂存机制**，与之前的[相同，用于部署过程(`skipStaging=true`)。](/web/20220706103857/https://www.baeldung.com/maven-deploy-nexus "Maven Deploy to Nexus")

同样类似于部署流程，**发布到 Nexus 是一个安全操作**——因此我们将再次使用 Nexus 的开箱即用`deployment`用户表单。

我们还需要在全局`settings.xml` ( `%USER_HOME%/.m2/settings.xml`)中为`nexus-releases`服务器配置凭证:

```java
<servers>
   <server>
      <id>nexus-releases</id>
      <username>deployment</username>
      <password>the_pass_for_the_deployment_user</password>
   </server>
</servers>
```

这是完整的配置

## 5。发布流程

让我们把发布过程分解成小而集中的步骤。当项目的当前版本是快照版本时，我们正在执行发布——比如说`0.1-SNAPSHOT`。

### 5.1。释放:清洁

**清洁释放**将:

*   删除发布描述符(`release.properties`)
*   删除任何备份的 POM 文件

5.2。发布:准备

发布流程的下一部分是**准备发布**；这将:

*   执行一些检查——应该没有未提交的更改，项目应该不依赖于任何快照依赖项
*   将 pom 文件中的项目版本更改为完整的发布版本号(删除快照后缀)——在我们的示例中是——`0.1`
*   运行项目**测试**套件
*   提交并推动变更
*   从这个非快照版本代码中创建**标签**
*   **在 pom 中增加项目的版本**——在我们的例子中—`0.2-SNAPSHOT`
*   提交并推动变更

5.3。释放:执行

释放过程的后一部分是**执行释放**；这将:

*   从 SCM 签出发行标签
*   构建和部署发布的代码

该流程的第二步依赖于准备步骤的输出—`release.properties`。

## 6。关于詹金斯

Jenkins 可以通过两种方式之一来执行发布过程——它可以使用自己的发布插件，也可以简单地通过运行正确发布步骤的标准 maven 作业来执行发布。

现有的专注于发布过程的 Jenkins 插件有:

*   [发布插件](https://web.archive.org/web/20220706103857/https://wiki.jenkins-ci.org/display/JENKINS/Release+Plugin "The Jenkins Release Plugin")
*   [M2 发布插件](https://web.archive.org/web/20220706103857/https://wiki.jenkins-ci.org/display/JENKINS/M2+Release+Plugin "The M2 Jenkins Release Plugin")

然而，由于执行发布的 Maven 命令足够简单，我们可以简单地定义一个标准的 Jenkins 作业来执行操作——不需要插件。

因此，对于一个新的 Jenkins 作业(构建一个 maven2/3 项目)——我们将定义 2 个字符串参数:`releaseVersion=0.1`和`developmentVersion=0.2-SNAPSHOT`。

在`**Build**`配置部分，我们可以简单地配置下面的 Maven 命令来运行:

```java
Release:Clean release:prepare release:perform 
   -DreleaseVersion=${releaseVersion} -DdevelopmentVersion=${developmentVersion}
```

运行参数化作业时，Jenkins 会提示用户指定这些参数的值，因此每次运行作业时，我们都需要为`releaseVersion`和`developmentVersion`填入正确的值。

此外，值得使用[工作空间清理插件](https://web.archive.org/web/20220706103857/https://wiki.jenkins-ci.org/display/JENKINS/Workspace+Cleanup+Plugin "Jenkins Workspace Cleanup Plugin")并检查这个构建的`Delete workspace before build starts`选项。但是请记住，发布的 **`perform`** 步骤必须由与`**prepare**`步骤相同的命令运行——这是因为后面的`**perform**`步骤将使用由`**prepare**`创建的`release.properties`文件。这意味着我们不能让一个 Jenkins 作业运行`**prepare**` 而另一个运行`**perform**`。

## 7。结论

本文展示了如何在有或没有 Jenkins 的情况下实现**发布 Maven 项目**的过程。与部署类似的[，该流程使用`nexus-staging-maven-plugin`与 Nexus 交互，并专注于 git 项目。](/web/20220706103857/https://www.baeldung.com/maven-deploy-nexus "Maven Deploy to Nexus")