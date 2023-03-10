# 弹簧 YAML 配置

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-yaml>

## 1。概述

配置 Spring 应用程序的方法之一是使用 YAML 配置文件。

在这个快速教程中，我们将使用 YAML 为一个简单的 Spring Boot 应用程序配置不同的配置文件。

## 延伸阅读:

## [Spring @ Value 快速指南](/web/20220829110104/https://www.baeldung.com/spring-value-annotation)

Learn to use the Spring @Value annotation to configure fields from property files, system properties, etc.[Read more](/web/20220829110104/https://www.baeldung.com/spring-value-annotation) →

## [使用带有默认值的 Spring @ Value】](/web/20220829110104/https://www.baeldung.com/spring-value-defaults)

A quick and practical guide to setting default values when using the @Value annotation in Spring.[Read more](/web/20220829110104/https://www.baeldung.com/spring-value-defaults) →

## [如何将属性值注入非 Spring 管理的类？](/web/20220829110104/https://www.baeldung.com/inject-properties-value-non-spring-class)

Learn how to initialize properties values in Java classes without the direct use of Spring's injection mechanism.[Read more](/web/20220829110104/https://www.baeldung.com/inject-properties-value-non-spring-class) →

## 2。春季 YAML 文件

Spring 概要文件帮助 Spring 应用程序为不同的环境定义不同的属性。

让我们来看一个简单的 YAML 文件，它包含两个概要文件。分隔两个配置文件的三个破折号表示一个新文档的开始，因此所有的配置文件都可以在同一个 YAML 文件中描述。

`application.yml`文件的相对路径是`/myApplication/src/main/resources/application.yml:`

```java
spring:
    config:
        activate:
            on-profile: test
name: test-YAML
environment: testing
enabled: false
servers: 
    - www.abc.test.com
    - www.xyz.test.com

---
spring:
    config:
        activate:
            on-profile: prod
name: prod-YAML
environment: production
enabled: true
servers: 
    - www.abc.com
    - www.xyz.com
```

请注意，这种设置并不意味着当我们启动应用程序时，这些配置文件中的任何一个都是活动的。除非我们明确指出，否则不会加载特定于配置文件的文档中定义的属性；默认情况下，唯一活动的配置文件将是'`default.`'

## 3。将 YAML 绑定到配置类

为了从属性文件中加载一组相关的属性，我们将创建一个 bean 类:

```java
@Configuration
@EnableConfigurationProperties
@ConfigurationProperties
public class YAMLConfig {

    private String name;
    private String environment;
    private boolean enabled;
    private List<String> servers = new ArrayList<>();

    // standard getters and setters

}
```

这里使用的注释是:

*   这将该类标记为 bean 定义的来源
*   `**@ConfigurationProperties**`–将外部配置绑定到配置类并进行验证
*   `**@EnableConfigurationProperties**`–该注释用于在 Spring 应用程序中启用`@ConfigurationProperties`注释 beans

## 4。访问 YAML 属性

为了访问 YAML 属性，我们将创建一个`YAMLConfig`类的对象，并使用该对象访问属性。

在属性文件中，我们将把环境变量`spring.profiles.active`设置为`prod`。如果我们不定义这个属性`,` ，那么只有`‘default'`配置文件将被激活。

属性文件的相对路径是`/myApplication/src/main/resources/application.properties:`

```java
spring.profiles.active=prod
```

在这个例子中，我们将使用`CommandLineRunner:`来显示属性

```java
@SpringBootApplication
public class MyApplication implements CommandLineRunner {

    @Autowired
    private YAMLConfig myConfig;

    public static void main(String[] args) {
        SpringApplication app = new SpringApplication(MyApplication.class);
        app.run();
    }

    public void run(String... args) throws Exception {
        System.out.println("using environment: " + myConfig.getEnvironment());
        System.out.println("name: " + myConfig.getName());
        System.out.println("enabled:" + myConfig.isEnabled());
        System.out.println("servers: " + myConfig.getServers());
    }
}
```

命令行上的输出:

```java
using environment: production
name: prod-YAML
enabled: true
servers: [www.abc.com, www.xyz.com]
```

## 5。YAML 房产压倒一切

在 Spring Boot，YAML 文件可以被其他 YAML 属性文件覆盖。

在 2.4.0 版之前，YAML 属性被以下位置的属性文件覆盖，按优先级最高的顺序排列:

*   放在打包 jar 外部的概要文件的属性
*   打包在打包 jar 中的概要文件的属性
*   放在打包 jar 外部的应用程序属性
*   打包在打包 jar 中的应用程序属性

从 Spring Boot 2.4 开始，外部文件总是覆盖打包的文件，不管它们是否是特定于概要文件的。

## 6。结论

在这篇简短的文章中，我们学习了如何使用 YAML 在 Spring Boot 应用程序中配置属性。我们还讨论了 Spring Boot 对 YAML 文件遵循的属性覆盖规则。

这篇文章的代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220829110104/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-properties)