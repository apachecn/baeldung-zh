# 《春云》中的网飞考古

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/netflix-archaius-spring-cloud-integration>

## 1.概观

网飞 Archaius 是一个强大的配置管理库。

简而言之，它是一个框架，可以用来从许多不同的来源收集配置属性，提供对它们的快速、线程安全的访问。

最重要的是，该库允许属性在运行时动态变化，使系统无需重启应用程序就可以获得这些变化。

在这个介绍性教程中，我们将设置一个简单的 Spring Cloud Archaius 配置，我们将解释它背后发生的事情，最后，我们将看到 Spring 如何允许扩展基本设置。

## 2.网飞考古特征

正如我们所知，Spring Boot 已经提供了管理[外部化配置](https://web.archive.org/web/20220627081809/https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-external-config.html)的工具，那么为什么还要设置一个不同的机制呢？

嗯， **Archaius 提供了一些其他配置框架**没有考虑到的方便而有趣的特性。其中一些要点是:

*   动态和类型化属性
*   在属性突变时调用的回调机制
*   动态配置源的现成实现，如 URL、JDBC 和 Amazon DynamoDB
*   Spring Boot 执行器或 JConsole 可以访问的 JMX MBean，用于检查和操作属性
*   动态属性验证

这些额外津贴在很多情况下都是有益的。

因此，Spring Cloud 开发了一个库，允许轻松配置一个“Spring 环境桥”,这样 Archaius 就可以从 Spring 环境中读取属性。

## 3.属国

让我们将`spring-cloud-starter-netflix-archaius `添加到我们的应用程序中，它会将所有必要的依赖项添加到我们的项目`.`中

可选地，我们也可以将`spring-cloud-netflix`添加到我们的`dependencyManagement `部分，并依赖它的工件版本规范:

```java
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-archaius</artifactId>
    </dependency>
</dependencies>

<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-netflix</artifactId>
            <version>2.0.1.RELEASE</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

注意:我们可以检查 Maven Central 来验证我们使用的是最新版本的 [starter 库](https://web.archive.org/web/20220627081809/https://search.maven.org/classic/#search%7Cgav%7C1%7Ca%3A%22spring-cloud-starter-netflix-archaius%22)。

## 4.使用

一旦我们添加了所需的依赖项，我们将能够访问由框架管理的属性:

```java
DynamicStringProperty dynamicProperty 
  = DynamicPropertyFactory.getInstance()
  .getStringProperty("baeldung.archaius.property", "default value");

String propertyCurrentValue = dynamicProperty.get();
```

让我们看一个简短的例子，看看这是如何现成可用的。

### 4.1.快速示例

默认情况下，它动态管理应用程序的类路径中名为`config.properties`的文件中定义的所有属性。

因此，让我们用一些任意属性将它添加到我们的资源文件夹中:

```java
#config.properties
baeldung.archaius.properties.one=one FROM:config.properties
```

现在，我们需要一种方法来检查任何特定时刻的属性值。在这种情况下，我们将创建一个`RestController`,以 JSON 响应的形式检索值:

```java
@RestController
public class ConfigPropertiesController {

    private DynamicStringProperty propertyOneWithDynamic
      = DynamicPropertyFactory.getInstance()
      .getStringProperty("baeldung.archaius.properties.one", "not found!");

    @GetMapping("/property-from-dynamic-management")
    public String getPropertyValue() {
	return propertyOneWithDynamic.getName() + ": " + propertyOneWithDynamic.get();
    }
}
```

让我们试一试。我们可以向这个端点发送一个请求，服务将按预期检索存储在`config.properties `中的值。

目前为止没什么大不了的，对吧？好了，让我们继续更改类路径文件中的属性值，而不需要重新启动服务。因此，大约一分钟后，对端点的调用应该会检索到新值。很酷，不是吗？

接下来，我们将尝试了解引擎盖下发生了什么。

## 5.它是如何工作的？

首先，让我们试着理解大局。

Archaius 是 Apache 的 Commons 配置库的扩展，增加了一些不错的功能，比如动态资源的轮询框架，具有高吞吐量和线程安全的实现。

**`spring-cloud-netflix-archaius`库随后发挥作用，合并所有不同的属性源，并用这些源自动配置 Archaius 工具。**

### 5.1.网飞阿歇斯图书馆

它定义了一个复合配置，即从不同来源获得的各种配置的集合。

此外，这些配置源中的一些可能支持在运行时对更改进行轮询。Archaius 提供了接口和一些预定义的实现来配置这些类型的源。

源的集合是分层的，因此如果一个属性出现在多个配置中，最终值将是最顶端的那个。

最后，`ConfigurationManager`处理系统范围的配置和部署上下文。它可以安装最终的组合配置，或者检索已安装的配置进行修改。

### 5.2.Spring 云支持

**春云 Archaius 库的主要任务是将所有不同的配置源合并为一个`ConcurrentCompositeConfiguration`，并使用`ConfigurationManager.`** 进行安装

库定义源代码的优先顺序是:

1.  上下文中定义的任何 Apache 通用配置`AbstractConfiguration` bean
2.  在`Autowired `弹簧`ConfigurableEnvironment`中定义的所有源
3.  默认的 Archaius 源，我们在上面的例子中看到了
4.  Apache 的`SystemConfiguration`和`EnvironmentConfiguration`来源

Spring Cloud 库提供的另一个有用的特性是定义了一个 Actuator `Endpoint `来监控属性并与之交互。它的用法超出了本教程的范围。

## 6.调整和扩展 Archaius 配置

既然我们对 Archaius 的工作原理有了更好的理解，我们就可以分析如何使配置适应我们的应用程序，或者如何使用我们的配置源来扩展功能。

### 6.1.Archaius 支持的配置属性

如果我们希望 Archaius 考虑其他类似于`config.properties`的配置文件，我们可以定义`archaius.configurationSource.additionalUrls` 系统属性。

该值被解析为由逗号分隔的 URL 列表，因此，例如，我们可以在启动应用程序时添加这个系统属性:

```java
-Darchaius.configurationSource.additionalUrls=
  "classpath:other-dir/extra.properties,
  file:///home/user/other-extra.properties"
```

Archaius 将首先读取`config.properties`文件，然后按照指定的顺序读取其他文件。因此，后一个文件中定义的属性将优先于前一个文件。

我们还可以使用其他一些系统属性来配置 Archaius 默认配置的各个方面:

*   `archaius.configurationSource.defaultFileName`:类路径中的默认配置文件名
*   `archaius.fixedDelayPollingScheduler.initialDelayMills`:读取配置源前的初始延迟
*   `archaius.fixedDelayPollingScheduler.delayMills`:源的两次读取之间的延迟；默认值是 1 分钟

### 6.2.使用 Spring 添加额外的配置源

我们如何添加一个不同的配置源来被所描述的框架管理？我们如何管理优先级高于 Spring 环境中定义的动态属性呢？

回顾我们在 4.2 节中提到的，我们可以意识到 Spring 定义的复合配置中的最高配置是上下文中定义的`AbstractConfiguration`bean。

因此，**我们需要做的就是使用 Archaius 提供的一些功能将这个 Apache 的抽象类的实现添加到我们的 Spring 上下文中，Spring 的自动配置会自动将其添加到托管配置属性中**。

为了简单起见，我们将看到一个例子，在这个例子中，我们配置了一个类似于缺省值`config.properties`的属性文件，但区别在于它比 Spring 环境和应用程序属性的其余部分具有更高的优先级:

```java
@Bean
public AbstractConfiguration addApplicationPropertiesSource() {
    URL configPropertyURL = (new ClassPathResource("other-config.properties")).getURL();
    PolledConfigurationSource source = new URLConfigurationSource(configPropertyURL);
    return new DynamicConfiguration(source, new FixedDelayPollingScheduler());
}
```

幸运的是，它考虑了几个配置源，我们几乎不用费力就可以设置好。它们的配置超出了本入门教程的范围。

## 7.结论

总而言之，我们已经了解了 Archaius 和它提供的一些利用配置管理的很酷的特性。

此外，我们还看到了 Spring Cloud 自动配置库是如何让我们方便地使用这个库的 API 的。

同样，我们可以在本教程和我们的 [Github repo](https://web.archive.org/web/20220627081809/https://github.com/eugenp/tutorials/tree/master/spring-cloud-modules/spring-cloud-archaius) 中找到所有的例子。