# 注册–通过电子邮件激活新账户

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/registration-verify-user-by-email>

[This article is part of a series:](javascript:void(0);)[• Spring Security Registration Tutorial](/web/20220926180610/https://www.baeldung.com/spring-security-registration)
[• The Registration Process With Spring Security](/web/20220926180610/https://www.baeldung.com/registration-with-spring-mvc-and-spring-security)
• Registration – Activate a New Account by Email (current article)[• Spring Security Registration – Resend Verification Email](/web/20220926180610/https://www.baeldung.com/spring-security-registration-verification-email)
[• Registration with Spring Security – Password Encoding](/web/20220926180610/https://www.baeldung.com/spring-security-registration-password-encoding-bcrypt)
[• The Registration API becomes RESTful](/web/20220926180610/https://www.baeldung.com/registration-restful-api)
[• Spring Security – Reset Your Password](/web/20220926180610/https://www.baeldung.com/spring-security-registration-i-forgot-my-password)
[• Registration – Password Strength and Rules](/web/20220926180610/https://www.baeldung.com/registration-password-strength-and-rules)
[• Updating your Password](/web/20220926180610/https://www.baeldung.com/updating-your-password/)

## 1。概述

这篇文章继续了正在进行的 **`Registration with Spring Security`系列** 的[部分，它是注册过程中缺失的部分之一——**验证用户的电子邮件以确认他们的帐户**。](/web/20220926180610/https://www.baeldung.com/spring-security-registration)

注册确认机制会强制用户回复成功注册后发送的“`**Confirm Registration**`”电子邮件，以验证其电子邮件地址并激活其帐户。用户通过点击通过电子邮件发送给他们的唯一激活链接来实现这一点。

按照这个逻辑，新注册的用户在这个过程完成之前将不能登录系统。

## 2。验证令牌

我们将使用一个简单的验证令牌作为验证用户的关键构件。

### 2.1。`VerificationToken`实体

`VerificationToken`实体必须满足以下标准:

1.  它必须链接回`User` (通过单向关系)
2.  它将在注册后立即创建
3.  它将在创建后的 24 小时内**到期**
4.  具有唯一的、随机生成的值

需求 2 和 3 是注册逻辑的一部分。另外两个在一个简单的`VerificationToken`实体中实现，如示例 2.1 中的实体。：

**例 2.1。**

```java
@Entity
public class VerificationToken {
    private static final int EXPIRATION = 60 * 24;

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;

    private String token;

    @OneToOne(targetEntity = User.class, fetch = FetchType.EAGER)
    @JoinColumn(nullable = false, name = "user_id")
    private User user;

    private Date expiryDate;

    private Date calculateExpiryDate(int expiryTimeInMinutes) {
        Calendar cal = Calendar.getInstance();
        cal.setTime(new Timestamp(cal.getTime().getTime()));
        cal.add(Calendar.MINUTE, expiryTimeInMinutes);
        return new Date(cal.getTime().getTime());
    }

    // standard constructors, getters and setters
}
```

注意用户身上的`nullable = false`，保证`VerificationToken<` - > `User`关联中数据的完整性和一致性。

### 2.2。将`enabled`字段添加到`User`

最初，当`User`被注册时，该`enabled`字段将被设置为`false`。在账户验证过程中，如果成功，它将变成`true`。

让我们从将字段添加到我们的 `User`实体开始:

```java
public class User {
    ...
    @Column(name = "enabled")
    private boolean enabled;

    public User() {
        super();
        this.enabled=false;
    }
    ...
}
```

请注意我们如何将该字段的默认值设置为`false`。

## 3。账户注册期间

让我们向用户注册用例添加两个额外的业务逻辑:

1.  为用户生成`VerificationToken`并持久化它
2.  发送电子邮件进行账户确认，其中包括一个带有`VerificationToken's` 值的确认链接

### 3.1。使用 Spring 事件创建令牌并发送验证电子邮件

这两个额外的逻辑不应该由控制器直接执行，因为它们是“并行的”后端任务。

控制器将发布一个 Spring `ApplicationEvent`来触发这些任务的执行。这就像注入*ApplicationEventPublisher*然后用它来发布注册完成一样简单。

例 3.1。显示了这个简单的逻辑:

**例 3.1。**

```java
@Autowired
ApplicationEventPublisher eventPublisher

@PostMapping("/user/registration")
public ModelAndView registerUserAccount(
  @ModelAttribute("user") @Valid UserDto userDto, 
  HttpServletRequest request, Errors errors) { 

    try {
        User registered = userService.registerNewUserAccount(userDto);

        String appUrl = request.getContextPath();
        eventPublisher.publishEvent(new OnRegistrationCompleteEvent(registered, 
          request.getLocale(), appUrl));
    } catch (UserAlreadyExistException uaeEx) {
        ModelAndView mav = new ModelAndView("registration", "user", userDto);
        mav.addObject("message", "An account for that username/email already exists.");
        return mav;
    } catch (RuntimeException ex) {
        return new ModelAndView("emailError", "user", userDto);
    }

    return new ModelAndView("successRegister", "user", userDto);
}
```

另一件需要注意的事情是围绕事件发布的`try catch`块。每当发布事件(在本例中是发送电子邮件)后执行的逻辑中出现异常时，这段代码都会显示一个错误页面。

### 3.2。事件和监听器

现在让我们看看我们的控制器发出的这个新的`OnRegistrationCompleteEvent`的实际实现，以及将要处理它的监听器:

**例 3.2.1。**—`OnRegistrationCompleteEvent`

```java
public class OnRegistrationCompleteEvent extends ApplicationEvent {
    private String appUrl;
    private Locale locale;
    private User user;

    public OnRegistrationCompleteEvent(
      User user, Locale locale, String appUrl) {
        super(user);

        this.user = user;
        this.locale = locale;
        this.appUrl = appUrl;
    }

    // standard getters and setters
}
```

**例 3.2.2。**–**`The RegistrationListener`**处理`OnRegistrationCompleteEvent`

```java
@Component
public class RegistrationListener implements 
  ApplicationListener<OnRegistrationCompleteEvent> {

    @Autowired
    private IUserService service;

    @Autowired
    private MessageSource messages;

    @Autowired
    private JavaMailSender mailSender;

    @Override
    public void onApplicationEvent(OnRegistrationCompleteEvent event) {
        this.confirmRegistration(event);
    }

    private void confirmRegistration(OnRegistrationCompleteEvent event) {
        User user = event.getUser();
        String token = UUID.randomUUID().toString();
        service.createVerificationToken(user, token);

        String recipientAddress = user.getEmail();
        String subject = "Registration Confirmation";
        String confirmationUrl 
          = event.getAppUrl() + "/regitrationConfirm?token=" + token;
        String message = messages.getMessage("message.regSucc", null, event.getLocale());

        SimpleMailMessage email = new SimpleMailMessage();
        email.setTo(recipientAddress);
        email.setSubject(subject);
        email.setText(message + "\r\n" + "http://localhost:8080" + confirmationUrl);
        mailSender.send(email);
    }
}
```

在这里，`confirmRegistration`方法将接收`OnRegistrationCompleteEvent`，从中提取所有必要的`User`信息，创建验证令牌，持久化它，然后在“`Confirm Registration`链接中将其作为参数发送。

如上所述，任何由`JavaMailSender`抛出的`javax.mail.AuthenticationFailedException`都将由控制器处理。

### 3.3。处理验证令牌参数

当用户收到“`Confirm Registration`”链接时，他们应该点击它。

一旦完成，控制器将提取结果 GET 请求中令牌参数的值，并使用它来启用`User`。

让我们在例 3.3.1 中看看这个过程。：

**例 3.3.1。–`RegistrationController`处理注册确认**

```java
@Autowired
private IUserService service;

@GetMapping("/regitrationConfirm")
public String confirmRegistration
  (WebRequest request, Model model, @RequestParam("token") String token) {

    Locale locale = request.getLocale();

    VerificationToken verificationToken = service.getVerificationToken(token);
    if (verificationToken == null) {
        String message = messages.getMessage("auth.message.invalidToken", null, locale);
        model.addAttribute("message", message);
        return "redirect:/badUser.html?lang=" + locale.getLanguage();
    }

    User user = verificationToken.getUser();
    Calendar cal = Calendar.getInstance();
    if ((verificationToken.getExpiryDate().getTime() - cal.getTime().getTime()) <= 0) {
        String messageValue = messages.getMessage("auth.message.expired", null, locale)
        model.addAttribute("message", messageValue);
        return "redirect:/badUser.html?lang=" + locale.getLanguage();
    } 

    user.setEnabled(true); 
    service.saveRegisteredUser(user); 
    return "redirect:/login.html?lang=" + request.getLocale().getLanguage(); 
}
```

如果出现以下情况，用户将被重定向到带有相应消息的错误页面:

1.  `VerificationToken`不存在，由于某种原因或
2.  `VerificationToken`已经过期

**见例 3.3.2。**查看错误页面。

**例 3.3.2。— `badUser.html`**

```java
<html>
<body>
    <h1 th:text="${param.message[0]}>Error Message</h1>
    <a th:href="@{/registration.html}" 
      th:text="#{label.form.loginSignUp}">signup</a>
</body>
</html>
```

如果没有发现错误，则用户被启用。

在处理 `VerificationToken`检查和到期场景方面有两个改进的机会:

1.  我们可以使用 Cron 作业在后台检查令牌过期情况
2.  一旦令牌过期，我们可以给用户机会获得新的令牌

我们将在以后的文章中推迟新令牌的生成，并假设用户在这里确实成功验证了他们的令牌。

## 4。在登录过程中添加账户激活检查

我们需要添加代码来检查用户是否被启用:

让我们在例 4.1 中看到这一点。其中显示了`MyUserDetailsService`的`loadUserByUsername`方法。

**例 4.1。**

```java
@Autowired
UserRepository userRepository;

public UserDetails loadUserByUsername(String email) 
  throws UsernameNotFoundException {

    boolean enabled = true;
    boolean accountNonExpired = true;
    boolean credentialsNonExpired = true;
    boolean accountNonLocked = true;
    try {
        User user = userRepository.findByEmail(email);
        if (user == null) {
            throw new UsernameNotFoundException(
              "No user found with username: " + email);
        }

        return new org.springframework.security.core.userdetails.User(
          user.getEmail(), 
          user.getPassword().toLowerCase(), 
          user.isEnabled(), 
          accountNonExpired, 
          credentialsNonExpired, 
          accountNonLocked, 
          getAuthorities(user.getRole()));
    } catch (Exception e) {
        throw new RuntimeException(e);
    }
}
```

正如我们所看到的，现在`MyUserDetailsService`不使用用户的`enabled`标志——因此它只允许启用用户身份验证。

现在，我们将添加一个`AuthenticationFailureHandler`来定制来自`MyUserDetailsService`的异常消息。我们的`CustomAuthenticationFailureHandler`如例 4.2 所示。`:`

**例 4.2。–`CustomAuthenticationFailureHandler`:**

```java
@Component
public class CustomAuthenticationFailureHandler extends SimpleUrlAuthenticationFailureHandler {

    @Autowired
    private MessageSource messages;

    @Autowired
    private LocaleResolver localeResolver;

    @Override
    public void onAuthenticationFailure(HttpServletRequest request, 
      HttpServletResponse response, AuthenticationException exception)
      throws IOException, ServletException {
        setDefaultFailureUrl("/login.html?error=true");

        super.onAuthenticationFailure(request, response, exception);

        Locale locale = localeResolver.resolveLocale(request);

        String errorMessage = messages.getMessage("message.badCredentials", null, locale);

        if (exception.getMessage().equalsIgnoreCase("User is disabled")) {
            errorMessage = messages.getMessage("auth.message.disabled", null, locale);
        } else if (exception.getMessage().equalsIgnoreCase("User account has expired")) {
            errorMessage = messages.getMessage("auth.message.expired", null, locale);
        }

        request.getSession().setAttribute(WebAttributes.AUTHENTICATION_EXCEPTION, errorMessage);
    }
}
```

我们需要修改`login.html`来显示错误消息。

**例 4.3。–在`login.html` :** 显示错误信息

```java
<div th:if="${param.error != null}" 
  th:text="${session[SPRING_SECURITY_LAST_EXCEPTION]}">error</div>
```

## 5。调整持久层

现在让我们提供一些操作的实际实现，包括验证令牌和用户。

我们将涵盖:

1.  新的`VerificationTokenRepository`
2.  `IUserInterface`中的新方法及其实现需要新的 CRUD 操作

示例 5.1–5.3。展示新的接口和实现:

**例 5.1。**—`VerificationTokenRepository`

```java
public interface VerificationTokenRepository 
  extends JpaRepository<VerificationToken, Long> {

    VerificationToken findByToken(String token);

    VerificationToken findByUser(User user);
}
```

**例 5.2。**–界面`IUserService`

```java
public interface IUserService {

    User registerNewUserAccount(UserDto userDto) 
      throws UserAlreadyExistException;

    User getUser(String verificationToken);

    void saveRegisteredUser(User user);

    void createVerificationToken(User user, String token);

    VerificationToken getVerificationToken(String VerificationToken);
}
```

**例 5.3。**`UserService`

```java
@Service
@Transactional
public class UserService implements IUserService {
    @Autowired
    private UserRepository repository;

    @Autowired
    private VerificationTokenRepository tokenRepository;

    @Override
    public User registerNewUserAccount(UserDto userDto) 
      throws UserAlreadyExistException {

        if (emailExist(userDto.getEmail())) {
            throw new UserAlreadyExistException(
              "There is an account with that email adress: " 
              + userDto.getEmail());
        }

        User user = new User();
        user.setFirstName(userDto.getFirstName());
        user.setLastName(userDto.getLastName());
        user.setPassword(userDto.getPassword());
        user.setEmail(userDto.getEmail());
        user.setRole(new Role(Integer.valueOf(1), user));
        return repository.save(user);
    }

    private boolean emailExist(String email) {
        return userRepository.findByEmail(email) != null;
    }

    @Override
    public User getUser(String verificationToken) {
        User user = tokenRepository.findByToken(verificationToken).getUser();
        return user;
    }

    @Override
    public VerificationToken getVerificationToken(String VerificationToken) {
        return tokenRepository.findByToken(VerificationToken);
    }

    @Override
    public void saveRegisteredUser(User user) {
        repository.save(user);
    }

    @Override
    public void createVerificationToken(User user, String token) {
        VerificationToken myToken = new VerificationToken(token, user);
        tokenRepository.save(myToken);
    }
}
```

## 6。结论

在本文中，我们扩展了注册过程，将基于电子邮件的账户激活过程包括在内**。**

帐户激活逻辑要求通过电子邮件向用户发送验证令牌，以便他们可以将其发送回控制器来验证他们的身份。

使用 Spring 安全教程注册的实现可以在 GitHub 项目中找到——这是一个基于 Eclipse 的项目，所以它应该很容易导入和运行。

Next **»**[Spring Security Registration – Resend Verification Email](/web/20220926180610/https://www.baeldung.com/spring-security-registration-verification-email)**«** Previous[The Registration Process With Spring Security](/web/20220926180610/https://www.baeldung.com/registration-with-spring-mvc-and-spring-security)