# 带弹簧的特征标志

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-feature-flags>

## 1。概述

在本文中，我们将简要定义特性标志，并提出一种自以为是的实用方法来在 Spring Boot 应用程序中实现它们。然后，我们将利用不同的 Spring Boot 特性挖掘更复杂的迭代。

我们将讨论可能需要特性标记的各种场景，并讨论可能的解决方案。我们将使用一个比特币矿工示例应用程序来实现这一点。

## 2。特征标志

特性标志——有时也称为特性切换——是一种机制，允许我们启用或禁用应用程序的特定功能，而无需修改代码，或者最好是重新部署应用程序。

根据给定特性标志所需的动态性，我们可能需要对它们进行全局配置、按应用实例配置，或者更精细地配置——可能是按用户或请求配置。

正如软件工程中的许多情况一样，重要的是要尝试使用最直接的方法来解决手头的问题，而不增加不必要的复杂性。

特性标志是一个强有力的工具，如果使用得当，可以给我们的系统带来可靠性和稳定性。然而，当它们被误用或维护不足时，它们会很快成为复杂性和令人头痛的根源。

在许多情况下，功能标志会派上用场:

**基于主干的开发和重要特性**

在基于主干的开发中，特别是当我们想要保持频繁集成时，我们可能会发现自己还没有准备好发布某项功能。特性标志可以派上用场，使我们能够保持发布，而不使我们的更改可用，直到完成。

**特定环境配置**

我们可能会发现自己需要某些功能来为 E2E 测试环境重置数据库。

或者，我们可能需要为非生产环境使用与生产环境不同的安全配置。

因此，我们可以利用特性标志在正确的环境中切换正确的设置。

**A/B 测试**

为同一个问题发布多个解决方案并衡量其影响是一项引人注目的技术，我们可以使用特性标志来实现它。

**金丝雀放飞**

当部署新特性时，我们可能会决定逐步进行，从一小组用户开始，并在验证其行为的正确性时扩大其采用范围。特性标志允许我们实现这一点。

在接下来的部分中，我们将尝试提供一种实用的方法来处理上述场景。

让我们分解不同的策略来突出标记，从最简单的场景开始，然后进入更细粒度和更复杂的设置。

## 3。应用级特征标志

如果我们需要解决前两个用例中的任何一个，应用级特性标志是一种简单的工作方式。

一个简单的特征标志通常包括一个属性和基于该属性的值的一些配置。

### 3.1。使用弹簧轮廓的特征标志

春天的时候，我们可以利用档案。**简档使我们能够方便地有选择地配置某些 beans。有了一些围绕它们的构造，我们可以快速地为应用程序级特性标志创建一个简单而优雅的解决方案。**

让我们假设我们正在建立一个比特币挖掘系统。我们的软件已经投入生产，我们的任务是创建一个实验性的，改进的挖掘算法。

在我们的`JavaConfig` 中，我们可以分析我们的组件:

```
@Configuration
public class ProfiledMiningConfig {

    @Bean
    @Profile("!experimental-miner")
    public BitcoinMiner defaultMiner() {
        return new DefaultBitcoinMiner();
    }

    @Bean
    @Profile("experimental-miner")
    public BitcoinMiner experimentalMiner() {
        return new ExperimentalBitcoinMiner();
    }
}
```

然后，**在之前的配置中，我们只需要包含我们的配置文件来选择加入我们的新功能。**有[多种方式来配置我们的应用](https://web.archive.org/web/20220627174545/https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-external-config.html)，尤其是[启用配置文件](https://web.archive.org/web/20220627174545/https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-profiles.html)。同样，有[测试工具](https://web.archive.org/web/20220627174545/https://docs.spring.io/spring/docs/current/spring-framework-reference/testing.html#activeprofiles)让我们的生活变得更容易。

只要我们的系统足够简单，我们就可以创建一个基于环境的配置来决定应用哪些特性标志，忽略哪些特性标志。

让我们想象一下，我们有一个基于卡片而不是表格的新 UI，以及之前的实验性 miner。

我们希望在我们的验收环境(UAT)中启用这两个特性。我们可以在我们的`application.yml`文件中创建以下配置文件组:

```
spring:
  profiles:
    group:
      uat: experimental-miner,ui-cards
```

有了前面的属性，我们只需要在 UAT 环境中启用 UAT 配置文件来获得所需的特性集。当然，我们也可以在项目中添加一个`application-uat.yml`文件，为我们的环境设置添加额外的属性。

在我们的例子中，我们希望`uat`概要文件也包括`experimental-miner`和`ui-cards.`

注意:如果我们使用的是 2.4.0 之前的 Spring Boot 版本，我们将使用 UAT 概要文件中的`spring.profiles.include`属性来配置额外的概要文件。与`spring.profiles.active,`相比，前者使我们能够以添加的方式包含配置文件。

### 3.2。使用自定义属性的功能标志

简介是完成工作的一个伟大而简单的方法。但是，我们可能需要用于其他目的的概要文件。或者，我们可能想要构建一个更加结构化的特征标志基础设施。

对于这些情况，自定义属性可能是一个理想的选择。

**让我们利用`@ConditionalOnProperty` 和我们的名称空间**重写我们之前的例子:

```
@Configuration
public class CustomPropsMiningConfig {

    @Bean
    @ConditionalOnProperty(
      name = "features.miner.experimental", 
      matchIfMissing = true)
    public BitcoinMiner defaultMiner() {
        return new DefaultBitcoinMiner();
    }

    @Bean
    @ConditionalOnProperty(
      name = "features.miner.experimental")
    public BitcoinMiner experimentalMiner() {
        return new ExperimentalBitcoinMiner();
    }
}
```

**前面的例子建立在 Spring Boot 条件配置的基础上，根据属性是设置为`true`还是`false`(或者完全省略)，配置一个组件或另一个组件。**

结果与 3.1 中的非常相似，但是现在，我们有了自己的名称空间。拥有我们的名称空间允许我们创建有意义的 YAML/属性文件:

```
#[...] Some Spring config

features:
  miner:
    experimental: true
  ui:
    cards: true

#[...] Other feature flags
```

此外，这个新设置允许我们给特性标志加上前缀——在我们的例子中，使用前缀`features`和`.`

这可能看起来是一个小细节，但是随着我们的应用程序的增长和复杂性的增加，这个简单的迭代将帮助我们控制我们的特性标志。

让我们谈谈这种方法的其他好处。

### 3.3。`ConfigurationProperties`使用@

一旦我们获得了一组带前缀的属性，我们就可以创建一个用@ConfigurationProperties 修饰的 [POJO，以便在我们的代码中获得一个编程句柄。](/web/20220627174545/https://www.baeldung.com/configuration-properties-in-spring-boot)

按照我们正在进行的例子:

```
@Component
@ConfigurationProperties(prefix = "features")
public class ConfigProperties {

    private MinerProperties miner;
    private UIProperties ui;

    // standard getters and setters

    public static class MinerProperties {
        private boolean experimental;
        // standard getters and setters
    }

    public static class UIProperties {
        private boolean cards;
        // standard getters and setters
    }
}
```

通过将我们的特征标志的状态放在一个内聚的单元中，我们打开了新的可能性，允许我们容易地将信息暴露给我们系统的其他部分，比如 UI，或者下游系统。

### 3.4。暴露功能配置

我们的比特币采矿系统得到了 UI 升级，但还没有完全准备好。因此，我们决定对它进行特色标记。我们可能有一个使用 React、Angular 或 Vue 的单页应用程序。

不管使用什么技术，我们都需要知道启用了什么功能，这样我们才能相应地呈现页面。

让我们创建一个简单的端点来服务我们的配置，以便我们的 UI 可以在需要时查询后端:

```
@RestController
public class FeaturesConfigController {

    private ConfigProperties properties;

    // constructor

    @GetMapping("/feature-flags")
    public ConfigProperties getProperties() {
        return properties;
    }
}
```

可能有更复杂的方式来提供这些信息，比如[创建定制的致动器端点](/web/20220627174545/https://www.baeldung.com/spring-boot-actuators)。但是对于本指南来说，控制器端点似乎是一个足够好的解决方案。

### 3.5。保持营地清洁

虽然这听起来很明显，但是一旦我们深思熟虑地实现了我们的特性标志，同样重要的是，一旦不再需要它们，就要有纪律地把它们去掉。

**第一个用例的特性标志——基于主干的开发和重要特性——通常是短暂的**。这意味着我们需要确保我们的【Java 配置和`YAML`文件保持干净和最新。

## 4。更精细的特征标志

有时我们会发现自己身处更复杂的场景中。对于 A/B 测试或 canary 发布，我们以前的方法是远远不够的。

为了获得更细粒度的特性标志，我们可能需要创建我们的解决方案。这可能包括定制我们的用户实体以包含特定于功能的信息，或者扩展我们的 web 框架。

然而，用特征标志污染我们的用户可能不是每个人都喜欢的主意，还有其他的解决方案。

作为替代，我们可以利用一些内置工具[，比如 Togglz](https://web.archive.org/web/20220627174545/https://www.togglz.org/) 。这个工具增加了一些复杂性，但提供了一个很好的开箱即用的解决方案，并且[提供了与 Spring Boot](https://web.archive.org/web/20220627174545/https://www.togglz.org/documentation/spring-boot-starter.html) 的一流集成。

Togglz 支持不同的[激活策略](https://web.archive.org/web/20220627174545/https://www.togglz.org/documentation/activation-strategies.html):

1.  **用户名:**与特定用户相关联的标志
2.  **逐步推广:**为一定比例的用户群启用标志。这对于 Canary 版本很有用，例如，当我们想要验证我们的特性的行为时
3.  **发布日期:**我们可以安排在某个日期和时间启用标志。这可能对产品发布、协调发布或优惠和折扣有用
4.  **客户端 IP:** 根据客户端 IP 标记的功能。在将特定配置应用于特定客户时，这些可能会派上用场，因为他们拥有静态 IP
5.  **服务器 IP:** 在这种情况下，服务器的 IP 用于确定是否应该启用某个功能。这对于 canary 版本可能也是有用的，它采用了与逐步推出略有不同的方法——比如当我们想要评估实例中的性能影响时
6.  **ScriptEngine:** 我们可以基于[任意脚本](https://web.archive.org/web/20220627174545/https://docs.oracle.com/en/java/javase/11/docs/api/java.scripting/javax/script/ScriptEngine.html)启用特性标志。这可以说是最灵活的选择
7.  **系统属性:**我们可以设置某些系统属性来确定特征标志的状态。这与我们用最简单的方法获得的结果非常相似

## 5。总结

在本文中，我们有机会讨论了特性标志。此外，我们讨论了 Spring 如何帮助我们在不添加新库的情况下实现这些功能。

我们从定义这个模式如何帮助我们处理一些常见的用例开始。

接下来，我们使用 Spring 和 Spring Boot 的现成工具构建了几个简单的解决方案。据此，我们提出了一个简单而强大的特征标记构造。

在下面，我们比较了几种选择。从更简单、更不灵活的解决方案转向更复杂的模式。

最后，我们简要地提供了一些构建更健壮的解决方案的指南。当我们需要更高的粒度时，这很有用。