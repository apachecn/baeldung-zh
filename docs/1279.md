# Maven 的 Spring Security

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-security-with-maven>

## 1。概述

在本文中，我们将解释如何用 Maven 设置 **Spring 安全，并回顾使用 Spring 安全依赖的具体用例。你可以在 Maven Central 上找到最新的春季安全发布[。](https://web.archive.org/web/20221122114744/https://search.maven.org/classic/#search|ga|1|g%3A%22org.springframework.security%22 "Spring Security on Maven Central")**

这是上一篇 [Spring with Maven 文章](/web/20221122114744/https://www.baeldung.com/spring-with-maven "Spring with Maven Tutorial")的后续文章，所以对于非安全性 Spring 依赖项，这是开始的地方。

## 2。Maven 的 Spring Security】

### 2.1。`spring-security-core`

核心的 Spring 安全支持——**`spring-security-core`**——包含认证和访问控制功能。所有使用 Spring Security 的项目都必须包含这个依赖项。

此外， **`spring-security-core`** 支持独立(非 web)应用程序、方法级安全和 JDBC:

```
<properties>
    <spring-security.version>5.3.4.RELEASE</spring-security.version>
    <spring.version>5.2.8.RELEASE</spring.version>
</properties>
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-core</artifactId>
    <version>${spring-security.version}</version>
</dependency>
```

注意 **Spring 和 Spring Security 在不同的发布时间表**上，所以版本号之间并不总是 1:1 匹配。

如果您使用的是 Spring 的旧版本——同样重要的是要理解这样一个事实，不言而喻， **Spring Security 4.1.x 不依赖于 Spring 4.1.x 版本！**例如，当 [Spring Security 4.1.0](https://web.archive.org/web/20221122114744/https://mvnrepository.com/artifact/org.springframework.security/spring-security-core/4.1.0.RELEASE) 发布时，Spring core framework 已经是 4.2.x 版本，因此包含了该版本作为其编译依赖。计划是在未来的版本中更紧密地结合这些依赖关系——更多细节见[这个 JIRA](https://web.archive.org/web/20221122114744/https://jira.springsource.org/browse/SEC-2123 "Upgrade dependencies of spring-security-web to match it's own version")——但是目前，这有实际的影响，我们接下来会看。
**2.2。`spring-security-web`**

为了给 Spring Security 添加 **Web 支持，我们需要`spring-security-web`依赖项:**

```
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-web</artifactId>
    <version>${spring-security.version}</version>
</dependency>
```

它包含过滤器和相关的 web 安全基础设施，支持 Servlet 环境中的 URL 访问控制。

### 2.3。Spring 安全和老 Spring 核心依赖问题

这个新的依赖关系也展示了 Maven 依赖图的一个问题。如上所述，Spring 安全 jar 并不依赖于最新的 Spring core jars(而是依赖于之前的版本)。这可能导致这些旧的**依赖项**出现在类路径的顶部，而不是新的 5.x Spring 构件。

为了理解为什么会发生这种情况，我们需要看看 **Maven 是如何解决冲突的。在版本冲突的情况下，Maven 将选择离树的根最近的 jar。例如，`spring-core`既由`spring-orm`(5 . 0 . 0`.RELEASE`版本)定义，也由`spring-security-core`(T4)定义。所以在这两种情况下，`spring-jdbc`被定义在距离我们项目的根 pom 深度为 1 的位置。因此，在我们自己的 pom 中定义`spring-orm`和`spring-security-core`的顺序实际上很重要。第一个版本优先，所以**我们可能会在我们的类路径**中使用其中一个版本。**

为了解决这个问题，我们必须**在我们自己的 pom 中明确定义一些 Spring 依赖关系**，而不依赖于隐含的 Maven 依赖关系解析机制。这样做将把特定的依赖关系放在距离我们的 pom 深度为 0 的位置(正如它在 pom 本身中定义的那样),因此它将获得优先权。以下所有内容都属于同一类别，并且都需要明确定义，或者直接定义，或者对于多模块项目，在父项目的`dependencyManagement`元素中定义:

```
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-core</artifactId>
    <version>${spring-version}</version>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>${spring-version}</version>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-jdbc</artifactId>
    <version>${spring-version}</version>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-beans</artifactId>
    <version>${spring-version}</version>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-aop</artifactId>
    <version>${spring-version}</version>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-tx</artifactId>
    <version>${spring-version}</version>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-expression</artifactId>
    <version>${spring-version}</version>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-web</artifactId>
    <version>${spring-version}</version>
</dependency>
```

### 2.4。`spring-security-config`等人

为了使用丰富的 Spring Security XML 名称空间和注释，我们需要`spring-security-config`依赖项:

```
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-config</artifactId>
    <version>${spring-security.version}</version>
</dependency>
```

最后，LDAP、ACL、CAS、OAuth 和 OpenID 支持在 Spring Security 中有各自的依赖:`spring-security-ldap`、`spring-security-acl`、`spring-security-cas, spring-security-oauth`和`spring-security-openid`。

## `3\. Using Spring Boot`

使用 Spring Boot 时， [`spring-boot-starter-security`](https://web.archive.org/web/20221122114744/https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-security) 启动器将自动包含所有依赖项，如`spring-security-core`、`spring-security-web,`和`spring-security-config`等:

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
    <version>2.3.3.RELEASE</version>
</dependency> 
```

由于 Spring Boot 将为我们自动管理所有的依赖，这也将消除之前提到的 Spring 安全和旧的核心依赖问题。

## 4。使用快照和里程碑

Spring 安全性[里程碑](https://web.archive.org/web/20221122114744/https://docs.spring.io/spring-security/site/docs/5.1.1.RELEASE/reference/html/get-spring-security.html "Spring Security Milestones")以及快照都可以在 Spring 提供的定制 Maven 存储库中获得。关于如何配置这些的更多细节，参见[如何使用快照和里程碑](/web/20221122114744/https://www.baeldung.com/spring-with-maven#milestones "How to Use Spring Milestones")。

## 5。结论

在这个快速教程中，我们讨论了在 Maven 中使用 **Spring Security 的实际细节。这里介绍的 Maven 依赖项当然是一些主要的依赖项，还有一些其他的依赖项可能值得一提，但还没有完成。然而，这应该是在支持 Maven 的项目中使用 Spring 的一个好的起点。**