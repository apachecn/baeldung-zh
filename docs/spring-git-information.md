# 将 Git 信息注入 Spring

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-git-information>

## 1。概述

在本教程中，我们将展示如何将 Git 存储库信息注入到 Maven 构建的基于 Spring Boot 的应用程序中。

为了做到这一点，我们将使用[maven-git-commit-id-plugin](https://web.archive.org/web/20221012201743/https://github.com/ktoso/maven-git-commit-id-plugin)——一个专门为此目的而创建的便利工具。

## 2。Maven 依赖关系

让我们向项目的`pom.xml`文件的`<plugins>`部分添加一个插件:

```java
<plugin>
    <groupId>pl.project13.maven</groupId>
    <artifactId>git-commit-id-plugin</artifactId>
    <version>2.2.1</version>
</plugin>
```

你可以在这里找到最新版本[。请记住，这个 **p** **卢金需要至少 3.1.1 版本的 Maven。**](https://web.archive.org/web/20221012201743/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22pl.project13.maven%22%20AND%20a%3A%22git-commit-id-plugin%22)

注意，这个插件有一个更新的版本(5.x 或更新的版本),可以在不同的库坐标上找到。但是，该版本需要 Java 11。由于我们将使用 Spring Boot 2.x 开发一个具有 Java 8 基线的示例应用程序，我们将需要使用该插件的旧版本。这使得我们能够保持 Spring Boot 和`git-commit-id-plugin`之间的兼容性。

## 3。配置

该插件有许多方便的标志和属性，扩展了它的功能。在本节中，我们将简要介绍其中的一些。如果你想了解所有这些，请访问 maven-git-commit-id-plugin 的[页面](https://web.archive.org/web/20221012201743/https://github.com/ktoso/maven-git-commit-id-plugin)，如果你想直接看例子，请到第 4 节。

以下代码片段包含插件属性的示例；根据您的需要在`<configuration></configuration>` 部分指定它们。

### 3.1。缺少存储库

如果找不到 Git 存储库，您可以将其配置为忽略错误:

```java
<failOnNoGitDirectory>false</failOnNoGitDirectory>
```

### 3.2。Git 存储库位置

如果要指定自定义的`.git`储存库位置，使用`dotGitDirectory`属性:

```java
<dotGitDirectory>${project.basedir}/submodule_directory/.git</dotGitDirectory>
```

### 3.3。输出文件

为了生成具有自定义名称和/或目录的属性文件，请使用以下部分:

```java
<generateGitPropertiesFilename>
    ${project.build.outputDirectory}/filename.properties
</generateGitPropertiesFilename>
```

### 3.4。详细程度

对于更多的日志记录用途:

```java
<verbose>true</verbose>
```

### 3.5。属性文件生成

您可以关闭`git.properties`文件的创建:

```java
<generateGitPropertiesFile>false</generateGitPropertiesFile>
```

### 3.6。“属性”前缀

如果要指定自定义属性前缀，请使用:

```java
<prefix>git</prefix>
```

### 3.7。仅用于父存储库

使用带有子模块的项目时，设置此标志可确保该插件仅适用于父存储库:

```java
<runOnlyOnce>true</runOnlyOnce>
```

### 3.8。属性排除

您可能希望排除一些敏感数据，如存储库用户信息:

```java
<excludeProperties>
    <excludeProperty>git.user.*</excludeProperty>
</excludeProperties>
```

### 3.9。属性包含

仅包括指定的数据也是可能的:

```java
<includeOnlyProperties>    
    <includeOnlyProperty>git.commit.id</includeOnlyProperty>
</includeOnlyProperties>
```

## 4。示例应用程序

让我们创建一个样本 REST 控制器，它将返回关于我们项目的基本信息。

我们将使用 Spring Boot 创建示例应用程序。如果您不知道如何设置 Spring Boot 应用程序，请参见介绍性文章:[配置 Spring Boot Web 应用程序](/web/20221012201743/https://www.baeldung.com/spring-boot-application-configuration)。

我们的应用程序将由两个类组成:`Application` 和`CommitIdController`

### 4.1。应用程序

`CommitIdApplication`将作为我们应用程序的根:

```java
@SpringBootApplication(scanBasePackages = { "com.baeldung.git" })
public class CommitIdApplication {

    public static void main(String[] args) {
        SpringApplication.run(CommitIdApplication.class, args);
    }

    @Bean
    public static PropertySourcesPlaceholderConfigurer placeholderConfigurer() {
        PropertySourcesPlaceholderConfigurer propsConfig 
          = new PropertySourcesPlaceholderConfigurer();
        propsConfig.setLocation(new ClassPathResource("git.properties"));
        propsConfig.setIgnoreResourceNotFound(true);
        propsConfig.setIgnoreUnresolvablePlaceholders(true);
        return propsConfig;
    }
}
```

除了配置我们的应用程序的根之外，我们还创建了`PropertyPlaceHolderConfigurer` bean，以便我们能够访问由插件生成的属性文件。

我们还设置了一些标志，这样即使 Spring 不能解析`git.properties`文件，应用程序也能顺利运行。

### 4.2。控制器

```java
@RestController
public class CommitInfoController {

    @Value("${git.commit.message.short}")
    private String commitMessage;

    @Value("${git.branch}")
    private String branch;

    @Value("${git.commit.id}")
    private String commitId;

    @RequestMapping("/commitId")
    public Map<String, String> getCommitId() {
        Map<String, String> result = new HashMap<>();
        result.put("Commit message",commitMessage);
        result.put("Commit branch", branch);
        result.put("Commit id", commitId);
        return result;
    }
}
```

如你所见，我们将 Git 属性注入到类字段中。

要查看所有可用的属性，请参考`git.properties` 文件或作者的 Github [页面](https://web.archive.org/web/20221012201743/https://github.com/ktoso/maven-git-commit-id-plugin)。我们还创建了一个简单的端点，对于 HTTP GET 请求，它将使用包含注入值的`a JSON`进行响应。

### 4.3。Maven 条目

我们将首先设置插件要执行的执行步骤，以及我们认为有用的任何其他配置属性:

```java
<plugin>
    <groupId>pl.project13.maven</groupId>
    <artifactId>git-commit-id-plugin</artifactId>
    <version>2.2.1</version>
    <executions>
        <execution>
            <id>get-the-git-infos</id>
            <goals>
                <goal>revision</goal>
            </goals>
        </execution>
        <execution>
            <id>validate-the-git-infos</id>
            <goals>
                <goal>validateRevision</goal>
            </goals>
        </execution>
    </executions>
    <configuration>
        <!-- ... -->
    </configuration>
</plugin>
```

为了让我们的代码正常工作，我们需要在我们的类路径中有一个`git.properties `文件。为了实现这一点，我们有两个选择。

第一种是让插件来生成文件。我们可以通过将`generateGitPropertiesFile`配置属性设置为`true`值来指定这一点:

```java
<configuration>
    <generateGitPropertiesFile>true</generateGitPropertiesFile>
</configuration>
```

第二个选项是在 resources 文件夹中包含一个`git.properties`文件。我们可以只包含我们将在项目中使用的条目:

```java
# git.properties
git.tags=${git.tags}
git.branch=${git.branch}
git.dirty=${git.dirty}
git.remote.origin.url=${git.remote.origin.url}
git.commit.id=${git.commit.id}
git.commit.id.abbrev=${git.commit.id.abbrev}
git.commit.id.describe=${git.commit.id.describe}
git.commit.id.describe-short=${git.commit.id.describe-short}
git.commit.user.name=${git.commit.user.name}
git.commit.user.email=${git.commit.user.email}
git.commit.message.full=${git.commit.message.full}
git.commit.message.short=${git.commit.message.short}
git.commit.time=${git.commit.time}
git.closest.tag.name=${git.closest.tag.name}
git.closest.tag.commit.count=${git.closest.tag.commit.count}
git.build.user.name=${git.build.user.name}
git.build.user.email=${git.build.user.email}
git.build.time=${git.build.time}
git.build.host=${git.build.host}
git.build.version=${git.build.version}
```

Maven 将用适当的值替换占位符。

注意:有些 ide 不能很好地与这个插件一起工作，当我们像上面那样定义属性时，可能会在 bootstrap 上抛出一个“循环占位符引用”错误。

在启动并请求`localhost:8080/commitId`之后，您可以看到一个 JSON 文件，其结构类似于以下内容:

```java
{
    "Commit id":"7adb64f1800f8a84c35fef9e5d15c10ab8ecffa6",
    "Commit branch":"commit_id_plugin",
    "Commit message":"Merge branch 'master' into commit_id_plugin"
}
```

## 5。与 Spring Boot 执行器集成

您可以轻松使用带有[弹簧致动器](/web/20221012201743/https://www.baeldung.com/spring-boot-actuators)的插件。

正如您在[文档](https://web.archive.org/web/20221012201743/https://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-endpoints.html)中所看到的，`GitInfoContributor`将获取`git.properties`文件(如果有的话)。因此，使用默认插件配置，调用`/info`端点时将返回 Git 信息:

```java
{
  "git": {
    "branch": "commit_id_plugin",
    "commit": {
      "id": "7adb64f",
      "time": "2016-08-17T19:30:34+0200"
    }
  }
}
```

## 6。结论

在本教程中，我们展示了使用[maven-git-commit-id-plugin](https://web.archive.org/web/20221012201743/https://github.com/ktoso/maven-git-commit-id-plugin)的基础，并创建了一个简单的 Spring Boot 应用程序，它利用了插件生成的属性。

所展示的配置并没有涵盖所有可用的标志和属性，但是它涵盖了开始使用这个插件所必需的所有基础知识。

你可以在 [Github](https://web.archive.org/web/20221012201743/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-artifacts) 上找到代码示例。