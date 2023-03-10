# 更改 Spring Boot 的百里香模板目录

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-thymeleaf-template-directory>

## 1.介绍

[百里香叶](https://web.archive.org/web/20220728105348/https://www.thymeleaf.org/)是一个模板引擎，我们可以将其用于我们的 [Spring Boot 应用](/web/20220728105348/https://www.baeldung.com/spring-boot-crud-thymeleaf)。和许多事情一样， **Spring Boot 提供了一个默认位置，它期望在那里找到我们的模板**。

在这个简短的教程中，我们将看看如何改变模板的位置。之后，我们将学习如何拥有多个位置。

## 2.设置

要使用百里香叶，我们需要在我们的`pom.xml`中添加[合适的 Spring Boot 发酵剂](https://web.archive.org/web/20220728105348/https://search.maven.org/search?q=a:spring-boot-starter-thymeleaf%20AND%20g:org.springframework.boot):

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
    <versionId>2.2.2.RELEASE</versionId>
</dependency>
```

## 3.更改默认位置

**默认情况下，Spring Boot 在`src/main/resources/templates`中寻找我们的模板。**我们可以把我们的模板放在那里，组织在子目录中，没有问题。

现在，让我们假设我们有一个需求，所有的模板都驻留在一个名为`templates-2`的目录中。

让我们创建一个打招呼的模板，并将其放入`src/main/resources/templates-2`:

```java
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
<meta charset="UTF-8">
<title>Enums in Thymeleaf</title>
</head>
<body>
    <h2>Hello from 'templates/templates-2'</h2>
</body>
</html>
```

我们还需要一个控制器:

```java
@GetMapping("/hello")
public String sayHello() {
    return "hello";
}
```

基本设置完成后，让我们通过覆盖`application.properties`中的一个属性来配置 Spring Boot 使用我们的`templates-2`目录:

```java
spring.thymeleaf.prefix=classpath:/templates-2/
```

现在，当我们给我们的`HelloController`打电话时，我们会看到来自`hello.html`的问候。

## 4.使用多个位置

现在我们已经学习了如何更改默认位置，让我们看看如何使用多个模板位置。

为此，让我们创建一个`ClassLoaderTemplateResolver` bean:

```java
@Bean
public ClassLoaderTemplateResolver secondaryTemplateResolver() {
    ClassLoaderTemplateResolver secondaryTemplateResolver = new ClassLoaderTemplateResolver();
    secondaryTemplateResolver.setPrefix("templates-2/");
    secondaryTemplateResolver.setSuffix(".html");
    secondaryTemplateResolver.setTemplateMode(TemplateMode.HTML);
    secondaryTemplateResolver.setCharacterEncoding("UTF-8");
    secondaryTemplateResolver.setOrder(1);
    secondaryTemplateResolver.setCheckExistence(true);

    return secondaryTemplateResolver;
}
```

在我们的自定义 bean 中，我们将前缀设置为我们正在使用的二级模板目录:`templates-2.`我们还将`CheckExistance`标志设置为`true`。这是允许解析器在链中操作的关键。

这样配置后，我们的应用程序可以使用默认的`main/resources/templates`目录和`main/resources/templates-2`中的模板。

## 5.错误

当我们使用百里香叶时，我们可能会看到以下错误:

```java
Error resolving template [hello], template might not exist or might not be accessible
  by any of the configured Template Resolvers
```

当百里香由于某种原因找不到模板时，我们会看到这条消息。让我们来看看出现这种情况的一些可能原因，以及如何修复它们。

### 5.1.控制器中的打印错误

我们经常可以看到由于一个简单的打字错误造成的错误。首先要检查的是我们的文件名减去扩展名和我们在控制器中要求的模板完全匹配。如果我们使用子目录，我们需要确保它们也是正确的。

此外，该问题可能是某些操作系统的问题。Windows 不区分大小写，但其他操作系统区分大小写。如果一切正常，比如说，在我们本地的 Windows 机器上，我们应该研究一下这个问题，但是一旦我们部署了，就不是这样了。

### 5.2.包括控制器中的文件扩展名

因为我们的文件通常有一个扩展名，所以当我们在控制器中返回模板路径时，很自然地会包含它们。**百里香叶自动追加后缀，我们要避免提供**。

### 5.3.不使用默认位置

如果我们把模板放在除了`src/main/resources/templates`以外的地方，我们也会看到这个错误。如果我们想使用不同的位置，我们需要设置`spring.thymeleaf.prefix`属性或者创建我们自己的`ClassLoaderTemplateResolver` bean 来处理多个位置。

## 6.结论

在这个快速教程中，我们了解了百里香模板的位置。首先，我们看到了如何通过设置属性来更改默认位置。然后我们在此基础上创建了自己的`ClassLoaderTemplateResolver`来使用多个位置。

我们最后讨论了当百里香找不到我们的模板时我们将看到的错误以及如何解决它。

和往常一样，示例代码可以在 [GitHub](https://web.archive.org/web/20220728105348/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-thymeleaf-5) 上找到。