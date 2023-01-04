# 具有 Spring 安全性的双因素认证

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-security-two-factor-authentication-with-soft-token>

## 1。概述

在本教程中，我们将使用软令牌和 Spring 安全实现[双因素认证功能](https://web.archive.org/web/20220822105735/https://en.wikipedia.org/wiki/Multi-factor_authentication)。

我们将在现有的简单登录流程中添加新功能[，并使用](https://web.archive.org/web/20220822105735/https://github.com/Baeldung/spring-security-registration)[谷歌认证应用](https://web.archive.org/web/20220822105735/https://play.google.com/store/apps/details?id=com.google.android.apps.authenticator2&hl=en)来生成令牌。

简而言之，双因素身份验证是一个验证过程，它遵循众所周知的“用户知道一些东西，用户拥有一些东西”的原则。

因此，用户在身份验证过程中提供了一个额外的“验证令牌”——基于基于时间的一次性密码 [TOTP](https://web.archive.org/web/20220822105735/https://tools.ietf.org/html/rfc6238) 算法的一次性密码验证码。

## 2。Maven 配置

首先，为了在我们的应用中使用 Google Authenticator，我们需要:

*   生成密钥
*   通过 QR 码向用户提供密钥
*   使用此密钥验证用户输入的令牌。

我们将使用一个简单的服务器端[库](https://web.archive.org/web/20220822105735/https://github.com/aerogear/aerogear-otp-java)，通过向我们的`pom.xml`添加以下依赖项来生成/验证一次性密码:

```java
<dependency>
    <groupId>org.jboss.aerogear</groupId>
    <artifactId>aerogear-otp-java</artifactId>
    <version>1.0.0</version>
</dependency>
```

## 3。用户实体

接下来，我们将修改用户实体以保存额外信息，如下所示:

```java
@Entity
public class User {
    ...
    private boolean isUsing2FA;
    private String secret;

    public User() {
        super();
        this.secret = Base32.random();
        ...
    }
}
```

请注意:

*   我们为每个用户保存了一个随机密码，供以后生成验证码时使用
*   我们的两步验证是可选的

## 4。额外登录参数

首先，我们需要调整我们的安全配置来接受额外的参数——验证令牌。我们可以通过使用 custom `AuthenticationDetailsSource`来实现这一点:

这是我们的`CustomWebAuthenticationDetailsSource`:

```java
@Component
public class CustomWebAuthenticationDetailsSource implements 
  AuthenticationDetailsSource<HttpServletRequest, WebAuthenticationDetails> {

    @Override
    public WebAuthenticationDetails buildDetails(HttpServletRequest context) {
        return new CustomWebAuthenticationDetails(context);
    }
}
```

这里是`CustomWebAuthenticationDetails`:

```java
public class CustomWebAuthenticationDetails extends WebAuthenticationDetails {

    private String verificationCode;

    public CustomWebAuthenticationDetails(HttpServletRequest request) {
        super(request);
        verificationCode = request.getParameter("code");
    }

    public String getVerificationCode() {
        return verificationCode;
    }
}
```

我们的安全配置:

```java
@Configuration
@EnableWebSecurity
public class LssSecurityConfig extends WebSecurityConfigurerAdapter {

    @Autowired
    private CustomWebAuthenticationDetailsSource authenticationDetailsSource;

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.formLogin()
            .authenticationDetailsSource(authenticationDetailsSource)
            ...
    } 
}
```

最后，向我们的登录表单添加额外的参数:

```java
<labelth:text="#{label.form.login2fa}">
    Google Authenticator Verification Code
</label>
<input type='text' name='code'/>
```

注意:我们需要在我们的安全配置中设置我们的自定义`AuthenticationDetailsSource`。

## 5。自定义认证提供商

接下来，我们需要一个定制的`AuthenticationProvider`来处理额外的参数验证:

```java
public class CustomAuthenticationProvider extends DaoAuthenticationProvider {

    @Autowired
    private UserRepository userRepository;

    @Override
    public Authentication authenticate(Authentication auth)
      throws AuthenticationException {
        String verificationCode 
          = ((CustomWebAuthenticationDetails) auth.getDetails())
            .getVerificationCode();
        User user = userRepository.findByEmail(auth.getName());
        if ((user == null)) {
            throw new BadCredentialsException("Invalid username or password");
        }
        if (user.isUsing2FA()) {
            Totp totp = new Totp(user.getSecret());
            if (!isValidLong(verificationCode) || !totp.verify(verificationCode)) {
                throw new BadCredentialsException("Invalid verfication code");
            }
        }

        Authentication result = super.authenticate(auth);
        return new UsernamePasswordAuthenticationToken(
          user, result.getCredentials(), result.getAuthorities());
    }

    private boolean isValidLong(String code) {
        try {
            Long.parseLong(code);
        } catch (NumberFormatException e) {
            return false;
        }
        return true;
    }

    @Override
    public boolean supports(Class<?> authentication) {
        return authentication.equals(UsernamePasswordAuthenticationToken.class);
    }
}
```

请注意，在我们验证了一次性密码验证码之后，我们只是将身份验证委托给了下游。

这是我们的身份验证提供者 bean

```java
@Bean
public DaoAuthenticationProvider authProvider() {
    CustomAuthenticationProvider authProvider = new CustomAuthenticationProvider();
    authProvider.setUserDetailsService(userDetailsService);
    authProvider.setPasswordEncoder(encoder());
    return authProvider;
}
```

## 6。注册流程

现在，为了让用户能够使用应用程序生成令牌，他们需要在注册时进行适当的设置。

因此，我们需要对注册流程做一些简单的修改——允许选择使用两步验证的用户**扫描他们稍后需要登录的二维码**。

首先，我们将这个简单的输入添加到注册表中:

```java
Use Two step verification <input type="checkbox" name="using2FA" value="true"/>
```

然后，在我们的`RegistrationController`中，我们根据用户确认注册后的选择重定向用户:

```java
@GetMapping("/registrationConfirm")
public String confirmRegistration(@RequestParam("token") String token, ...) {
    String result = userService.validateVerificationToken(token);
    if(result.equals("valid")) {
        User user = userService.getUser(token);
        if (user.isUsing2FA()) {
            model.addAttribute("qr", userService.generateQRUrl(user));
            return "redirect:/qrcode.html?lang=" + locale.getLanguage();
        }

        model.addAttribute(
          "message", messages.getMessage("message.accountVerified", null, locale));
        return "redirect:/login?lang=" + locale.getLanguage();
    }
    ...
}
```

这是我们的方法:

```java
public static String QR_PREFIX = 
  "https://chart.googleapis.com/chart?chs=200x200&chld;=M%%7C0&cht;=qr&chl;=";

@Override
public String generateQRUrl(User user) {
    return QR_PREFIX + URLEncoder.encode(String.format(
      "otpauth://totp/%s:%s?secret=%s&issuer;=%s", 
      APP_NAME, user.getEmail(), user.getSecret(), APP_NAME),
      "UTF-8");
}
```

这是我们的`qrcode.html`:

```java
<html>
<body>
<div id="qr">
    <p>
        Scan this Barcode using Google Authenticator app on your phone 
        to use it later in login
    </p>
    <img th:src="${param.qr[0]}"/>
</div>
<a href="/login" class="btn btn-primary">Go to login page</a>
</body>
</html>
```

请注意:

*   `generateQRUrl()`用于生成二维码网址的方法
*   这个二维码将被使用谷歌认证应用的用户手机扫描
*   该应用程序将生成一个 6 位数字的代码，有效期仅为 30 秒，这是理想的验证码
*   此验证码将在登录时使用我们的自定义`AuthenticationProvider`进行验证

## 7。启用两步验证

接下来，我们将确保用户可以随时更改他们的登录首选项，如下所示:

```java
@PostMapping("/user/update/2fa")
public GenericResponse modifyUser2FA(@RequestParam("use2FA") boolean use2FA) 
  throws UnsupportedEncodingException {
    User user = userService.updateUser2FA(use2FA);
    if (use2FA) {
        return new GenericResponse(userService.generateQRUrl(user));
    }
    return null;
}
```

这里是`updateUser2FA()`:

```java
@Override
public User updateUser2FA(boolean use2FA) {
    Authentication curAuth = SecurityContextHolder.getContext().getAuthentication();
    User currentUser = (User) curAuth.getPrincipal();
    currentUser.setUsing2FA(use2FA);
    currentUser = repository.save(currentUser);

    Authentication auth = new UsernamePasswordAuthenticationToken(
      currentUser, currentUser.getPassword(), curAuth.getAuthorities());
    SecurityContextHolder.getContext().setAuthentication(auth);
    return currentUser;
}
```

这是前端:

```java
<div th:if="${#authentication.principal.using2FA}">
    You are using Two-step authentication 
    <a href="#" onclick="disable2FA()">Disable 2FA</a> 
</div>
<div th:if="${! #authentication.principal.using2FA}">
    You are not using Two-step authentication 
    <a href="#" onclick="enable2FA()">Enable 2FA</a> 
</div>
<br/>
<div id="qr" style="display:none;">
    <p>Scan this Barcode using Google Authenticator app on your phone </p>
</div>

<script src="http://ajax.googleapis.com/ajax/libs/jquery/1.11.2/jquery.min.js"></script>
<script type="text/javascript">
function enable2FA(){
    set2FA(true);
}
function disable2FA(){
    set2FA(false);
}
function set2FA(use2FA){
    $.post( "/user/update/2fa", { use2FA: use2FA } , function( data ) {
        if(use2FA){
        	$("#qr").append('<img src="'+data.message+'" />').show();
        }else{
            window.location.reload();
        }
    });
}
</script>
```

## 8。结论

在这个快速教程中，我们展示了如何使用具有 Spring 安全性的软令牌实现双因素身份验证。

完整的源代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220822105735/https://github.com/Baeldung/spring-security-registration)