# spring Security–重置您的密码

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-security-registration-i-forgot-my-password>

[This article is part of a series:](javascript:void(0);)[• Spring Security Registration Tutorial](/web/20220625235825/https://www.baeldung.com/spring-security-registration)
[• The Registration Process With Spring Security](/web/20220625235825/https://www.baeldung.com/registration-with-spring-mvc-and-spring-security)
[• Registration – Activate a New Account by Email](/web/20220625235825/https://www.baeldung.com/registration-verify-user-by-email)
[• Spring Security Registration – Resend Verification Email](/web/20220625235825/https://www.baeldung.com/spring-security-registration-verification-email)
[• Registration with Spring Security – Password Encoding](/web/20220625235825/https://www.baeldung.com/spring-security-registration-password-encoding-bcrypt)
[• The Registration API becomes RESTful](/web/20220625235825/https://www.baeldung.com/registration-restful-api)
• Spring Security – Reset Your Password (current article)[• Registration – Password Strength and Rules](/web/20220625235825/https://www.baeldung.com/registration-password-strength-and-rules)
[• Updating your Password](/web/20220625235825/https://www.baeldung.com/updating-your-password/)

## 1。概述

在本教程中——我们继续正在进行的 **`Registration with Spring Security`系列**，看看**“T1”的基本功能**——这样用户就可以在需要时安全地重置自己的密码。

## 2.请求重置您的密码

密码重置流程通常在用户单击登录页面上的某种“重置”按钮时开始。然后，我们可以向用户询问他们的电子邮件地址或其他识别信息。确认后，我们可以生成一个令牌并向用户发送电子邮件。

下图显示了我们将在本文中实现的流程:

[![Request password reset e-mail](img/3d4fa71aa4125a62c6f0849b50f389f6.png)](/web/20220625235825/https://www.baeldung.com/wp-content/uploads/2015/02/reset1-1024x737-1.png)

## 3。密码重置令牌

让我们首先创建一个`PasswordResetToken`实体，用它来重置用户的密码:

```java
@Entity
public class PasswordResetToken {

    private static final int EXPIRATION = 60 * 24;

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;

    private String token;

    @OneToOne(targetEntity = User.class, fetch = FetchType.EAGER)
    @JoinColumn(nullable = false, name = "user_id")
    private User user;

    private Date expiryDate;
}
```

当触发密码重置时，将创建一个令牌，并且将通过电子邮件向用户发送一个包含该令牌的特殊链接**。**

令牌和链接仅在设定的时间段内有效(在本例中为 24 小时)。

## 4。`forgotPassword.html`

该过程的第一页是**“`I forgot my password`”页**——在这里，用户被提示输入他们的电子邮件地址，以便开始实际的重置过程。

因此，让我们设计一个简单的`forgotPassword.html`询问用户的电子邮件地址:

```java
<html>
<body>
    <h1 th:text="#{message.resetPassword}">reset</h1>

    <label th:text="#{label.user.email}">email</label>
    <input id="email" name="email" type="email" value="" />
    <button type="submit" onclick="resetPass()" 
      th:text="#{message.resetPassword}">reset</button>

<a th:href="@{/registration.html}" th:text="#{label.form.loginSignUp}">
    registration
</a>
<a th:href="@{/login}" th:text="#{label.form.loginLink}">login</a>

<script src="jquery.min.js"></script>
<script th:inline="javascript">
var serverContext = [[@{/}]];
function resetPass(){
    var email = $("#email").val();
    $.post(serverContext + "user/resetPassword",{email: email} ,
      function(data){
          window.location.href = 
           serverContext + "login?message=" + data.message;
    })
    .fail(function(data) {
    	if(data.responseJSON.error.indexOf("MailError") > -1)
        {
            window.location.href = serverContext + "emailError.html";
        }
        else{
            window.location.href = 
              serverContext + "login?message=" + data.responseJSON.message;
        }
    });
}

</script>
</body>

</html>
```

我们现在需要从登录页面链接到这个新的“`reset password`”页面:

```java
<a th:href="@{/forgetPassword.html}" 
  th:text="#{message.resetPassword}">reset</a>
```

## 5。创建`PasswordResetToken`

让我们从创建新的`PasswordResetToken`开始，并通过电子邮件发送给用户:

```java
@PostMapping("/user/resetPassword")
public GenericResponse resetPassword(HttpServletRequest request, 
  @RequestParam("email") String userEmail) {
    User user = userService.findUserByEmail(userEmail);
    if (user == null) {
        throw new UserNotFoundException();
    }
    String token = UUID.randomUUID().toString();
    userService.createPasswordResetTokenForUser(user, token);
    mailSender.send(constructResetTokenEmail(getAppUrl(request), 
      request.getLocale(), token, user));
    return new GenericResponse(
      messages.getMessage("message.resetPasswordEmail", null, 
      request.getLocale()));
}
```

这里是`createPasswordResetTokenForUser()`方法:

```java
public void createPasswordResetTokenForUser(User user, String token) {
    PasswordResetToken myToken = new PasswordResetToken(token, user);
    passwordTokenRepository.save(myToken);
}
```

下面是方法`constructResetTokenEmail()`–用于发送带有重置令牌的电子邮件:

```java
private SimpleMailMessage constructResetTokenEmail(
  String contextPath, Locale locale, String token, User user) {
    String url = contextPath + "/user/changePassword?token=" + token;
    String message = messages.getMessage("message.resetPassword", 
      null, locale);
    return constructEmail("Reset Password", message + " \r\n" + url, user);
}

private SimpleMailMessage constructEmail(String subject, String body, 
  User user) {
    SimpleMailMessage email = new SimpleMailMessage();
    email.setSubject(subject);
    email.setText(body);
    email.setTo(user.getEmail());
    email.setFrom(env.getProperty("support.email"));
    return email;
}
```

请注意我们如何使用一个简单的对象`GenericResponse`来表示我们对客户端的响应:

```java
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

## 6。`PasswordResetToken`勾选

一旦用户点击他们电子邮件中的链接，`user/changePassword`端点:

*   验证令牌是否有效
*   向用户显示`updatePassword`页面，用户可以在其中输入新密码

然后，新密码和令牌被传递到`user/savePassword`端点:
[![Reset Password](img/fab609ac050477685bce8e9fedb0fc38.png)](/web/20220625235825/https://www.baeldung.com/wp-content/uploads/2015/02/reset2-1024x425-1.png)

用户收到带有重置密码的唯一链接的电子邮件，然后单击该链接:

```java
@GetMapping("/user/changePassword")
public String showChangePasswordPage(Locale locale, Model model, 
  @RequestParam("token") String token) {
    String result = securityService.validatePasswordResetToken(token);
    if(result != null) {
        String message = messages.getMessage("auth.message." + result, null, locale);
        return "redirect:/login.html?lang=" 
            + locale.getLanguage() + "&message;=" + message;
    } else {
        model.addAttribute("token", token);
        return "redirect:/updatePassword.html?lang=" + locale.getLanguage();
    }
}
```

这里是`validatePasswordResetToken()`方法:

```java
public String validatePasswordResetToken(String token) {
    final PasswordResetToken passToken = passwordTokenRepository.findByToken(token);

    return !isTokenFound(passToken) ? "invalidToken"
            : isTokenExpired(passToken) ? "expired"
            : null;
}

private boolean isTokenFound(PasswordResetToken passToken) {
    return passToken != null;
}

private boolean isTokenExpired(PasswordResetToken passToken) {
    final Calendar cal = Calendar.getInstance();
    return passToken.getExpiryDate().before(cal.getTime());
}
```

## 7。更改密码

此时，用户会看到简单的`Password Reset`页面——这里唯一可能的选项是**提供一个新密码**:

### 7.1。`updatePassword.html`

```java
<html>
<body>
<div sec:authorize="hasAuthority('CHANGE_PASSWORD_PRIVILEGE')">
    <h1 th:text="#{message.resetYourPassword}">reset</h1>
    <form>
        <label th:text="#{label.user.password}">password</label>
        <input id="password" name="newPassword" type="password" value="" />

        <label th:text="#{label.user.confirmPass}">confirm</label>
        <input id="matchPassword" type="password" value="" />

        <label th:text="#{token.message}">token</label>
        <input id="token" name="token" value="" />

        <div id="globalError" style="display:none" 
          th:text="#{PasswordMatches.user}">error</div>
        <button type="submit" onclick="savePass()" 
          th:text="#{message.updatePassword}">submit</button>
    </form>

<script th:inline="javascript">
var serverContext = [[@{/}]];
$(document).ready(function () {
    $('form').submit(function(event) {
        savePass(event);
    });

    $(":password").keyup(function(){
        if($("#password").val() != $("#matchPassword").val()){
            $("#globalError").show().html(/*[[#{PasswordMatches.user}]]*/);
        }else{
            $("#globalError").html("").hide();
        }
    });
});

function savePass(event){
    event.preventDefault();
    if($("#password").val() != $("#matchPassword").val()){
        $("#globalError").show().html(/*[[#{PasswordMatches.user}]]*/);
        return;
    }
    var formData= $('form').serialize();
    $.post(serverContext + "user/savePassword",formData ,function(data){
        window.location.href = serverContext + "login?message="+data.message;
    })
    .fail(function(data) {
        if(data.responseJSON.error.indexOf("InternalError") > -1){
            window.location.href = serverContext + "login?message=" + data.responseJSON.message;
        }
        else{
            var errors = $.parseJSON(data.responseJSON.message);
            $.each( errors, function( index,item ){
                $("#globalError").show().html(item.defaultMessage);
            });
            errors = $.parseJSON(data.responseJSON.error);
            $.each( errors, function( index,item ){
                $("#globalError").show().append(item.defaultMessage+"<br/>");
            });
        }
    });
}
</script>    
</div>
</body>
</html>
```

注意，我们显示了 reset 标记，并在下面保存密码的调用中将它作为 POST 参数传递。

### 7.2。保存密码

最后，当提交上一个 post 请求时，新的用户密码被保存:

```java
@PostMapping("/user/savePassword")
public GenericResponse savePassword(final Locale locale, @Valid PasswordDto passwordDto) {

    String result = securityUserService.validatePasswordResetToken(passwordDto.getToken());

    if(result != null) {
        return new GenericResponse(messages.getMessage(
            "auth.message." + result, null, locale));
    }

    Optional user = userService.getUserByPasswordResetToken(passwordDto.getToken());
    if(user.isPresent()) {
        userService.changeUserPassword(user.get(), passwordDto.getNewPassword());
        return new GenericResponse(messages.getMessage(
            "message.resetPasswordSuc", null, locale));
    } else {
        return new GenericResponse(messages.getMessage(
            "auth.message.invalid", null, locale));
    }
}
```

这里是`changeUserPassword()`方法:

```java
public void changeUserPassword(User user, String password) {
    user.setPassword(passwordEncoder.encode(password));
    repository.save(user);
}
```

而`PasswordDto`:

```java
public class PasswordDto {

    private String oldPassword;

    private  String token;

    @ValidPassword
    private String newPassword;
} 
```

## 8。结论

在本文中，我们为成熟的身份验证过程实现了一个简单但非常有用的特性——作为系统的用户，可以选择重置自己的密码。

本教程的**完整实现**可以在[的 GitHub 项目](https://web.archive.org/web/20220625235825/https://github.com/Baeldung/spring-security-registration "The Full Registration/Authentication Example Project on Github ")中找到——这是一个基于 Eclipse 的项目，所以它应该很容易导入和运行。

Next **»**[Registration – Password Strength and Rules](/web/20220625235825/https://www.baeldung.com/registration-password-strength-and-rules)**«** Previous[The Registration API becomes RESTful](/web/20220625235825/https://www.baeldung.com/registration-restful-api)