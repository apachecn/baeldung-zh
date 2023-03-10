# Spring Boot 的自定义验证消息源

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-custom-validation-message-source>

## 1。概述

是 Spring 应用程序中的一个强大特性。这有助于应用程序开发人员通过编写大量额外的代码来处理各种复杂的场景，例如特定于环境的配置、国际化或可配置的值。

另一个场景是将默认的验证消息修改为更加用户友好/定制的消息。

在本教程中，**我们将看到如何使用 Spring Boot** 在应用程序中配置和管理自定义验证`MessageSource`。

## 2。Maven 依赖关系

让我们从添加必要的 Maven 依赖项开始:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```

你可以在 Maven Central 上找到这些库的最新版本。

## 3。自定义验证消息示例

让我们考虑一个场景，我们必须开发一个支持多种语言的应用程序。如果用户没有提供正确的细节作为输入，我们希望根据用户的地区显示错误消息。

让我们以一个登录表单 bean 为例:

```java
public class LoginForm {

    @NotEmpty(message = "{email.notempty}")
    @Email
    private String email;

    @NotNull
    private String password;

    // standard getter and setters
}
```

这里，我们添加了验证约束，用于验证是否根本没有提供电子邮件，或者提供了电子邮件，但没有遵循标准的电子邮件地址样式。

为了显示定制的和特定于地区的消息，我们可以为`@NotEmpty`注释提供一个占位符。

`MessageSource`配置将从属性文件中解析出`email.notempty `属性。

## 4。定义`MessageSource`比恩

应用程序上下文将消息解析委托给一个名为`messageSource.` 的 bean

`ReloadableResourceBundleMessageSource`是最常见的`MessageSource`实现，它从不同地区的资源包中解析消息:

```java
@Bean
public MessageSource messageSource() {
    ReloadableResourceBundleMessageSource messageSource
      = new ReloadableResourceBundleMessageSource();

    messageSource.setBasename("classpath:messages");
    messageSource.setDefaultEncoding("UTF-8");
    return messageSource;
}
```

这里，提供`basename`很重要，因为特定于地区的文件名将根据提供的名称进行解析。

## 5。定义 `LocalValidatorFactoryBean `

**要在属性文件中使用自定义名称消息，我们需要定义一个`LocalValidatorFactoryBean`并注册`messageSource:`**

```java
@Bean
public LocalValidatorFactoryBean getValidator() {
    LocalValidatorFactoryBean bean = new LocalValidatorFactoryBean();
    bean.setValidationMessageSource(messageSource());
    return bean;
}
```

但是，请注意，如果我们已经扩展了`WebMvcConfigurerAdapter`，为了避免自定义验证器被忽略，我们必须通过覆盖父类的`getValidator()`方法来设置验证器。

现在我们可以定义一个属性消息，如下所示:

”`email.notempty=<Custom_Message>”`

代替

`“javax.validation.constraints.NotEmpty.message=<Custom_message>”`

## 6。定义属性文件

**最后一步是在`src/main/resources`目录下创建一个属性文件，其名称在第 4 步的`basename`中提供:**

```java
# messages.properties
email.notempty=Please provide valid email id.
```

在这里，我们可以利用国际化的优势。假设我们想以法语用户的语言向他们显示消息。

在这种情况下，我们必须在相同的位置再添加一个名为`messages_fr.properties`的属性文件(根本不需要修改代码):

```java
# messages_fr.properties
email.notempty=Veuillez fournir un identifiant de messagerie valide.
```

## 7。结论

在本文中，我们介绍了如果事先正确配置，如何在不修改代码的情况下更改默认验证消息。

我们还可以利用对国际化的支持，使应用程序更加用户友好。

和往常一样，完整的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221103025345/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-mvc)