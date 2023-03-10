# 使用 Spring 的项目配置

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/project-configuration-with-spring>

**目录**

*   [**1。**配置必须针对特定环境](#overview)
*   [**2。**每个环境的`.properties`文件](#properties)
*   [**3。**弹簧配置](#config)
*   [**4。**在每个环境中设置属性](#env)
*   [**5。**测试和美文](#maven)
*   [**6。**更进一步](#further)
*   [**7。**结论](#conclusion)

## 1。配置必须针对特定环境

配置必须特定于环境，这是不争的事实。如果不是这样，那就不是配置，我们只是在代码中硬编码值。

对于一个 Spring 应用程序，你可以使用几种解决方案——从简单的解决方案到超级灵活、高度复杂的替代方案。

一个更常见和简单的解决方案是灵活使用属性文件和由 Spring 提供的一级属性支持。

出于本文的目的，作为概念验证，我们将研究一种特定类型的属性——数据库配置。将一种数据库配置用于生产，将另一种用于测试，将另一种用于开发环境，这是非常合理的。

## 2。每个环境的`.properties`文件

让我们开始概念验证，首先定义我们想要定位的环境:

Seven hundred and twenty

接下来，让我们创建 3 个属性文件，每个环境一个:

*   `persistence-dev.properties`
*   `persistence-staging.properties`
*   `persistence-production.properties`

在一个典型的 Maven 应用程序中，它们可以驻留在`src/main/resources`中，但是无论它们在哪里，当应用程序被部署时，它们都需要在类路径中可用。

一个重要的旁注—**让所有的属性文件都处于版本控制之下使得配置更加透明和可复制。这与将配置放在磁盘的某个地方并简单地将 Spring 指向它们是相反的。**

## 3。弹簧配置

在 Spring 中，我们将根据环境包含正确的文件:

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans 
   xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
   xmlns:context="http://www.springframework.org/schema/context"
   xsi:schemaLocation="http://www.springframework.org/schema/beans
      http://www.springframework.org/schema/beans/spring-beans-4.0.xsd
      http://www.springframework.org/schema/context
      http://www.springframework.org/schema/context/spring-context-4.0.xsd">

      <context:property-placeholder
         location="
         classpath*:*persistence-${envTarget}.properties" />

</beans>
```

当然，Java 配置也可以做到这一点:

```java
@PropertySource({ "classpath:persistence-${envTarget:dev}.properties" })
```

这种方法允许为特定的、有针对性的目的灵活地拥有多个`*.properties`文件。例如——在我们的例子中，持久性 Spring 配置导入了持久性属性——这非常有意义。安全配置将导入与安全相关的属性等等。

## 4。在每个环境中设置属性

最终的、可部署的 war **将包含所有属性文件**——为了持久化，三个变量`persistence-*.properties`。由于这些文件实际上被不同地命名，所以不必担心意外地包含错误的文件。我们将把**设置为`envTarget`变量**，从而从多个现有变量中选择我们想要的实例。

`envTarget`变量可以在操作系统/环境中设置，或者作为 JVM 命令行的参数:

```java
-DenvTarget=dev
```

## 5。测试和 Maven

对于需要启用持久性的集成测试，我们只需在 pom.xml 中设置`envTarget`属性:

```java
<plugin>
   <groupId>org.apache.maven.plugins</groupId>
   <artifactId>maven-surefire-plugin</artifactId>
   <configuration>
      <systemPropertyVariables>
         <envTarget>h2_test</envTarget>
      </systemPropertyVariables>
   </configuration>
</plugin>
```

相应的`persistence-h2_test.properties`文件可以放在`src/test/resources`中，这样**将只用于测试**，而不是在运行时不必要地包含和部署到 war 中。

## 6。更进一步

如果需要，有几种方法可以为该解决方案增加额外的灵活性。

一种方法是对属性文件的名称使用更复杂的编码**，不仅指定它们将被使用的环境，还指定更多的信息(比如持久性提供者)。例如，我们可以使用以下类型的属性:`persistence-h2.properties`、`persistence-mysql.properties`或者更具体的:`persistence-dev_h2.properties`、`persistence-staging_mysql.properties`、`persistence-production_amazonRDS.properties`。**

这种命名约定的优点——它仅仅是一个约定,因为整个方法没有任何变化——就是透明。现在，只需查看名称，就可以清楚地了解配置的用途:

*   **`persistence-dev_h2.properties`**:`dev`环境的持久性提供者是一个轻量级、内存中的 H2 数据库
*   **`persistence-staging_mysql.properties`**:`staging`环境的持久性提供者是一个 MySQL 实例
*   **`persistence-production_amazon_rds.propertie`** :环境`production`的持久性提供者是 Amazon RDS

## 7。结论

本文讨论了在 Spring 中进行特定于环境的配置的灵活解决方案。使用配置文件[的替代解决方案可在此处](https://web.archive.org/web/20221001115719/https://www.javacodegeeks.com/2012/06/spring-31-profiles-and-tomcat.html "Using Profiles to configure the project")找到。

该解决方案的实现可以在 GitHub 项目中找到——这是一个基于 Maven 的项目，因此它应该很容易导入和运行。