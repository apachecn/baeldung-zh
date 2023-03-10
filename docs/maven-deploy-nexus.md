# Maven 部署到 Nexus

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/maven-deploy-nexus>

## 1。概述

在上一篇文章的[中，我讨论了 Maven 项目如何在本地安装一个尚未部署在 Maven Central(或任何其他大型公共托管存储库)上的第三方 jar。](/web/20220625170805/https://www.baeldung.com/install-local-jar-with-maven/ "Install local jar with Maven")

这种解决方案应该只应用于安装、运行和维护一个完整的 Nexus 服务器可能是多余的小型项目。然而，随着项目的增长，

Nexus 很快成为托管第三方工件以及跨开发流重用内部工件的唯一真正成熟的选择。

本文将展示如何使用 Maven 将项目**的工件部署到 Nexus。**

## 2。`pom.xml`中的 Nexus 要求

为了让 Maven 能够部署它在构建的`package`阶段创建的工件，它需要通过`distributionManagement`元素 **:** 定义打包的工件将被部署到的存储库信息

```java
<distributionManagement>
   <snapshotRepository>
      <id>nexus-snapshots</id>
      <url>http://localhost:8081/nexus/content/repositories/snapshots</url>
   </snapshotRepository>
</distributionManagement>
```

Nexus 上有一个托管的、公共的`Snapshots`存储库，因此不需要进一步创建或配置任何东西。Nexus 使得确定其托管存储库的 URL 变得很容易——每个存储库显示要添加到项目 pom 的`<distributionManagement>`中的准确条目，在`Summary`选项卡下。

## 3。插件

默认情况下，Maven 通过`maven-deploy-plugin`处理部署机制——这映射到默认 Maven 生命周期的`deployment`阶段:

```java
<plugin>
   <artifactId>maven-deploy-plugin</artifactId>
   <version>2.8.1</version>
   <executions>
      <execution>
         <id>default-deploy</id>
         <phase>deploy</phase>
         <goals>
            <goal>deploy</goal>
         </goals>
      </execution>
   </executions>
</plugin>
```

对于处理部署到 Nexus 项目的工件的任务来说,`maven-deploy-plugin`是一个可行的选择，但是它不是为了充分利用 Nexus 所提供的功能而构建的。因为这个事实，Sonatype 构建了一个 Nexus 特定的插件—`nexus-staging-maven-plugin`,它实际上是为了充分利用 Nexus 必须提供的更高级的功能——比如 staging。

虽然对于一个简单的部署过程，我们不需要分段功能，但我们将继续使用这个定制的 Nexus 插件，因为它的构建目的很明确，就是为了更好地与 Nexus 对话。

使用`maven-deploy-plugin`的唯一原因是保留在未来使用 Nexus 替代品的选择——例如， [Artifactory](https://web.archive.org/web/20220625170805/https://www.jfrog.com/open-source/#os-arti "Artifactory") 仓库。然而，与其他可能在项目的整个生命周期中发生变化的组件不同，Maven Repository Manager 不太可能发生变化，因此不需要灵活性。

因此，在部署阶段使用另一个部署插件的第一步是禁用现有的默认映射:

```java
<plugin>
   <groupId>org.apache.maven.plugins</groupId>
   <artifactId>maven-deploy-plugin</artifactId>
   <version>${maven-deploy-plugin.version}</version>
   <configuration>
      <skip>true</skip>
   </configuration>
</plugin>
```

现在，我们可以定义:

```java
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
      <serverId>nexus</serverId>
      <nexusUrl>http://localhost:8081/nexus/</nexusUrl>
      <skipStaging>true</skipStaging>
   </configuration>
</plugin>
```

插件的`deploy`目标被映射到 Maven 构建的`deploy`阶段。

还要注意，如前所述，在将`-SNAPSHOT`工件简单部署到 Nexus 时，我们不需要 staging 功能，因此可以通过`<skipStaging>`元素完全禁用该功能。

默认情况下，部署目标包括登台工作流，这是发布版本的推荐工作流。

## 4。`settings.xml`全局

部署到 Nexus 是一个**安全操作**，并且在任何 Nexus 实例上都有一个`deployment`用户用于此目的。

用这个`deployment`用户的凭证配置 Maven，以便它可以正确地与 Nexus 交互，这在项目的`pom.xml`中是无法完成的。这是因为`pom`的语法不允许这样做，更不用说`pom`可能是一个公共工件，因此不太适合保存凭证信息。

服务器的凭证必须在全局 Maven `setting.xml`中定义:

```java
<servers>
   <server>
      <id>nexus-snapshots</id>
      <username>deployment</username>
      <password>the_pass_for_the_deployment_user</password>
   </server>
</servers>
```

还可以将服务器配置为使用基于密钥的安全性，而不是原始的明文凭证。

## 5。部署流程

执行部署过程非常简单:

```java
mvn clean deploy -Dmaven.test.skip=true
```

在部署作业的上下文中，跳过测试是可以的，因为这个作业应该是项目的**部署管道**中的最后一个作业。

这种部署管道的一个常见示例是一连串的 Jenkins 作业，每个作业只有在成功完成时才会触发下一个作业。因此，运行项目中的所有测试套件是管道中先前作业的责任——到部署作业运行时，所有测试都应该已经通过了。

如果运行单个命令，那么测试可以在`deployment`阶段执行之前保持活动运行:

```java
mvn clean deploy
```

## 6。结论

对于将 Maven 工件部署到 Nexus，这是一个简单而高效的解决方案。

也有点自以为是——用`nexus-staging-maven-plugin`代替默认的`maven-deploy-plugin`；转移功能被禁用等—正是这些选择使解决方案变得简单而实用。

潜在地激活完整的暂存功能可能是未来文章的主题。

最后，我们将在下一篇文章中讨论发布过程。