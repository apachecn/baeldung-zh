# 用春天 MVC 和百里香叶保护 CSRF

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/csrf-thymeleaf-with-spring-security>

## 1。简介

[百里叶](https://web.archive.org/web/20221208143856/http://www.thymeleaf.org/)是一个 Java 模板引擎，用于处理和创建 HTML、XML、JavaScript、CSS 和明文。关于百里香和春天的介绍，请看[这篇文章](/web/20221208143856/https://www.baeldung.com/thymeleaf-in-spring-mvc)。

在这篇文章中，我们将讨论如何使用百里香应用程序在 Spring MVC 中**防止跨站点请求伪造(CSRF)攻击**。更具体地说，我们将测试 HTTP POST 方法的 CSRF 攻击。

CSRF 是一种攻击，它迫使最终用户在当前已通过身份验证的 web 应用程序中执行不需要的操作。

## 2。Maven 依赖关系

首先，让我们看看将百里香与 Spring 集成所需的配置。在我们的依赖关系中需要`thymeleaf-spring`库:

```java
<dependency>
    <groupId>org.thymeleaf</groupId>
    <artifactId>thymeleaf</artifactId>
    <version>3.0.11.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.thymeleaf</groupId>
    <artifactId>thymeleaf-spring5</artifactId>
    <version>3.0.11.RELEASE</version>
</dependency>
```

注意，对于 Spring 4 项目，必须使用`thymeleaf-spring4`库来代替`thymeleaf-spring5`。依赖关系的最新版本可以在[这里](https://web.archive.org/web/20221208143856/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.thymeleaf%22%20AND%20a%3A%22thymeleaf-spring5%22)找到。

此外，为了使用 Spring 安全性，我们需要添加以下依赖项:

```java
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-web</artifactId>
    <version>5.7.3</version>
</dependency>
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-config</artifactId>
    <version>5.7.3</version>
</dependency> 
```

两个与 Spring 安全相关的库的最新版本可以在这里[和](https://web.archive.org/web/20221208143856/https://search.maven.org/classic/#search%7Cgav%7C1%7Ca%3A%22spring-security-web%22)[这里](https://web.archive.org/web/20221208143856/https://search.maven.org/classic/#search%7Cgav%7C1%7Ca%3A%22spring-security-config%22)获得。

## 3。Java 配置

除了此处提到的[百里香配置，我们还需要添加 Spring 安全配置。为了做到这一点，我们需要创建类:](/web/20221208143856/https://www.baeldung.com/thymeleaf-in-spring-mvc)

```java
@Configuration
@EnableWebSecurity
@EnableGlobalMethodSecurity(securedEnabled = true, prePostEnabled = true)
public class WebMVCSecurity {

    @Bean
    public InMemoryUserDetailsManager userDetailsService() {
        UserDetails user = User.withUsername("user1")
            .password("{noop}user1Pass")
            .authorities("ROLE_USER")
            .build();

        return new InMemoryUserDetailsManager(user);
    }

    @Bean
    public WebSecurityCustomizer webSecurityCustomizer() {
        return (web) -> web.ignoring()
            .antMatchers("/resources/**");
    }

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http.authorizeRequests()
            .anyRequest()
            .authenticated()
            .and()
            .httpBasic();
        return http.build();
    }
}
```

有关安全配置的更多详细信息和描述，我们可以参考带有 Spring 的[安全系列。](/web/20221208143856/https://www.baeldung.com/security-spring)

【Java 配置默认启用 CSRF 保护。为了禁用这个有用的特性，我们需要在`configure(…)`方法中添加这个:

```java
.csrf().disable()
```

在 XML 配置中，我们需要手动指定 CSRF 保护，否则将不起作用:

```java
<security:http 
  auto-config="true"
  disable-url-rewriting="true" 
  use-expressions="true">
    <security:csrf />

    <!-- Remaining configuration ... -->
</security:http>
```

还请注意，如果我们使用带有登录表单的登录页面，我们需要在代码中手动将 CSRF 令牌作为隐藏参数包含在登录表单中:

```java
<input 
  type="hidden" 
  th:name="${_csrf.parameterName}" 
  th:value="${_csrf.token}" />
```

对于剩余的表单，CSRF 令牌将自动添加到具有隐藏输入的表单中:

```java
<input 
  type="hidden" 
  name="_csrf"
  value="32e9ae18-76b9-4330-a8b6-08721283d048" /> 
<!-- Example token -->
```

## 4。视图配置

让我们继续讨论 HTML 文件的主要部分，包括表单操作和测试过程的创建。在第一个视图中，我们尝试向列表中添加新学生:

```java
<!DOCTYPE html>
<html 
	xmlns:th="http://www.thymeleaf.org">
<head>
<title>Add Student</title>
</head>
<body>
    <h1>Add Student</h1>
        <form action="#" th:action="@{/saveStudent}" th:object="${student}"
          method="post">
            <ul>
                <li th:errors="*{id}" />
                <li th:errors="*{name}" />
                <li th:errors="*{gender}" />
                <li th:errors="*{percentage}" />
            </ul>
    <!-- Remaining part of HTML -->
    </form>
</body>
</html>
```

在这个视图中，我们通过提供`id`、`name`、`gender`和`percentage`向列表中添加一个学生(可选，如表单验证中所述)。在执行这个表单之前，我们需要提供`user`和`password`，以便在 web 应用程序中验证我们。

### 4.1。浏览器 CSRF 攻击测试

现在我们进入第二个 HTML 视图。它的目的是试图做 CSRF 的攻击:

```java
<!DOCTYPE html>
<html>
<head>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8"/>
</head>
<body>
<form action="http://localhost:8080/spring-thymeleaf/saveStudent" method="post">
    <input type="hidden" name="payload" value="CSRF attack!"/>
    <input type="submit" />
</form>
</body>
</html>
```

我们知道动作 URL 是`http://localhost:8080/spring-thymeleaf/saveStudent`。黑客想要访问该页面来执行攻击。

为了进行测试，请在另一个浏览器中打开 HTML 文件，而不要登录到应用程序。当您尝试提交表格时，我们会收到以下页面:

[![Zrzut-ekranu](img/5a66fde67337fdb9b8860a5cdc8878d5.png)](/web/20221208143856/https://www.baeldung.com/wp-content/uploads/2016/09/Zrzut-ekranu-2016-09-22-23.02.24-1024x171.png)

我们的请求被拒绝，因为我们发送的请求没有 CSRF 令牌。

请注意，使用 HTTP 会话是为了存储 CSRF 令牌。当发送请求时，Spring 将生成的令牌与存储在会话中的令牌进行比较，以确认用户没有受到攻击。

### 4.2。JUnit CSRF 攻击测试

如果您不想使用浏览器测试 CSRF 攻击，您也可以通过快速集成测试来完成；让我们从那个测试的 Spring 配置开始:

```java
@RunWith(SpringJUnit4ClassRunner.class)
@WebAppConfiguration
@ContextConfiguration(classes = { 
  WebApp.class, WebMVCConfig.class, WebMVCSecurity.class, InitSecurity.class })
public class CsrfEnabledIntegrationTest {

    // configuration

}
```

继续进行实际的测试:

```java
@Test
public void addStudentWithoutCSRF() throws Exception {
    mockMvc.perform(post("/saveStudent").contentType(MediaType.APPLICATION_JSON)
      .param("id", "1234567").param("name", "Joe").param("gender", "M")
      .with(testUser())).andExpect(status().isForbidden());
}

@Test
public void addStudentWithCSRF() throws Exception {
    mockMvc.perform(post("/saveStudent").contentType(MediaType.APPLICATION_JSON)
      .param("id", "1234567").param("name", "Joe").param("gender", "M")
      .with(testUser()).with(csrf())).andExpect(status().isOk());
}
```

由于缺少 CSRF 令牌，第一个测试将导致禁止状态，而第二个测试将正确执行。

## 5。结论

在本文中，我们讨论了如何使用 Spring Security 和百里香框架来防止 CSRF 攻击。

本教程的完整实现可以在 GitHub 项目中找到。