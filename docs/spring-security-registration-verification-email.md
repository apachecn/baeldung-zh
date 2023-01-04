# 春季安全注册–重新发送验证电子邮件

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-security-registration-verification-email>

[This article is part of a series:](javascript:void(0);)[• Spring Security Registration Tutorial](/web/20220529025330/https://www.baeldung.com/spring-security-registration)
[• The Registration Process With Spring Security](/web/20220529025330/https://www.baeldung.com/registration-with-spring-mvc-and-spring-security)
[• Registration – Activate a New Account by Email](/web/20220529025330/https://www.baeldung.com/registration-verify-user-by-email)
• Spring Security Registration – Resend Verification Email (current article)[• Registration with Spring Security – Password Encoding](/web/20220529025330/https://www.baeldung.com/spring-security-registration-password-encoding-bcrypt)
[• The Registration API becomes RESTful](/web/20220529025330/https://www.baeldung.com/registration-restful-api)
[• Spring Security – Reset Your Password](/web/20220529025330/https://www.baeldung.com/spring-security-registration-i-forgot-my-password)
[• Registration – Password Strength and Rules](/web/20220529025330/https://www.baeldung.com/registration-password-strength-and-rules)
[• Updating your Password](/web/20220529025330/https://www.baeldung.com/updating-your-password/)

## 1。概述

在本教程中——我们继续正在进行的 **`Registration with Spring Security`系列**,看看如何向用户重新发送验证链接，以防在他们有机会激活他们的帐户之前过期。

## 2。重新发送验证链接

首先，让我们看看当用户**请求另一个验证链接**时会发生什么，以防前一个过期。

首先，我们将使用新的`expireDate`重置现有令牌。然后，我们将向用户发送一封新的电子邮件，其中包含新的链接/令牌:

```
@GetMapping("/user/resendRegistrationToken")
public GenericResponse resendRegistrationToken(
  HttpServletRequest request, @RequestParam("token") String existingToken) {
    VerificationToken newToken = userService.generateNewVerificationToken(existingToken);

    User user = userService.getUser(newToken.getToken());
    String appUrl = 
      "http://" + request.getServerName() + 
      ":" + request.getServerPort() + 
      request.getContextPath();
    SimpleMailMessage email = 
      constructResendVerificationTokenEmail(appUrl, request.getLocale(), newToken, user);
    mailSender.send(email);

    return new GenericResponse(
      messages.getMessage("message.resendToken", null, request.getLocale()));
}
```

以及用于实际构建用户收到的电子邮件消息的实用程序—`constructResendVerificationTokenEmail()`:

```
private SimpleMailMessage constructResendVerificationTokenEmail
  (String contextPath, Locale locale, VerificationToken newToken, User user) {
    String confirmationUrl = 
      contextPath + "/regitrationConfirm.html?token=" + newToken.getToken();
    String message = messages.getMessage("message.resendToken", null, locale);
    SimpleMailMessage email = new SimpleMailMessage();
    email.setSubject("Resend Registration Token");
    email.setText(message + " rn" + confirmationUrl);
    email.setFrom(env.getProperty("support.email"));
    email.setTo(user.getEmail());
    return email;
}
```

我们还需要修改现有的注册功能——在模型**上添加一些关于令牌**到期的新信息:

```
@GetMapping("/registrationConfirm")
public String confirmRegistration(
  Locale locale, Model model, @RequestParam("token") String token) {
    VerificationToken verificationToken = userService.getVerificationToken(token);
    if (verificationToken == null) {
        String message = messages.getMessage("auth.message.invalidToken", null, locale);
        model.addAttribute("message", message);
        return "redirect:/badUser.html?lang=" + locale.getLanguage();
    }

    User user = verificationToken.getUser();
    Calendar cal = Calendar.getInstance();
    if ((verificationToken.getExpiryDate().getTime() - cal.getTime().getTime()) <= 0) {
        model.addAttribute("message", messages.getMessage("auth.message.expired", null, locale));
        model.addAttribute("expired", true);
        model.addAttribute("token", token);
        return "redirect:/badUser.html?lang=" + locale.getLanguage();
    }

    user.setEnabled(true);
    userService.saveRegisteredUser(user);
    model.addAttribute("message", messages.getMessage("message.accountVerified", null, locale));
    return "redirect:/login.html?lang=" + locale.getLanguage();
}
```

## 3。异常处理程序

在某些情况下，前面的功能是抛出异常；这些异常需要被处理，我们将用定制的异常处理程序来处理这些异常:

```
@ControllerAdvice
public class RestResponseEntityExceptionHandler extends ResponseEntityExceptionHandler {

    @Autowired
    private MessageSource messages;

    @ExceptionHandler({ UserNotFoundException.class })
    public ResponseEntity<Object> handleUserNotFound(RuntimeException ex, WebRequest request) {
        logger.error("404 Status Code", ex);
        GenericResponse bodyOfResponse = new GenericResponse(
          messages.getMessage("message.userNotFound", null, request.getLocale()), "UserNotFound");

        return handleExceptionInternal(
          ex, bodyOfResponse, new HttpHeaders(), HttpStatus.NOT_FOUND, request);
    }

    @ExceptionHandler({ MailAuthenticationException.class })
    public ResponseEntity<Object> handleMail(RuntimeException ex, WebRequest request) {
        logger.error("500 Status Code", ex);
        GenericResponse bodyOfResponse = new GenericResponse(
          messages.getMessage(
            "message.email.config.error", null, request.getLocale()), "MailError");

        return handleExceptionInternal(
          ex, bodyOfResponse, new HttpHeaders(), HttpStatus.NOT_FOUND, request);
    }

    @ExceptionHandler({ Exception.class })
    public ResponseEntity<Object> handleInternal(RuntimeException ex, WebRequest request) {
        logger.error("500 Status Code", ex);
        GenericResponse bodyOfResponse = new GenericResponse(
          messages.getMessage(
            "message.error", null, request.getLocale()), "InternalError");

        return handleExceptionInternal(
          ex, bodyOfResponse, new HttpHeaders(), HttpStatus.NOT_FOUND, request);
    }
}
```

请注意:

*   我们使用`@ControllerAdvice`注释来处理整个应用程序中的异常
*   我们使用一个简单的对象`GenericResponse`来发送响应:

```
public class GenericResponse {
    private String message;
    private String error;

    public GenericResponse(String message) {
        super();
        this.message = message;
    }

    public GenericResponse(String message, String error) {
        super();
        this.message = message;
        this.error = error;
    }
}
```

## 4。修改`badUser.html`

我们现在将修改`badUser.html`,使用户仅在其令牌过期时才能获得新的`VerificationToken`:

```
<html>
<head>
<title th:text="#{label.badUser.title}">bad user</title>
</head>
<body>
<h1 th:text="${param.message[0]}">error</h1>
<br>
<a th:href="@{/user/registration}" th:text="#{label.form.loginSignUp}">
  signup</a>

<div th:if="${param.expired[0]}">
<h1 th:text="#{label.form.resendRegistrationToken}">resend</h1>
<button onclick="resendToken()" 
  th:text="#{label.form.resendRegistrationToken}">resend</button>

<script src="jquery.min.js"></script>
<script type="text/javascript">

var serverContext = [[@{/}]];

function resendToken(){
    $.get(serverContext + "user/resendRegistrationToken?token=" + token, 
      function(data){
            window.location.href = 
              serverContext +"login.html?message=" + data.message;
    })
    .fail(function(data) {
        if(data.responseJSON.error.indexOf("MailError") > -1) {
            window.location.href = serverContext + "emailError.html";
        }
        else {
            window.location.href = 
              serverContext + "login.html?message=" + data.responseJSON.message;
        }
    });
}
</script>
</div>
</body>
</html>
```

注意，我们在这里使用了一些非常基本的 javascript 和 JQuery 来处理“/user/resendRegistrationToken”的响应，并基于它重定向用户。

## 5。结论

在这篇短文中，我们允许用户**请求一个新的验证链接来激活他们的帐户**，以防旧帐户过期。

本教程的**完整实现**可以在[的 github 项目](https://web.archive.org/web/20220529025330/https://github.com/Baeldung/spring-security-registration "The Full Registration/Authentication Example Project on Github ")中找到——这是一个基于 Eclipse 的项目，所以它应该很容易导入和运行。

Next **»**[Registration with Spring Security – Password Encoding](/web/20220529025330/https://www.baeldung.com/spring-security-registration-password-encoding-bcrypt)**«** Previous[Registration – Activate a New Account by Email](/web/20220529025330/https://www.baeldung.com/registration-verify-user-by-email)