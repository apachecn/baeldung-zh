# 向 Spring 注册——集成 reCAPTCHA

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-security-registration-captcha>

## 1。概述

在本教程中，我们将继续[春季安全注册](/web/20220917055838/https://www.baeldung.com/spring-security-registration)系列，在注册过程中添加**谷歌**[reCAPTCHA](/web/20220917055838/https://www.baeldung.com/cs/captcha-intro)，以区分人类和机器人。

## 2。整合谷歌的 reCAPTCHA

为了集成 Google 的 reCAPTCHA web 服务，我们首先需要向该服务注册我们的站点，将他们的库添加到我们的页面，然后用 web 服务验证用户的 CAPTCHA 响应。

让我们在 https://www.google.com/recaptcha/admin 注册我们的网站。注册过程产生用于访问网络服务的**站点密钥**和**秘密密钥**。

### 2.1。存储 API 密钥对

我们将密钥存储在`application.properties:`中

```java
google.recaptcha.key.site=6LfaHiITAAAA...
google.recaptcha.key.secret=6LfaHiITAAAA...
```

并使用标注了`@ConfigurationProperties:`的 bean 将它们暴露给 Spring

```java
@Component
@ConfigurationProperties(prefix = "google.recaptcha.key")
public class CaptchaSettings {

    private String site;
    private String secret;

    // standard getters and setters
}
```

### 2.2。显示小工具

在系列教程的基础上，我们现在将修改`registration.html`来包含 Google 的库。

在我们的注册表单中，我们添加了 reCAPTCHA 小部件，它期望属性`data-sitekey`包含**站点关键字**。

当提交时，小部件将追加**请求参数`g-recaptcha-response`:**

```java
<!DOCTYPE html>
<html>
<head>

...

<script src='https://www.google.com/recaptcha/api.js'></script>
</head>
<body>

    ...

    <form action="/" method="POST" enctype="utf8">
        ...

        <div class="g-recaptcha col-sm-5"
          th:attr="data-sitekey=${@captchaSettings.getSite()}"></div>
        <span id="captchaError" class="alert alert-danger col-sm-4"
          style="display:none"></span>
```

## 3。服务器端验证

新的请求参数对我们的站点密钥和唯一的字符串进行编码，该字符串标识用户成功完成了挑战。

然而，由于我们自己不能辨别，我们不能相信用户提交的是合法的。服务器端请求用 web 服务 API 验证`captcha response`。

端点接受 URL[https://www.google.com/recaptcha/api/siteverify](https://web.archive.org/web/20220917055838/https://www.google.com/recaptcha/api/siteverify)上的 HTTP 请求，带有查询参数`**secret**`、`**response**`和`**remoteip.**` ，它返回一个 json 响应，其模式为:

```java
{
    "success": true|false,
    "challenge_ts": timestamp,
    "hostname": string,
    "error-codes": [ ... ]
}
```

### 3.1。检索用户的响应

使用`HttpServletRequest`从请求参数`g-recaptcha-response`中检索用户对 reCAPTCHA 质询的响应，并用我们的`CaptchaService`进行验证。处理响应时引发的任何异常都将中止注册逻辑的其余部分:

```java
public class RegistrationController {

    @Autowired
    private ICaptchaService captchaService;

    ...

    @RequestMapping(value = "/user/registration", method = RequestMethod.POST)
    @ResponseBody
    public GenericResponse registerUserAccount(@Valid UserDto accountDto, HttpServletRequest request) {
        String response = request.getParameter("g-recaptcha-response");
        captchaService.processResponse(response);

        // Rest of implementation
    }

    ...
}
```

### 3.2。验证服务

应首先清理获得的验证码响应。使用一个简单的正则表达式。

如果响应看起来合法，我们就用**密钥**、**验证码响应**和客户端的 **IP 地址**向 web 服务发出请求:

```java
public class CaptchaService implements ICaptchaService {

    @Autowired
    private CaptchaSettings captchaSettings;

    @Autowired
    private RestOperations restTemplate;

    private static Pattern RESPONSE_PATTERN = Pattern.compile("[A-Za-z0-9_-]+");

    @Override
    public void processResponse(String response) {
        if(!responseSanityCheck(response)) {
            throw new InvalidReCaptchaException("Response contains invalid characters");
        }

        URI verifyUri = URI.create(String.format(
          "https://www.google.com/recaptcha/api/siteverify?secret=%s&response;=%s&remoteip;=%s",
          getReCaptchaSecret(), response, getClientIP()));

        GoogleResponse googleResponse = restTemplate.getForObject(verifyUri, GoogleResponse.class);

        if(!googleResponse.isSuccess()) {
            throw new ReCaptchaInvalidException("reCaptcha was not successfully validated");
        }
    }

    private boolean responseSanityCheck(String response) {
        return StringUtils.hasLength(response) && RESPONSE_PATTERN.matcher(response).matches();
    }
}
```

### 3.3。具体化验证

用`Jackson`注释修饰的 Java-bean 封装了验证响应:

```java
@JsonInclude(JsonInclude.Include.NON_NULL)
@JsonIgnoreProperties(ignoreUnknown = true)
@JsonPropertyOrder({
    "success",
    "challenge_ts",
    "hostname",
    "error-codes"
})
public class GoogleResponse {

    @JsonProperty("success")
    private boolean success;

    @JsonProperty("challenge_ts")
    private String challengeTs;

    @JsonProperty("hostname")
    private String hostname;

    @JsonProperty("error-codes")
    private ErrorCode[] errorCodes;

    @JsonIgnore
    public boolean hasClientError() {
        ErrorCode[] errors = getErrorCodes();
        if(errors == null) {
            return false;
        }
        for(ErrorCode error : errors) {
            switch(error) {
                case InvalidResponse:
                case MissingResponse:
                    return true;
            }
        }
        return false;
    }

    static enum ErrorCode {
        MissingSecret,     InvalidSecret,
        MissingResponse,   InvalidResponse;

        private static Map<String, ErrorCode> errorsMap = new HashMap<String, ErrorCode>(4);

        static {
            errorsMap.put("missing-input-secret",   MissingSecret);
            errorsMap.put("invalid-input-secret",   InvalidSecret);
            errorsMap.put("missing-input-response", MissingResponse);
            errorsMap.put("invalid-input-response", InvalidResponse);
        }

        @JsonCreator
        public static ErrorCode forValue(String value) {
            return errorsMap.get(value.toLowerCase());
        }
    }

    // standard getters and setters
}
```

顾名思义，`success`属性中的 true 值意味着用户已经过验证。否则,`errorCodes`属性将填充原因。

`hostname`指的是将用户重定向到 reCAPTCHA 的服务器。如果您管理许多域并希望它们共享同一个密钥对，您可以选择自己验证`hostname`属性。

### 3.4。验证失败

如果验证失败，将引发异常。reCAPTCHA 库需要指示客户端创建一个新的挑战。

我们在客户端的注册错误处理程序中这样做，通过调用库的`grecaptcha`小部件上的 reset:

```java
register(event){
    event.preventDefault();

    var formData= $('form').serialize();
    $.post(serverContext + "user/registration", formData, function(data){
        if(data.message == "success") {
            // success handler
        }
    })
    .fail(function(data) {
        grecaptcha.reset();
        ...

        if(data.responseJSON.error == "InvalidReCaptcha"){ 
            $("#captchaError").show().html(data.responseJSON.message);
        }
        ...
    }
}
```

## 4。保护服务器资源

恶意客户端不需要遵守浏览器沙箱的规则。因此，我们的安全思维应该关注暴露的资源以及它们可能被滥用的方式。

### 4.1。尝试缓存

重要的是要理解，通过集成 reCAPTCHA，每个请求都将导致服务器创建一个套接字来验证请求。

虽然我们需要一个更加分层的方法来真正缓解 DoS，但我们可以实现一个基本缓存，将客户端限制为 4 个失败的验证码响应:

```java
public class ReCaptchaAttemptService {
    private int MAX_ATTEMPT = 4;
    private LoadingCache<String, Integer> attemptsCache;

    public ReCaptchaAttemptService() {
        super();
        attemptsCache = CacheBuilder.newBuilder()
          .expireAfterWrite(4, TimeUnit.HOURS).build(new CacheLoader<String, Integer>() {
            @Override
            public Integer load(String key) {
                return 0;
            }
        });
    }

    public void reCaptchaSucceeded(String key) {
        attemptsCache.invalidate(key);
    }

    public void reCaptchaFailed(String key) {
        int attempts = attemptsCache.getUnchecked(key);
        attempts++;
        attemptsCache.put(key, attempts);
    }

    public boolean isBlocked(String key) {
        return attemptsCache.getUnchecked(key) >= MAX_ATTEMPT;
    }
}
```

### 4.2。重构验证服务

如果客户端已经超过尝试限制，则首先通过中止来合并缓存。否则，当处理一个不成功的`GoogleResponse`时，我们记录包含客户端响应错误的尝试。成功的验证会清除尝试缓存:

```java
public class CaptchaService implements ICaptchaService {

    @Autowired
    private ReCaptchaAttemptService reCaptchaAttemptService;

    ...

    @Override
    public void processResponse(String response) {

        ...

        if(reCaptchaAttemptService.isBlocked(getClientIP())) {
            throw new InvalidReCaptchaException("Client exceeded maximum number of failed attempts");
        }

        ...

        GoogleResponse googleResponse = ...

        if(!googleResponse.isSuccess()) {
            if(googleResponse.hasClientError()) {
                reCaptchaAttemptService.reCaptchaFailed(getClientIP());
            }
            throw new ReCaptchaInvalidException("reCaptcha was not successfully validated");
        }
        reCaptchaAttemptService.reCaptchaSucceeded(getClientIP());
    }
}
```

## 5。整合谷歌的 reCAPTCHA v3

谷歌的 reCAPTCHA v3 与之前的版本不同，因为它不需要任何用户交互。它只是为我们发送的每个请求给出一个分数，并让我们决定对我们的 web 应用程序采取什么样的最终行动。

同样，要集成 Google 的 reCAPTCHA 3，我们首先需要向服务注册我们的站点，将他们的库添加到我们的页面，然后用 web 服务验证令牌响应。

因此，让我们在`[https://www.google.com/recaptcha/admin/create](https://web.archive.org/web/20220917055838/https://www.google.com/recaptcha/admin/create)` 注册我们的站点，在选择 reCAPTCHA v3 之后，我们将获得新的秘密和站点密钥。

### 5.1。更新 `application.properties`和`CaptchaSettings`****

注册后，我们需要用新的密钥和我们选择的分数阈值更新`application.properties` :

```java
google.recaptcha.key.site=6LefKOAUAAAAAE...
google.recaptcha.key.secret=6LefKOAUAAAA...
google.recaptcha.key.threshold=0.5
```

值得注意的是，设置为`0.5`的阈值是默认值，可以通过分析[谷歌管理控制台](https://web.archive.org/web/20220917055838/https://www.google.com/recaptcha/admin)中的实际阈值来随时调整。

接下来，让我们更新我们的`CaptchaSettings`类:

```java
@Component
@ConfigurationProperties(prefix = "google.recaptcha.key")
public class CaptchaSettings {
    // ... other properties
    private float threshold;

    // standard getters and setters
}
```

### 5.2。前端集成

我们现在将修改`registration.html`来包含 Google 的图书馆和我们的站点密钥。

在我们的注册表单中，我们添加了一个隐藏字段，用于存储从对`grecaptcha.execute`函数的调用中收到的响应令牌:

```java
<!DOCTYPE html>
<html>
<head>

...

<script th:src='|https://www.google.com/recaptcha/api.js?render=${@captchaService.getReCaptchaSite()}'></script>
</head>
<body>

    ...

    <form action="/" method="POST" enctype="utf8">
        ...

        <input type="hidden" id="response" name="response" value="" />
        ...
    </form>

   ...

<script th:inline="javascript">
   ...
   var siteKey = /*[[${@captchaService.getReCaptchaSite()}]]*/;
   grecaptcha.execute(siteKey, {action: /*[[${T(com.baeldung.captcha.CaptchaService).REGISTER_ACTION}]]*/}).then(function(response) {
	$('#response').val(response);    
    var formData= $('form').serialize();
```

### 5.3。服务器端验证

我们将不得不发出与 [reCAPTCHA 服务器端验证](/web/20220917055838/https://www.baeldung.com/spring-security-registration-captcha#Server)中相同的服务器端请求，用 web 服务 API 验证响应令牌。

响应 JSON 对象将包含两个附加属性:

```java
{
    ...
    "score": number,
    "action": string
}
```

该分数基于用户的交互，并且是介于 0(很可能是机器人)和 1.0(很可能是人类)之间的值。

Action 是 Google 引入的一个新概念，这样我们就可以在同一个网页上执行许多 reCAPTCHA 请求。

每次执行 reCAPTCHA v3 时，都必须指定一个动作。而且，我们必须验证响应中的`action`属性的值对应于预期的名称。

### 5.4。检索响应令牌

使用`HttpServletRequest`从`response`请求参数中检索 reCAPTCHA v3 响应令牌，并用我们的`CaptchaService`进行验证。该机制与 reCAPTCHA 中上方[所示的机制相同:](/web/20220917055838/https://www.baeldung.com/spring-security-registration-captcha#Retrieve)

```java
public class RegistrationController {

    @Autowired
    private ICaptchaService captchaService;

    ...

    @RequestMapping(value = "/user/registration", method = RequestMethod.POST)
    @ResponseBody
    public GenericResponse registerUserAccount(@Valid UserDto accountDto, HttpServletRequest request) {
        String response = request.getParameter("response");
        captchaService.processResponse(response, CaptchaService.REGISTER_ACTION);

        // rest of implementation
    }

    ...
}
```

### 5.5。用 v3 重构验证服务

重构后的`CaptchaService`验证服务类包含一个`processResponse`方法，类似于先前版本的 [`processResponse`方法](/web/20220917055838/https://www.baeldung.com/spring-security-registration-captcha#Validation)，但是它会仔细检查`GoogleResponse`的`action`和`score`参数:

```java
public class CaptchaService implements ICaptchaService {

    public static final String REGISTER_ACTION = "register";
    ...

    @Override
    public void processResponse(String response, String action) {
        ...

        GoogleResponse googleResponse = restTemplate.getForObject(verifyUri, GoogleResponse.class);        
        if(!googleResponse.isSuccess() || !googleResponse.getAction().equals(action) 
            || googleResponse.getScore() < captchaSettings.getThreshold()) {
            ...
            throw new ReCaptchaInvalidException("reCaptcha was not successfully validated");
        }
        reCaptchaAttemptService.reCaptchaSucceeded(getClientIP());
    }
}
```

如果验证失败，我们将抛出一个异常，但是注意在 v3 中，JavaScript 客户机中没有可以调用的`reset`方法。

我们仍将使用上面中的[来保护服务器资源。](/web/20220917055838/https://www.baeldung.com/spring-security-registration-captcha#Protecting)

### 5.6。更新`GoogleResponse`类

我们需要将新的属性`score`和`action`添加到 [`GoogleResponse`](/web/20220917055838/https://www.baeldung.com/spring-security-registration-captcha#Objectifying) Java bean 中:

```java
@JsonPropertyOrder({
    "success",
    "score", 
    "action",
    "challenge_ts",
    "hostname",
    "error-codes"
})
public class GoogleResponse {
    // ... other properties
    @JsonProperty("score")
    private float score;
    @JsonProperty("action")
    private String action;

    // standard getters and setters
}
```

## 6。结论

在本文中，我们将 Google 的 reCAPTCHA 库集成到我们的注册页面中，并实现了一个服务来验证服务器端请求的 CAPTCHA 响应。

后来，我们用 Google 的 reCAPTCHA v3 库升级了注册页面，看到注册表单变得更精简了，因为用户不再需要采取任何行动。

本教程的完整实现可在 GitHub 上的[处获得。](https://web.archive.org/web/20220917055838/https://github.com/Baeldung/spring-security-registration/tree/master)