# Spring Boot 配置属性迁移器

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-properties-migrator>

## 1.介绍

在本教程中，我们将探索一个由 Spring 提供的支持系统来促进 Spring Boot 升级。特别是，我们将关注 [`spring-boot-properties-migrator`](https://web.archive.org/web/20220520161837/https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-properties-migrator) 模块。它有助于迁移应用程序属性。

每次升级 Spring Boot 版本时，可能会有一些属性被标记为不推荐使用、不再受支持或新引入。Spring 为每次升级发布了全面的变更日志。然而，浏览这些变更日志可能会有点乏味。这就是`spring-boot-properties-migrator `模块来救援的地方。它通过为我们的设置提供个性化信息来做到这一点。

让我们看看这是怎么回事！

## 2.演示应用程序

让我们将我们的 Spring Boot 应用程序从版本 2.3.0 升级到版本 2.6.3。

### 2.1.性能

在我们的演示应用程序中，我们有两个属性文件。在默认属性文件`application.properties`中，我们添加:

```java
spring.resources.cache.period=31536000
spring.resources.chain.compressed=false
spring.resources.chain.html-application-cache=false
```

对于`dev`档案`YAML`档案`application-dev.yaml`:

```java
spring:
  resources:
    cache:
      period: 31536000
    chain:
      compressed: true
      html-application-cache: true
```

我们的属性文件包含几个在 Spring Boot 2.3.0 和 2.6.3 之间被替换或删除的属性。此外，为了更好的演示，我们还有`.properties`和`.yaml`文件。

### 2.2.添加依赖关系

首先，让我们在模块中添加 [`spring-boot-properties-migrator`](https://web.archive.org/web/20220520161837/https://search.maven.org/artifact/org.springframework.boot/spring-boot-properties-migrator) 作为依赖项:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-properties-migrator</artifactId>
    <scope>runtime</scope>
</dependency>
```

如果我们使用 Gradle，我们可以添加:

```java
runtime("org.springframework.boot:spring-boot-properties-migrator") 
```

依赖的范围应该是`runtime`。

## 3.运行扫描

其次，让我们打包并运行我们的应用程序。我们将使用 Maven 来构建和打包:

```java
mvn clean package
```

最后，让我们运行它:

```java
java -jar target/spring-boot-properties-migrator-demo-1.0-SNAPSHOT.jar
```

结果，`spring-boot-properties-migrator`模块扫描了我们的应用程序属性文件并发挥了它的魔力！稍后会有更多的介绍。

## 4.了解扫描输出

让我们浏览日志，了解扫描的建议。

### 4.1.可替换属性

对于具有已知替换的属性，**我们看到来自`PropertiesMigrationListener`类**的`WARN` **日志:**

```java
WARN 34777 --- [           main] o.s.b.c.p.m.PropertiesMigrationListener  : 
The use of configuration keys that have been renamed was found in the environment:

Property source 'Config resource 'class path resource [application.properties]' via location 'optional:classpath:/'':
	Key: spring.resources.cache.period
		Line: 2
		Replacement: spring.web.resources.cache.period
	Key: spring.resources.chain.compressed
		Line: 3
		Replacement: spring.web.resources.chain.compressed

Property source 'Config resource 'class path resource [application-dev.yaml]' via location 'optional:classpath:/'':
	Key: spring.resources.cache.period
		Line: 5
		Replacement: spring.web.resources.cache.period
	Key: spring.resources.chain.compressed
		Line: 7
		Replacement: spring.web.resources.chain.compressed

Each configuration key has been temporarily mapped to its replacement for your convenience. To silence this warning, please update your configuration to use the new keys.
```

**我们在日志中看到所有的关键信息，例如** **哪个属性文件、关键码、行号和替换关键码属于每个条目**。这有助于我们轻松识别和替换所有此类属性。**此外，该模块在运行时将这些属性替换为可用的替换属性**，使我们能够运行应用程序，而无需进行任何更改。

### 4.2.不支持的属性

对于没有已知替换的属性，**我们看到来自`PropertiesMigrationListener`类**的`ERROR` **日志:**

```java
ERROR 34777 --- [           main] o.s.b.c.p.m.PropertiesMigrationListener  : 
The use of configuration keys that are no longer supported was found in the environment:

Property source 'Config resource 'class path resource [application.properties]' via location 'optional:classpath:/'':
	Key: spring.resources.chain.html-application-cache
		Line: 4
		Reason: none

Property source 'Config resource 'class path resource [application-dev.yaml]' via location 'optional:classpath:/'':
	Key: spring.resources.chain.html-application-cache
		Line: 8
		Reason: none
```

**和前面的场景一样，** **我们看到违规的属性文件、键、属性文件中的行号，以及键被移除背后的原因**。但是，与前面的场景不同，应用程序的启动可能会失败，这取决于所涉及的属性。我们还可能面临运行时问题，因为这些属性不能自动迁移。

## 5.更新配置属性

现在，有了扫描提供给我们的重要信息，我们就可以更好地升级酒店了。我们知道要转到的属性文件、行号和键，可以用建议的替换项进行替换，也可以查阅发行说明，查找没有替换项的特定键。

让我们修复我们的属性文件。在默认属性文件`application.properties`中，让我们按照建议替换属性:

```java
spring.web.resources.cache.period=31536000
spring.web.resources.chain.compressed=false
```

同样，让我们更新`dev`概要文件`YAML`文件`application-dev.yaml`:

```java
spring:
  web:
    resources:
      cache:
        period: 31536000
      chain:
        compressed: false
```

总而言之，我们用`spring.web.resources.cache.period`替换了属性`spring.resources.cache.period`，用`spring.web.resources.chain.compressed`替换了属性`spring.resources.chain.compressed`。新版本不再支持`spring.resources.chain.html-application-cache`键。因此，在这种情况下，我们已经删除了它。

让我们再次运行扫描。首先，让我们构建应用程序:

```java
mvn clean package
```

那么，让我们运行它:

```java
java -jar target/spring-boot-properties-migrator-demo-1.0-SNAPSHOT.jar
```

现在，我们之前看到的来自`PropertiesMigrationListener`类的所有信息日志都消失了，这表明我们的属性迁移成功了！

## 6.所有这些是如何工作的？引擎盖下的一瞥

[Spring Boot 模块 JAR 在`META-INF`文件夹](https://web.archive.org/web/20220520161837/https://docs.spring.io/spring-boot/docs/current/reference/html/configuration-metadata.html)中包含一个`spring-configuration-metadata.json`文件。这些 JSON 文件是`spring-boot-properties-migrator`模块的信息来源。当它扫描我们的属性文件时，它从这些 JSON 文件中提取相关属性的元数据信息来构建扫描报告。

文件中的一个示例显示了我们在生成的报告中看到的各种信息:

在`spring-autoconfigure:2.6.3.jar` `META-INF/spring-configuration-metadata.json`中，我们将找到`spring.resources.cache.period`的条目:

```java
{
    "name": "spring.resources.cache.period",
    "type": "java.time.Duration",
    "deprecated": true,
    "deprecation": {
        "level": "error",
        "replacement": "spring.web.resources.cache.period"
    }
}
```

同样，在`spring-boot:2.0.0.RELEASE.jar` `META-INF/spring-configuration-metadata.json`中，我们会找到`banner.image.location`的条目:

```java
{
    "defaultValue": "banner.gif",
    "deprecated": true,
    "name": "banner.image.location",
    "description": "Banner image file location (jpg\/png can also be used).",
    "type": "org.springframework.core.io.Resource",
    "deprecation": {
        "level": "error",
        "replacement": "spring.banner.image.location"
    }
}
```

## 7.警告

在结束本文之前，让我们回顾一下关于`spring-boot-properties-migrator`的一些注意事项。

### 7.1.不要在生产中保留这种依赖性

该模块仅用于开发环境中的升级。一旦我们确定了要更新或删除的属性，然后纠正它们，我们就可以从依赖项中删除该模块。最终，我们不应该在更高的环境中包含这个模块。由于相关的某些成本，不建议使用。

### 7.2.历史属性

我们不应该在升级期间跳转版本，因为模块可能无法检测在更老的版本中被弃用的真正旧的属性。例如，让我们将`banner.image.location`添加到我们的`application.properties`文件:

```java
banner.image.location="myBanner.txt"
```

在 Spring Boot 2.0 中，此属性[已被弃用。如果我们尝试直接用 Spring Boot 版本 2.6.3 运行我们的应用程序，我们不会看到任何关于它的警告或错误消息。我们必须使用 Spring Boot 2.0 运行扫描才能检测到该属性:](https://web.archive.org/web/20220520161837/https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-2.0-Configuration-Changelog)

```java
WARN 25015 --- [           main] o.s.b.c.p.m.PropertiesMigrationListener  : 
The use of configuration keys that have been renamed was found in the environment:

Property source 'applicationConfig: [classpath:/application.properties]':
    Key: banner.image.location
	Line: 5
	Replacement: spring.banner.image.location

Each configuration key has been temporarily mapped to its replacement for your convenience. To silence this warning, please update your configuration to use the new keys.
```

## 8.结论

在本文中，我们探索了`spring-boot-properties-migrator`。这是一个方便的工具，可以扫描我们的属性文件，并给出易于操作的扫描报告。我们还查看了该模块如何实现其壮举的高级视图。最后，为了确保这个工具的最佳利用，我们检查了一些警告。

和往常一样，代码可以在 GitHub 的[上获得。](https://web.archive.org/web/20220520161837/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-properties-migrator-demo)