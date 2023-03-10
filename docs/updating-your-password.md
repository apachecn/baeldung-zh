# 更新您的密码

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/updating-your-password>

[This article is part of a series:](javascript:void(0);)[• Spring Security Registration Tutorial](/web/20221006082221/https://www.baeldung.com/spring-security-registration)
[• The Registration Process With Spring Security](/web/20221006082221/https://www.baeldung.com/registration-with-spring-mvc-and-spring-security)
[• Registration – Activate a New Account by Email](/web/20221006082221/https://www.baeldung.com/registration-verify-user-by-email)
[• Spring Security Registration – Resend Verification Email](/web/20221006082221/https://www.baeldung.com/spring-security-registration-verification-email)
[• Registration with Spring Security – Password Encoding](/web/20221006082221/https://www.baeldung.com/spring-security-registration-password-encoding-bcrypt)
[• The Registration API becomes RESTful](/web/20221006082221/https://www.baeldung.com/registration-restful-api)
[• Spring Security – Reset Your Password](/web/20221006082221/https://www.baeldung.com/spring-security-registration-i-forgot-my-password)
[• Registration – Password Strength and Rules](/web/20221006082221/https://www.baeldung.com/registration-password-strength-and-rules)
• Updating your Password (current article)

## 1。概述

在这篇简短的文章中，我们将实现一个简单的“更改我自己的密码”功能，供用户在注册和登录后使用。

## 2。客户端–更改我的密码页面

让我们来看看这个非常简单的客户端页面:

```java
<html>
<body>
<div id="errormsg" style="display:none"></div>
<div>
    <input id="oldpass" name="oldpassword" type="password" />
    <input id="pass" name="password" type="password" />
    <input id="passConfirm" type="password" />              
    <span id="error" style="display:none">Password mismatch</span>

   <button type="submit" onclick="savePass()">Change Password</button>
</div>

<script src="jquery.min.js"></script>
<script type="text/javascript">

var serverContext = [[@{/}]];
function savePass(){
    var pass = $("#pass").val();
    var valid = pass == $("#passConfirm").val();
    if(!valid) {
      $("#error").show();
      return;
    }
    $.post(serverContext + "user/updatePassword",
      {password: pass, oldpassword: $("#oldpass").val()} ,function(data){
        window.location.href = serverContext +"/home.html?message="+data.message;
    })
    .fail(function(data) {
        $("#errormsg").show().html(data.responseJSON.message);
    });
}
</script> 
</body>
</html>
```

## 3。更新用户密码

现在让我们也实现服务器端操作:

```java
@PostMapping("/user/updatePassword")
@PreAuthorize("hasRole('READ_PRIVILEGE')")
public GenericResponse changeUserPassword(Locale locale, 
  @RequestParam("password") String password, 
  @RequestParam("oldpassword") String oldPassword) {
    User user = userService.findUserByEmail(
      SecurityContextHolder.getContext().getAuthentication().getName());

    if (!userService.checkIfValidOldPassword(user, oldPassword)) {
        throw new InvalidOldPasswordException();
    }
    userService.changeUserPassword(user, password);
    return new GenericResponse(messages.getMessage("message.updatePasswordSuc", null, locale));
}
```

注意这个方法是如何通过`@PreAuthorize`注释得到保护的，因为它应该**只对登录的用户**可用。

## 4。API 测试

最后，让我们用一些 API 测试来消费 API，以确保一切都工作正常；我们将从测试的简单配置和数据初始化开始:

```java
@ExtendWith(SpringExtension.class)
@ContextConfiguration(
  classes = { ConfigTest.class, PersistenceJPAConfig.class }, 
  loader = AnnotationConfigContextLoader.class)
public class ChangePasswordApiTest {
    private final String URL_PREFIX = "http://localhost:8080/"; 
    private final String URL = URL_PREFIX + "/user/updatePassword";

    @Autowired
    private UserRepository userRepository;

    @Autowired
    private PasswordEncoder passwordEncoder;

    FormAuthConfig formConfig = new FormAuthConfig(
      URL_PREFIX + "/login", "username", "password");

    @BeforeEach
    public void init() {
        User user = userRepository.findByEmail("[[email protected]](/web/20221006082221/https://www.baeldung.com/cdn-cgi/l/email-protection)");
        if (user == null) {
            user = new User();
            user.setFirstName("Test");
            user.setLastName("Test");
            user.setPassword(passwordEncoder.encode("test"));
            user.setEmail("[[email protected]](/web/20221006082221/https://www.baeldung.com/cdn-cgi/l/email-protection)");
            user.setEnabled(true);
            userRepository.save(user);
        } else {
            user.setPassword(passwordEncoder.encode("test"));
            userRepository.save(user);
        }
    }
}
```

现在，让我们试着**为登录用户**更改密码:

```java
@Test
public void givenLoggedInUser_whenChangingPassword_thenCorrect() {
    RequestSpecification request = RestAssured.given().auth()
      .form("[[email protected]](/web/20221006082221/https://www.baeldung.com/cdn-cgi/l/email-protection)", "test", formConfig);

    Map<String, String> params = new HashMap<String, String>();
    params.put("oldpassword", "test");
    params.put("password", "newtest");

    Response response = request.with().params(params).post(URL);

    assertEquals(200, response.statusCode());
    assertTrue(response.body().asString().contains("Password updated successfully"));
}
```

接下来——假设旧密码错误，让我们尝试更改密码**:**

```java
@Test
public void givenWrongOldPassword_whenChangingPassword_thenBadRequest() {
    RequestSpecification request = RestAssured.given().auth()
      .form("[[email protected]](/web/20221006082221/https://www.baeldung.com/cdn-cgi/l/email-protection)", "test", formConfig);

    Map<String, String> params = new HashMap<String, String>();
    params.put("oldpassword", "abc");
    params.put("password", "newtest");

    Response response = request.with().params(params).post(URL);

    assertEquals(400, response.statusCode());
    assertTrue(response.body().asString().contains("Invalid Old Password"));
}
```

最后，让我们尝试在没有身份验证的情况下更改密码**:**

```java
@Test
public void givenNotAuthenticatedUser_whenChangingPassword_thenRedirect() {
    Map<String, String> params = new HashMap<String, String>();
    params.put("oldpassword", "abc");
    params.put("password", "xyz");

    Response response = RestAssured.with().params(params).post(URL);

    assertEquals(302, response.statusCode());
    assertFalse(response.body().asString().contains("Password updated successfully"));
}
```

注意对于每个测试，我们如何提供一个`FormAuthConfig`来处理认证。

我们还通过`init()`重置了密码，以确保我们在测试前使用正确的密码。

## 5。结论

这是一种简单的方法，允许用户在注册和登录应用程序后更改自己的密码。

本教程的**完整实现**可以在[的 github 项目](https://web.archive.org/web/20221006082221/https://github.com/Baeldung/spring-security-registration "The Full Registration/Authentication Example Project on Github ")中找到——这是一个基于 Eclipse 的项目，所以它应该很容易导入和运行。

**«** Previous[Registration – Password Strength and Rules](/web/20221006082221/https://www.baeldung.com/registration-password-strength-and-rules)