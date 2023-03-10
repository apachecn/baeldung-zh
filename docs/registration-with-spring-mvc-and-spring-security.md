# Spring Security 的注册过程

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/registration-with-spring-mvc-and-spring-security>

[This article is part of a series:](javascript:void(0);)[• Spring Security Registration Tutorial](/web/20220926191758/https://www.baeldung.com/spring-security-registration)
• The Registration Process With Spring Security (current article)[• Registration – Activate a New Account by Email](/web/20220926191758/https://www.baeldung.com/registration-verify-user-by-email)
[• Spring Security Registration – Resend Verification Email](/web/20220926191758/https://www.baeldung.com/spring-security-registration-verification-email)
[• Registration with Spring Security – Password Encoding](/web/20220926191758/https://www.baeldung.com/spring-security-registration-password-encoding-bcrypt)
[• The Registration API becomes RESTful](/web/20220926191758/https://www.baeldung.com/registration-restful-api)
[• Spring Security – Reset Your Password](/web/20220926191758/https://www.baeldung.com/spring-security-registration-i-forgot-my-password)
[• Registration – Password Strength and Rules](/web/20220926191758/https://www.baeldung.com/registration-password-strength-and-rules)
[• Updating your Password](/web/20220926191758/https://www.baeldung.com/updating-your-password/)

## 1。概述

在本教程中，我们将使用 Spring Security 实现一个基本的注册过程。我们将建立在我们在[上一篇文章](/web/20220926191758/https://www.baeldung.com/spring-security-login-error-handling-localization)中探索的概念之上，在那篇文章中我们讨论了登录。

这里的目标是给**添加一个完整的注册过程**，允许用户注册，并验证和保存用户数据。

## 延伸阅读:

## [Servlet 3 异步支持 Spring MVC 和 Spring Security](/web/20220926191758/https://www.baeldung.com/spring-mvc-async-security)

Quick intro to the Spring Security support for async requests in Spring MVC.[Read more](/web/20220926191758/https://www.baeldung.com/spring-mvc-async-security) →

## [春天用百里香叶安全](/web/20220926191758/https://www.baeldung.com/spring-security-thymeleaf)

A quick guide to integrating Spring Security and Thymeleaf[Read more](/web/20220926191758/https://www.baeldung.com/spring-security-thymeleaf) →

## [Spring 安全–缓存控制头](/web/20220926191758/https://www.baeldung.com/spring-security-cache-control-headers)

A guide to controlling HTTP cache control headers with Spring Security.[Read more](/web/20220926191758/https://www.baeldung.com/spring-security-cache-control-headers) →

## 2。注册页面

首先，我们将实现一个简单的注册页面，显示以下字段:

*   `name`(名和姓)
*   `email`
*   `password`(和密码确认栏)

下面的例子显示了一个简单的`registration.html`页面:

**例 2.1。**

```java
<html>
<body>
<h1 th:text="#{label.form.title}">form</h1>
<form action="/" th:object="${user}" method="POST" enctype="utf8">
    <div>
        <label th:text="#{label.user.firstName}">first</label>
        <input th:field="*{firstName}"/>
        <p th:each="error: ${#fields.errors('firstName')}" 
          th:text="${error}">Validation error</p>
    </div>
    <div>
        <label th:text="#{label.user.lastName}">last</label>
        <input th:field="*{lastName}"/>
        <p th:each="error : ${#fields.errors('lastName')}" 
          th:text="${error}">Validation error</p>
    </div>
    <div>
        <label th:text="#{label.user.email}">email</label>
        <input type="email" th:field="*{email}"/>
        <p th:each="error : ${#fields.errors('email')}" 
          th:text="${error}">Validation error</p>
    </div>
    <div>
        <label th:text="#{label.user.password}">password</label>
        <input type="password" th:field="*{password}"/>
        <p th:each="error : ${#fields.errors('password')}" 
          th:text="${error}">Validation error</p>
    </div>
    <div>
        <label th:text="#{label.user.confirmPass}">confirm</label>
        <input type="password" th:field="*{matchingPassword}"/>
    </div>
    <button type="submit" th:text="#{label.form.submit}">submit</button>
</form>

<a th:href="@{/login.html}" th:text="#{label.form.loginLink}">login</a>
</body>
</html>
```

## 3。 **用户 DTO 对象**

我们需要一个数据传输对象将所有注册信息发送到我们的 Spring 后端。 **DTO** 对象应该拥有我们稍后创建和填充`User`对象时需要的所有信息:

```java
public class UserDto {
    @NotNull
    @NotEmpty
    private String firstName;

    @NotNull
    @NotEmpty
    private String lastName;

    @NotNull
    @NotEmpty
    private String password;
    private String matchingPassword;

    @NotNull
    @NotEmpty
    private String email;

    // standard getters and setters
}
```

注意，我们在 DTO 对象的字段上使用了标准的`javax.validation`注释。稍后，我们还将**实现我们自己的定制验证注释**来验证电子邮件地址的格式，以及密码确认(参见**第 5 节)。**

## 4。注册控制器

在`login`页面上的一个`Sign-Up`链接将把用户带到 `registration`页面。该页面的后端驻留在注册控制器中，并映射到`“/user/registration”`:

**例 4.1。`showRegistration`法**

```java
@GetMapping("/user/registration")
public String showRegistrationForm(WebRequest request, Model model) {
    UserDto userDto = new UserDto();
    model.addAttribute("user", userDto);
    return "registration";
}
```

当控制器收到请求`“/user/registration,”`时，它创建新的`UserDto`对象，该对象支持`registration`表单，绑定它并返回。

## 5。验证注册数据

接下来，我们将查看控制器在注册新帐户时将执行的验证:

1.  所有必填字段均已填写(无空字段)。
2.  电子邮件地址有效(格式正确)。
3.  密码确认字段与密码字段相匹配。
4.  该帐户不存在。

### 5.1。内置验证

对于简单的检查，我们将在 DTO 对象上使用现成的 bean 验证注释。这些是类似`@NotNull`、`@NotEmpty`等的注释。

然后，为了触发验证过程，我们将简单地用`@Valid`注释来注释控制器层中的对象:

```java
public ModelAndView registerUserAccount(@ModelAttribute("user") @Valid UserDto userDto,
  HttpServletRequest request, Errors errors) {
    //...
}
```

### 5.2。检查电子邮件有效性的自定义验证

然后，我们将验证电子邮件地址，并确保其格式正确。为此，我们将构建一个**自定义验证器**，以及一个**自定义验证注释；**我们就叫它`@ValidEmail`。

值得注意的是，我们正在滚动我们自己的定制注释**而不是 Hibernate 的** `@Email`，因为 Hibernate 认为旧的内部网地址格式`[[email protected]](/web/20220926191758/https://www.baeldung.com/cdn-cgi/l/email-protection),`是有效的(参见 [Stackoverflow](https://web.archive.org/web/20220926191758/https://stackoverflow.com/questions/4459474/hibernate-validator-email-accepts-askstackoverflow-as-valid) 文章)，这并不好。

这是电子邮件验证注释和自定义验证器:

**例 5.2.1。电子邮件验证的自定义注释**

```java
@Target({TYPE, FIELD, ANNOTATION_TYPE})
@Retention(RUNTIME)
@Constraint(validatedBy = EmailValidator.class)
@Documented
public @interface ValidEmail {
    String message() default "Invalid email";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
}
```

注意，我们在`FIELD`级别定义了注释，因为这是它在概念上适用的地方。

**例 5.2.2。`EmailValidato`风俗** r:

```java
public class EmailValidator 
  implements ConstraintValidator<ValidEmail, String> {

    private Pattern pattern;
    private Matcher matcher;
    private static final String EMAIL_PATTERN = "^[_A-Za-z0-9-+]+
        (.[_A-Za-z0-9-]+)*@" + "[A-Za-z0-9-]+(.[A-Za-z0-9]+)*
        (.[A-Za-z]{2,})$"; 
    @Override
    public void initialize(ValidEmail constraintAnnotation) {
    }
    @Override
    public boolean isValid(String email, ConstraintValidatorContext context){
        return (validateEmail(email));
    } 
    private boolean validateEmail(String email) {
        pattern = Pattern.compile(EMAIL_PATTERN);
        matcher = pattern.matcher(email);
        return matcher.matches();
    }
}
```

然后我们将**在我们的`UserDto`实现中使用新的注释**:

```java
@ValidEmail
@NotNull
@NotEmpty
private String email;
```

### 5.3。使用自定义验证进行密码确认

我们还需要一个定制的注释和验证器来确保`password`和`matchingPassword`字段匹配:

**例 5.3.1。用于验证密码确认的自定义注释**

```java
@Target({TYPE,ANNOTATION_TYPE})
@Retention(RUNTIME)
@Constraint(validatedBy = PasswordMatchesValidator.class)
@Documented
public @interface PasswordMatches {
    String message() default "Passwords don't match";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
}
```

注意，`@Target`注释表明这是一个 `TYPE`级注释。这是因为我们需要整个`UserDto`对象来执行验证。

该注释将调用的自定义验证器如下所示:

**例 5.3.2。`PasswordMatchesValidator`自定义验证器**

```java
public class PasswordMatchesValidator
  implements ConstraintValidator<PasswordMatches, Object> {

    @Override
    public void initialize(PasswordMatches constraintAnnotation) {
    }
    @Override
    public boolean isValid(Object obj, ConstraintValidatorContext context){
        UserDto user = (UserDto) obj;
        return user.getPassword().equals(user.getMatchingPassword());
    }
}
```

然后，`@PasswordMatches`注释应该应用于我们的`UserDto`对象:

```java
@PasswordMatches
public class UserDto {
    //...
}
```

当然，在整个验证过程运行时，所有的定制验证都会和所有的标准注释一起被评估。

### 5.4。检查账户是否已经存在

我们将实现的第四个检查是验证数据库中是否已经不存在帐户`email`。

这是在表单被验证后执行的，并且是在`UserService` 实现的帮助下完成的。

**例 5.4.1。控制器的`registerUserAccount`方法调用`UserService`对象**

```java
@PostMapping("/user/registration")
public ModelAndView registerUserAccount(
  @ModelAttribute("user") @Valid UserDto userDto,
  HttpServletRequest request,
  Errors errors) {

    try {
        User registered = userService.registerNewUserAccount(userDto);
    } catch (UserAlreadyExistException uaeEx) {
        mav.addObject("message", "An account for that username/email already exists.");
        return mav;
    }

    // rest of the implementation
} 
```

**例 5.4.2。`User` *服务*检查重复邮件**

```java
@Service
@Transactional
public class UserService implements IUserService {
    @Autowired
    private UserRepository repository;

    @Override
    public User registerNewUserAccount(UserDto userDto) throws UserAlreadyExistException {
        if (emailExists(userDto.getEmail())) {
            throw new UserAlreadyExistException("There is an account with that email address: "
              + userDto.getEmail());
        }

        // the rest of the registration operation
    }
    private boolean emailExists(String email) {
        return userRepository.findByEmail(email) != null;
    }
}
```

UserService 依赖于`UserRepository`类来检查数据库中是否已经存在具有给定电子邮件地址的用户。

持久层中`UserRepository`的实际实现与本文无关；然而，一个快捷的方法是[使用 Spring 数据来生成存储库层](/web/20220926191758/https://www.baeldung.com/the-persistence-layer-with-spring-data-jpa "Introduction to Spring Data JPA")。

## 6。持久化数据并完成表单处理

接下来，我们将在控制器层实现注册逻辑:

 ***例 6.1。控制器**中的`RegisterAccount`方法

```java
@PostMapping("/user/registration")
public ModelAndView registerUserAccount(
  @ModelAttribute("user") @Valid UserDto userDto,
  HttpServletRequest request,
  Errors errors) {

    try {
        User registered = userService.registerNewUserAccount(userDto);
    } catch (UserAlreadyExistException uaeEx) {
        mav.addObject("message", "An account for that username/email already exists.");
        return mav;
    }

    return new ModelAndView("successRegister", "user", userDto);
} 
```

上面代码中需要注意的事项:

1.  控制器正在返回一个`ModelAndView`对象，这是一个方便的类，用于发送绑定到视图的模型数据(`user`)。
2.  如果在验证时设置了任何错误，控制器将重定向到注册表单。

## 7。**`UserService`–登记操作**

最后，我们将在`UserService`中完成注册操作的实现:

**例 7.1。`IUserService`界面**

```java
public interface IUserService {
    User registerNewUserAccount(UserDto userDto);
}
```

**例 7.2。`UserService`班**

```java
@Service
@Transactional
public class UserService implements IUserService {
    @Autowired
    private UserRepository repository;

    @Override
    public User registerNewUserAccount(UserDto userDto) throws UserAlreadyExistException {
        if (emailExists(userDto.getEmail())) {
            throw new UserAlreadyExistException("There is an account with that email address: "
              + userDto.getEmail());
        }

        User user = new User();
        user.setFirstName(userDto.getFirstName());
        user.setLastName(userDto.getLastName());
        user.setPassword(userDto.getPassword());
        user.setEmail(userDto.getEmail());
        user.setRoles(Arrays.asList("ROLE_USER"));

        return repository.save(user);
    }

    private boolean emailExists(String email) {
        return userRepository.findByEmail(email) != null;
    }
}
```

## 8。加载安全登录的用户详细信息

在我们的前一篇文章中，登录使用了硬编码的凭证。我们将通过使用新注册的用户信息和凭证**来改变这一点。此外，我们将实现一个自定义的`UserDetailsService`来检查持久层的登录凭证。**

### 8.1。 `UserDetailsService` 的风俗

我们将从自定义用户详细信息服务实现开始:

```java
@Service
@Transactional
public class MyUserDetailsService implements UserDetailsService {

    @Autowired
    private UserRepository userRepository;

    public UserDetails loadUserByUsername(String email) throws UsernameNotFoundException {
        User user = userRepository.findByEmail(email);
        if (user == null) {
            throw new UsernameNotFoundException("No user found with username: " + email);
        }
        boolean enabled = true;
        boolean accountNonExpired = true;
        boolean credentialsNonExpired = true;
        boolean accountNonLocked = true;

        return new org.springframework.security.core.userdetails.User(
          user.getEmail(), user.getPassword().toLowerCase(), enabled, accountNonExpired,
          credentialsNonExpired, accountNonLocked, getAuthorities(user.getRoles()));
    }

    private static List<GrantedAuthority> getAuthorities (List<String> roles) {
        List<GrantedAuthority> authorities = new ArrayList<>();
        for (String role : roles) {
            authorities.add(new SimpleGrantedAuthority(role));
        }
        return authorities;
    }
}
```

### 8.2。启用新的认证提供者

为了在 Spring 安全配置中启用新的用户服务，我们只需要在 `authentication-manager`元素中添加对`UserDetailsService`的引用，并添加`UserDetailsService` bean:

**例 8.2。认证管理器和`UserDetailsService`**

```java
<authentication-manager>
    <authentication-provider user-service-ref="userDetailsService" />
</authentication-manager>

<beans:bean id="userDetailsService" class="com.baeldung.security.MyUserDetailsService" />
```

另一种选择是通过 Java 配置:

```java
@Autowired
private MyUserDetailsService userDetailsService;

@Override
protected void configure(AuthenticationManagerBuilder auth) throws Exception {
    auth.userDetailsService(userDetailsService);
}
```

## 9。结论

在本文中，我们展示了一个完整的几乎**生产就绪的注册过程**，它是用 Spring Security 和 Spring MVC 实现的。接下来，我们将讨论通过验证新用户的电子邮件来激活新注册的帐户的过程。

这篇 Spring Security REST 文章的实现可以在 GitHub 上找到[。](https://web.archive.org/web/20220926191758/https://github.com/Baeldung/spring-security-registration "The Full Registration Example Project on Github ")

Next **»**[Registration – Activate a New Account by Email](/web/20220926191758/https://www.baeldung.com/registration-verify-user-by-email)**«** Previous[Spring Security Registration Tutorial](/web/20220926191758/https://www.baeldung.com/spring-security-registration)*