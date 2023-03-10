# 使用 Spring Boot 创建自定义启动器

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-custom-starter>

## 1。概述

核心 [Spring Boot](/web/20220707143835/https://www.baeldung.com/spring-boot-start) 开发者为大多数流行的开源项目提供[启动器](/web/20220707143835/https://www.baeldung.com/spring-boot-starters)，但我们并不局限于此。

我们也可以编写自己的自定义启动器。如果我们有一个内部库供我们的组织内部使用，如果要在 Spring Boot 环境中使用，为它编写一个启动器将是一个好的实践。

这些启动器使开发人员能够避免冗长的配置并快速启动他们的开发。然而，由于后台发生了很多事情，有时很难理解一个注释或者仅仅是在`pom.xml`中包含一个依赖项是如何实现这么多特性的。

在本文中，我们将揭开 Spring Boot 魔术的神秘面纱，看看幕后发生了什么。然后我们将使用这些概念为我们自己的定制库创建一个启动器。

## 2。揭秘 Spring Boot 的自动配置

### 2.1。自动配置类别

当 Spring Boot 启动时，它在类路径中查找名为`spring.factories`的文件。这个文件位于`META-INF`目录中。让我们来看看这个来自 spring-boot-autoconfigure 项目的[文件的一个片段:](https://web.archive.org/web/20220707143835/https://github.com/spring-projects/spring-boot/blob/master/spring-boot-project/spring-boot-autoconfigure/src/main/resources/META-INF/spring.factories)

```java
# Auto Configure
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
org.springframework.boot.autoconfigure.amqp.RabbitAutoConfiguration,\
org.springframework.boot.autoconfigure.cassandra.CassandraAutoConfiguration,\
org.springframework.boot.autoconfigure.mongo.MongoAutoConfiguration,\
org.springframework.boot.autoconfigure.orm.jpa.HibernateJpaAutoConfiguration
```

该文件将一个名称映射到 Spring Boot 将尝试运行的不同配置类。因此，按照这个片段，Spring Boot 将尝试运行 RabbitMQ、Cassandra、MongoDB 和 Hibernate 的所有配置类。

这些类是否会实际运行将取决于类路径中依赖类的存在。例如，如果在类路径中找到了 MongoDB 的类，`[MongoAutoConfiguration](https://web.archive.org/web/20220707143835/https://github.com/spring-projects/spring-boot/blob/master/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/mongo/MongoAutoConfiguration.java)`将会运行，所有与 mongo 相关的 beans 都将被初始化。

这个条件初始化由`[@ConditionalOnClass](https://web.archive.org/web/20220707143835/https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/autoconfigure/condition/ConditionalOnClass.html)`注释启用。让我们看看来自`MongoAutoConfiguration`类的代码片段，看看它的用法:

```java
@Configuration
@ConditionalOnClass(MongoClient.class)
@EnableConfigurationProperties(MongoProperties.class)
@ConditionalOnMissingBean(type = "org.springframework.data.mongodb.MongoDbFactory")
public class MongoAutoConfiguration {
    // configuration code
}
```

现在，如果`[MongoClient](https://web.archive.org/web/20220707143835/https://mongodb.github.io/mongo-java-driver/)`在类路径中可用，这个配置类将运行，用默认配置设置初始化的`MongoClient`填充 Spring bean 工厂。

### 2.2。来自`application.properties`文件的自定义属性

Spring Boot 使用一些预先配置的默认值来初始化 beans。为了覆盖这些缺省值，我们通常在`application.properties`文件中用某个特定的名称来声明它们。Spring Boot 容器会自动获取这些属性。

让我们看看它是如何工作的。

在`MongoAutoConfiguration`、`@EnableConfigurationProperties`的代码片段中，注释是用充当自定义属性容器的[、`MongoProperties`、](https://web.archive.org/web/20220707143835/https://github.com/spring-projects/spring-boot/blob/master/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/mongo/MongoProperties.java)类声明的:

```java
@ConfigurationProperties(prefix = "spring.data.mongodb")
public class MongoProperties {

    private String host;

    // other fields with standard getters and setters
}
```

前缀加上字段名构成了`application.properties`文件中属性的名称。因此，要为 MongoDB 设置`host`,我们只需在属性文件中编写以下内容:

```java
spring.data.mongodb.host = localhost
```

同样，类中其他字段的值也可以使用属性文件来设置。

## 3。创建自定义启动器

基于第 2 节中的概念，要创建一个定制的启动器，我们需要编写以下组件:

1.  一个用于我们库的自动配置类，以及一个用于定制配置的属性类。
2.  starter `pom`引入库和自动配置项目的依赖关系。

为了进行演示，我们创建了一个简单的问候库,它将接收一天中不同时间的问候消息作为配置参数，并输出问候消息。我们还将创建一个示例 Spring Boot 应用程序来演示我们的自动配置和启动模块的用法。

### 3.1。自动配置模块

我们将调用我们的自动配置模块`greeter-spring-boot-autoconfigure`。该模块将有两个主要的类，即`GreeterProperties`和`GreeterAutoConfiguartion`，前者将通过`application.properties`文件启用自定义属性设置，后者将为**迎宾**库创建 beans。

让我们看看这两个类的代码:

```java
@ConfigurationProperties(prefix = "baeldung.greeter")
public class GreeterProperties {

    private String userName;
    private String morningMessage;
    private String afternoonMessage;
    private String eveningMessage;
    private String nightMessage;

    // standard getters and setters

}
```

```java
@Configuration
@ConditionalOnClass(Greeter.class)
@EnableConfigurationProperties(GreeterProperties.class)
public class GreeterAutoConfiguration {

    @Autowired
    private GreeterProperties greeterProperties;

    @Bean
    @ConditionalOnMissingBean
    public GreetingConfig greeterConfig() {

        String userName = greeterProperties.getUserName() == null
          ? System.getProperty("user.name") 
          : greeterProperties.getUserName();

        // ..

        GreetingConfig greetingConfig = new GreetingConfig();
        greetingConfig.put(USER_NAME, userName);
        // ...
        return greetingConfig;
    }

    @Bean
    @ConditionalOnMissingBean
    public Greeter greeter(GreetingConfig greetingConfig) {
        return new Greeter(greetingConfig);
    }
}
```

我们还需要在`src/main/resources/META-INF`目录中添加一个`spring.factories`文件，内容如下:

```java
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
  com.baeldung.greeter.autoconfigure.GreeterAutoConfiguration
```

在应用程序启动时，如果类路径中存在类`Greeter`，那么`GreeterAutoConfiguration`类将会运行。如果运行成功，它将通过`GreeterProperties`类读取属性，用`GreeterConfig`和`Greeter`bean 填充 Spring 应用程序上下文。

[`@ConditionalOnMissingBean`](https://web.archive.org/web/20220707143835/https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/autoconfigure/condition/ConditionalOnMissingBean.html) 注释将确保这些 beans 只有在不存在的情况下才会被创建。这使得开发人员可以通过在一个`@Configuration`类中定义自己的 beanss 来完全覆盖自动配置的 bean。

### 3.2。创造`pom.xml`

现在让我们创建 starter `pom`，它将引入自动配置模块和欢迎库的依赖关系。

按照命名约定，所有不由核心 Spring Boot 团队管理的启动器都应该以库名开头，后面加上后缀`-spring-boot-starter`。所以我们称我们的启动器为`greeter-spring-boot-starter:`

```java
<project ...>
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.baeldung</groupId>
    <artifactId>greeter-spring-boot-starter</artifactId>
    <version>0.0.1-SNAPSHOT</version>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <greeter.version>0.0.1-SNAPSHOT</greeter.version>
        <spring-boot.version>2.2.6.RELEASE</spring-boot.version>
    </properties>

    <dependencies>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
            <version>${spring-boot.version}</version>
        </dependency>

        <dependency>
            <groupId>com.baeldung</groupId>
            <artifactId>greeter-spring-boot-autoconfigure</artifactId>
            <version>${project.version}</version>
        </dependency>

        <dependency>
            <groupId>com.baeldung</groupId>
            <artifactId>greeter</artifactId>
            <version>${greeter.version}</version>
        </dependency>

    </dependencies>

</project>
```

### 3.3。使用启动器

让我们创建将使用启动器的`greeter-spring-boot-sample-app`。在`pom.xml`中，我们需要将它添加为一个依赖项:

```java
<dependency>
    <groupId>com.baeldung</groupId>
    <artifactId>greeter-spring-boot-starter</artifactId>
    <version>${greeter-starter.version}</version>
</dependency>
```

Spring Boot 将自动配置一切，我们将有一个`Greeter` bean 准备好被注入和使用。

让我们通过在带有`baeldung.greeter`前缀的`application.properties`文件中定义它们来改变`GreeterProperties`的一些默认值:

```java
baeldung.greeter.userName=Baeldung
baeldung.greeter.afternoonMessage=Woha\ Afternoon
```

最后，让我们在应用程序中使用`Greeter` bean:

```java
@SpringBootApplication
public class GreeterSampleApplication implements CommandLineRunner {

    @Autowired
    private Greeter greeter;

    public static void main(String[] args) {
        SpringApplication.run(GreeterSampleApplication.class, args);
    }

    @Override
    public void run(String... args) throws Exception {
        String message = greeter.greet();
        System.out.println(message);
    }
}
```

## 4。结论

在这个快速教程中，我们重点介绍了如何推出一个定制的 Spring Boot 启动器，以及这些启动器如何与自动配置机制一起在后台工作，以消除大量的手动配置。

我们在本文中创建的所有模块的完整源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220707143835/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-custom-starter)