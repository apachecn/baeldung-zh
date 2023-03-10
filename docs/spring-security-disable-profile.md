# 禁用 Spring Boot 配置文件的安全性

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-security-disable-profile>

## 1.概观

在本教程中，我们将看看如何对给定的配置文件禁用 Spring 安全性。

## 2.配置

首先，让我们定义一个简单地允许所有请求的安全配置。

我们可以通过注册一个`WebSecurityCustomizer` bean 并忽略所有路径的请求来实现这一点:

```java
@Configuration
public class ApplicationNoSecurity {

    @Bean
    public WebSecurityCustomizer webSecurityCustomizer() {
        return (web) -> web.ignoring()
            .antMatchers("/**");
    }
}
```

请记住，这不仅会关闭身份验证，还会关闭任何像 XSS 这样的安全保护。

## 3.指定配置文件

现在我们只想为一个给定的概要文件激活这个配置。

假设我们有一个不需要安全性的单元测试套件。如果这个测试套件使用名为“test”的**概要文件运行，我们可以简单地用`@Profile`** 对我们的配置进行**注释:**

```java
@Configuration
@Profile("test")
public class ApplicationNoSecurity {

    @Bean
    public WebSecurityCustomizer webSecurityCustomizer() {
        return (web) -> web.ignoring()
            .antMatchers("/**");
    }
}
```

因此，我们的测试环境将会不同，这可能是我们不希望的。或者，我们可以打开安全，使用 [Spring Security 的测试支持](/web/20221128041950/https://www.baeldung.com/spring-security-method-security#testing-method-security)。

## 4.结论

在本教程中，我们展示了如何为特定的概要文件禁用 Spring 安全性。

和往常一样，完整的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221128041950/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-security)