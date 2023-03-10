# Spring Maven 仓库

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-maven-repository>

## 1。概述

本文将展示在一个项目中使用 Spring 工件时应该使用什么样的 Maven 仓库——参见 Spring wiki 上的[仓库的完整列表。以前的 SpringSource 工件管理基础设施是`maven.springframework.org`——现在已经被弃用，取而代之的是更强大的`repo.spring.io`。](https://web.archive.org/web/20220817074457/https://github.com/SpringSource/spring-framework/wiki/SpringSource-repository-FAQ#what-repositories-are-available "Available Repositories")

## 2。Maven 发布

所有的 GA/Release 工件都发布到 Maven Central，所以如果只需要发布，就不需要在`pom`中添加任何新的 repo。然而，有一个定制，[可浏览的](https://web.archive.org/web/20220817074457/https://repo.spring.io/release/ "RELEASE (generally available) versions") **Maven 存储库也可用于春季版本**，如果由于某种原因中央不可用:

```java
<repositories>
    <repository> 
        <id>repository.spring.release</id> 
        <name>Spring GA Repository</name> 
        <url>http://repo.spring.io/release</url> 
    </repository>
</repositories>
```

Spring 工件版本化规则在项目 wiki 上有解释[。](https://web.archive.org/web/20220817074457/https://github.com/SpringSource/spring-framework/wiki/Downloading-Spring-artifacts#wiki-artifact_versioning "Spring Artifact Versioning")

里程碑和快照不直接发布到 Maven Central，所以它们有自己特定的 repos。

## 3。Maven 里程碑和候选版本

对于里程碑和 RCs，需要将以下 repo 添加到`pom`:

```java
<repositories>
    <repository> 
        <id>repository.spring.milestone</id> 
        <name>Spring Milestone Repository</name> 
        <url>http://repo.spring.io/milestone</url> 
    </repository>
</repositories>
```

一旦定义了这个存储库，项目就可以开始使用 Spring [里程碑依赖关系](https://web.archive.org/web/20220817074457/https://repo.spring.io/milestone/ "M* (milestone) and RC (release candidate) versions"):

```java
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-core</artifactId>
    <version>4.2.0.RC3</version>
</dependency>
```

## 4。Maven 快照

与里程碑类似，Spring 快照托管在一个定制的存储库中:

```java
<repositories>
    <repository> 
        <id>repository.spring.snapshot</id> 
        <name>Spring Snapshot Repository</name> 
        <url>http://repo.spring.io/snapshot</url> 
    </repository>
</repositories>
```

一旦在 pom 中启用了存储库，项目就可以开始使用 Spring 快照了:

```java
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-core</artifactId>
    <version>4.2.5.BUILD-SNAPSHOT</version>
</dependency>
```

甚至:

```java
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-core</artifactId>
    <version>4.3.0.BUILD-SNAPSHOT</version>
</dependency>
```

快照存储库现在也可以通过[浏览](https://web.archive.org/web/20220817074457/https://repo.spring.io/snapshot/ "Spring Snapshots")。

## 5。Spring OSGI 的 Maven 库

兼容 OSGI 的 Spring 工件保存在 SpringSource[Enterprise Bundle Repository](https://web.archive.org/web/20220817074457/https://docs.spring.io/s2-dmserver/2.0.0.M5/user-guide/html/ch04s04.html "Spring OSGI Repos")中——简而言之，EBR。这些存储库包含整个 Spring 框架的有效 OSGI 包和库，以及这些库的一整套依赖关系。对于捆绑包:

```java
<repository>
    <id>com.springsource.repository.bundles.release</id> 
    <name>SpringSource Enterprise Bundle Repository - SpringSource Bundle Releases</name> 
    <url>http://repository.springsource.com/maven/bundles/release</url> 
</repository>

<repository> 
    <id>com.springsource.repository.bundles.external</id> 
    <name>SpringSource Enterprise Bundle Repository - External Bundle Releases</name> 
    <url>http://repository.springsource.com/maven/bundles/external</url> 
</repository>
```

对于 OSGI 兼容库:

```java
<repository>
    <id>com.springsource.repository.libraries.release</id>
    <name>SpringSource Enterprise Bundle Repository - SpringSource Library Releases</name>
    <url>http://repository.springsource.com/maven/libraries/release</url>
</repository>
<repository>
    <id>com.springsource.repository.libraries.external</id>
    <name>SpringSource Enterprise Bundle Repository - External Library Releases</name>
    <url>http://repository.springsource.com/maven/libraries/external</url>
</repository>
```

注意: **SpringSource EBR 现在是只读的**，并且不会在那里发布更多的 Spring Framework 3.2.x 版本。

## 6。结论

本文描述了关于在`pom`中设置 Spring 特定 Maven 仓库的实用信息——以便使用发布候选、里程碑和快照。