# 注册 API 变得 RESTful

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/registration-restful-api>

 ![](img/10d9864f35e35b3e2b39703039cdf38d.png)

It’s just plain hard to get true, **real-time visibility into a running auth flow.**

Parts of the process can be completely hidden from us; if the complete authorization process requires a redirect from a remote OAuth production server, then every debugging effort must go through the production server.

It’s practically unfeasible to debug this locally. There’s no way to reproduce the exact state and no way to inspect what is actually happening under the hood. Not ideal.

Knowing these types of challenges, we built Lightrun - a real-time production debugging tool - to allow you to understand complicated flows with code-level information. Add logs, take snapshots (virtual breakpoints), and instrument metrics without a remote debugger, without stopping the running service, and, most importantly - **in real-time and without side effects**.

**Learn more with this 5-minute tutorial** focused on debugging these kinds of scenarios using Lightrun:

[>> Debugging Authentication and Authorization Using Lightrun](/web/20220523145855/https://www.baeldung.com/lightrun-n-security)

[This article is part of a series:](javascript:void(0);)[• Spring Security Registration Tutorial](/web/20220523145855/https://www.baeldung.com/spring-security-registration)
[• The Registration Process With Spring Security](/web/20220523145855/https://www.baeldung.com/registration-with-spring-mvc-and-spring-security)
[• Registration – Activate a New Account by Email](/web/20220523145855/https://www.baeldung.com/registration-verify-user-by-email)
[• Spring Security Registration – Resend Verification Email](/web/20220523145855/https://www.baeldung.com/spring-security-registration-verification-email)
[• Registration with Spring Security – Password Encoding](/web/20220523145855/https://www.baeldung.com/spring-security-registration-password-encoding-bcrypt)
• The Registration API becomes RESTful (current article)[• Spring Security – Reset Your Password](/web/20220523145855/https://www.baeldung.com/spring-security-registration-i-forgot-my-password)
[• Registration – Password Strength and Rules](/web/20220523145855/https://www.baeldung.com/registration-password-strength-and-rules)
[• Updating your Password](/web/20220523145855/https://www.baeldung.com/updating-your-password/)

## 1。概述

在 Baeldung 注册系列的最后几篇文章中，我们以 MVC 的方式构建了我们需要的大部分功能。

我们现在将把其中一些 API 转换成一种更 RESTful 的方法。

## 2。 `Register` 行动

让我们从主寄存器操作开始:

```java
@PostMapping("/user/registration")
public GenericResponse registerUserAccount(
      @Valid UserDto accountDto, HttpServletRequest request) {
    logger.debug("Registering user account with information: {}", accountDto);
    User registered = createUserAccount(accountDto);
    if (registered == null) {
        throw new UserAlreadyExistException();
    }
    String appUrl = "http://" + request.getServerName() + ":" + 
      request.getServerPort() + request.getContextPath();

    eventPublisher.publishEvent(
      new OnRegistrationCompleteEvent(registered, request.getLocale(), appUrl));

    return new GenericResponse("success");
}
```

那么——这与最初的以 MVC 为中心的实现有什么不同呢？

这是:

*   请求现在被正确地映射到 HTTP POST
*   我们现在返回一个正确的 DTO，并将它直接编组到响应的主体中
*   我们不再处理方法中的错误处理

我们还删除了旧的**`showRegistrationPage()`**——因为不需要简单地显示注册页面。

## 3。`registration.html`

有了这些变化，我们现在需要将`registration.html`修改为:

*   使用 Ajax 提交注册表单
*   以 JSON 的形式接收操作的结果

这是:

```java
<html>
<head>
<title th:text="#{label.form.title}">form</title>
</head>
<body>
<form action="/" method="POST" enctype="utf8">
    <input  name="firstName" value="" />
    <span id="firstNameError" style="display:none"></span>

    <input  name="lastName" value="" />
    <span id="lastNameError" style="display:none"></span>

    <input  name="email" value="" />           
    <span id="emailError" style="display:none"></span>

    <input name="password" value="" type="password" />
    <span id="passwordError" style="display:none"></span>

    <input name="matchingPassword" value="" type="password" />
    <span id="globalError" style="display:none"></span>

    <a href="#" onclick="register()" th:text="#{label.form.submit}>submit</a>
</form>

<script src="jquery.min.js"></script>
<script type="text/javascript">
var serverContext = [[@{/}]];

function register(){
    $(".alert").html("").hide();
    var formData= $('form').serialize();
    $.post(serverContext + "/user/registration",formData ,function(data){
        if(data.message == "success"){
            window.location.href = serverContext +"/successRegister.html";
        }
    })
    .fail(function(data) {
        if(data.responseJSON.error.indexOf("MailError") > -1)
        {
            window.location.href = serverContext + "/emailError.html";
        }
        else if(data.responseJSON.error.indexOf("InternalError") > -1){
            window.location.href = serverContext + 
              "/login.html?message=" + data.responseJSON.message;
        }
        else if(data.responseJSON.error == "UserAlreadyExist"){
            $("#emailError").show().html(data.responseJSON.message);
        }
        else{
            var errors = $.parseJSON(data.responseJSON.message);
            $.each( errors, function( index,item ){
                $("#"+item.field+"Error").show().html(item.defaultMessage);
            });
            errors = $.parseJSON(data.responseJSON.error);
            $.each( errors, function( index,item ){
                $("#globalError").show().append(item.defaultMessage+"<br>");
            });
 }
}
</script>
</body>
</html>
```

## 4。异常处理

随着更多 RESTful API 的出现，异常处理逻辑当然也会变得更加成熟。

我们使用相同的`@ControllerAdvice`机制来干净地处理应用程序抛出的异常——现在我们需要一种新类型的异常。

这是`BindException`–当`UserDto`生效时抛出(如果无效)。我们将覆盖默认的`ResponseEntityExceptionHandler`方法`handleBindException()` ，在响应体中添加错误:

```java
@Override
protected ResponseEntity<Object> handleBindException
  (BindException ex, HttpHeaders headers, HttpStatus status, WebRequest request) {
    logger.error("400 Status Code", ex);
    BindingResult result = ex.getBindingResult();
    GenericResponse bodyOfResponse = 
      new GenericResponse(result.getFieldErrors(), result.getGlobalErrors());

    return handleExceptionInternal(
      ex, bodyOfResponse, new HttpHeaders(), HttpStatus.BAD_REQUEST, request);
}
```

我们还需要处理我们的自定义`Exception``UserAlreadyExistException`——当用户注册一个已经存在的电子邮件时抛出:

```java
@ExceptionHandler({ UserAlreadyExistException.class })
public ResponseEntity<Object> handleUserAlreadyExist(RuntimeException ex, WebRequest request) {
    logger.error("409 Status Code", ex);
    GenericResponse bodyOfResponse = new GenericResponse(
      messages.getMessage("message.regError", null, request.getLocale()), "UserAlreadyExist");

    return handleExceptionInternal(
      ex, bodyOfResponse, new HttpHeaders(), HttpStatus.CONFLICT, request);
}
```

## 5。`GenericResponse`

我们还需要改进`GenericResponse`实现来控制这些验证错误:

```java
public class GenericResponse {

    public GenericResponse(List<FieldError> fieldErrors, List<ObjectError> globalErrors) {
        super();
        ObjectMapper mapper = new ObjectMapper();
        try {
            this.message = mapper.writeValueAsString(fieldErrors);
            this.error = mapper.writeValueAsString(globalErrors);
        } catch (JsonProcessingException e) {
            this.message = "";
            this.error = "";
        }
    }
}
```

## 6。UI–字段和全局错误

最后，让我们看看如何使用 jQuery 处理字段和全局错误:

```java
var serverContext = [[@{/}]];

function register(){
    $(".alert").html("").hide();
    var formData= $('form').serialize();
    $.post(serverContext + "/user/registration",formData ,function(data){
        if(data.message == "success"){
            window.location.href = serverContext +"/successRegister.html";
        }
    })
    .fail(function(data) {
        if(data.responseJSON.error.indexOf("MailError") > -1)
        {
            window.location.href = serverContext + "/emailError.html";
        }
        else if(data.responseJSON.error.indexOf("InternalError") > -1){
            window.location.href = serverContext + 
              "/login.html?message=" + data.responseJSON.message;
        }
        else if(data.responseJSON.error == "UserAlreadyExist"){
            $("#emailError").show().html(data.responseJSON.message);
        }
        else{
            var errors = $.parseJSON(data.responseJSON.message);
            $.each( errors, function( index,item ){
                $("#"+item.field+"Error").show().html(item.defaultMessage);
            });
            errors = $.parseJSON(data.responseJSON.error);
            $.each( errors, function( index,item ){
                $("#globalError").show().append(item.defaultMessage+"<br>");
            });
 }
}
```

请注意:

*   如果存在验证错误，那么`message`对象包含字段错误，而`error`对象包含全局错误
*   我们在字段旁边显示每个字段错误
*   我们在表单末尾的一个地方显示所有的全局错误

## 7 .**。结论**

这篇快速文章的重点是将 API 引入一个更 RESTful 的方向，并展示一种在前端处理该 API 的简单方法。

jQuery 前端本身并不是重点——它只是一个基本的潜在客户端，可以在任意数量的 JS 框架中实现，而 API 保持不变。

本教程的**完整实现**可以在[的 github 项目](https://web.archive.org/web/20220523145855/https://github.com/Baeldung/spring-security-registration "The Full Registration/Authentication Example Project on Github ")中找到——这是一个基于 Eclipse 的项目，所以它应该很容易导入和运行。

Next **»**[Spring Security – Reset Your Password](/web/20220523145855/https://www.baeldung.com/spring-security-registration-i-forgot-my-password)**«** Previous[Registration with Spring Security – Password Encoding](/web/20220523145855/https://www.baeldung.com/spring-security-registration-password-encoding-bcrypt)