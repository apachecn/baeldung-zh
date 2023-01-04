# Reddit 应用程序的第四轮改进

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/reddit-web-app-improvements-4>

## 1。概述

在本教程中，我们将继续改进我们正在构建的简单 Reddit 应用程序，它是[公共案例研究](/web/20220813071757/https://www.baeldung.com/case-study-a-reddit-app-with-spring)的一部分。

## 2。为管理员提供更好的表格

首先，我们将通过使用 jQuery DataTable 插件，使管理页面中的表与面向用户的应用程序中的表处于相同的级别。

### 2.1。让用户分页–服务层

让我们在服务层中添加启用分页的操作:

```java
public List<User> getUsersList(int page, int size, String sortDir, String sort) {
    PageRequest pageReq = new PageRequest(page, size, Sort.Direction.fromString(sortDir), sort);
    return userRepository.findAll(pageReq).getContent();
}
public PagingInfo generatePagingInfo(int page, int size) {
    return new PagingInfo(page, size, userRepository.count());
}
```

### 2.2。用户 DTO

下一步——现在让我们确保一致地将 dto 干净地返回给客户端。

我们将需要一个用户 DTO，因为到目前为止，API 将实际的`User`实体返回给客户端:

```java
public class UserDto {
    private Long id;

    private String username;

    private Set<Role> roles;

    private long scheduledPostsCount;
}
```

### 2.3。让用户分页——在控制器中

现在，让我们也在控制器层实现这个简单的操作:

```java
public List<UserDto> getUsersList(
  @RequestParam(value = "page", required = false, defaultValue = "0") int page, 
  @RequestParam(value = "size", required = false, defaultValue = "10") int size,
  @RequestParam(value = "sortDir", required = false, defaultValue = "asc") String sortDir, 
  @RequestParam(value = "sort", required = false, defaultValue = "username") String sort, 
  HttpServletResponse response) {
    response.addHeader("PAGING_INFO", userService.generatePagingInfo(page, size).toString());
    List<User> users = userService.getUsersList(page, size, sortDir, sort);

    return users.stream().map(
      user -> convertUserEntityToDto(user)).collect(Collectors.toList());
}
```

这是 DTO 转换逻辑:

```java
private UserDto convertUserEntityToDto(User user) {
    UserDto dto = modelMapper.map(user, UserDto.class);
    dto.setScheduledPostsCount(scheduledPostService.countScheduledPostsByUser(user));
    return dto;
}
```

### 2.4。前端

最后，在客户端，让我们使用这个新操作并重新实现我们的管理员用户页面:

```java
<table><thead><tr>
<th>Username</th><th>Scheduled Posts Count</th><th>Roles</th><th>Actions</th>
</tr></thead></table>

<script>           
$(function(){
    $('table').dataTable( {
        "processing": true,
        "searching":false,
        "columnDefs": [
            { "name": "username",   "targets": 0},
            { "name": "scheduledPostsCount",   "targets": 1,"orderable": false},
            { "targets": 2, "data": "roles", "width":"20%", "orderable": false, 
              "render": 
                function ( data, type, full, meta ) { return extractRolesName(data); } },
            { "targets": 3, "data": "id", "render": function ( data, type, full, meta ) {
                return '<a onclick="showEditModal('+data+',\'' + 
                  extractRolesName(full.roles)+'\')">Modify User Roles</a>'; }}
                     ],
        "columns": [
            { "data": "username" },
            { "data": "scheduledPostsCount" }
        ],
        "serverSide": true,
        "ajax": function(data, callback, settings) {
            $.get('admin/users', {
                size: data.length, 
                page: (data.start/data.length), 
                sortDir: data.order[0].dir, 
                sort: data.columns[data.order[0].column].name
            }, function(res,textStatus, request) {
                var pagingInfo = request.getResponseHeader('PAGING_INFO');
                var total = pagingInfo.split(",")[0].split("=")[1];
                callback({
                    recordsTotal: total,recordsFiltered: total,data: res
            });});
        }
});});
</script>
```

## 3。禁用用户

接下来，我们将构建一个简单的管理特性—**禁用用户**的能力。

我们首先需要的是`User`实体中的`enabled`字段:

```java
private boolean enabled;
```

然后，我们可以在我们的`UserPrincipal`实现中使用它来确定主体是否被启用:

```java
public boolean isEnabled() {
    return user.isEnabled();
}
```

这里是处理禁用/启用用户的 API 操作:

```java
@PreAuthorize("hasRole('USER_WRITE_PRIVILEGE')")
@RequestMapping(value = "/users/{id}", method = RequestMethod.PUT)
@ResponseStatus(HttpStatus.OK)
public void setUserEnabled(@PathVariable("id") Long id, 
  @RequestParam(value = "enabled") boolean enabled) {
    userService.setUserEnabled(id, enabled);
}
```

下面是简单的服务层实现:

```java
public void setUserEnabled(Long userId, boolean enabled) {
    User user = userRepository.findOne(userId);
    user.setEnabled(enabled);
    userRepository.save(user);
}
```

## 4。处理会话超时

接下来，让我们配置应用程序**来处理会话超时**——我们将向我们的上下文[添加一个简单的`SessionListener`来控制会话超时](/web/20220813071757/https://www.baeldung.com/servlet-session-timeout):

```java
public class SessionListener implements HttpSessionListener {

    @Override
    public void sessionCreated(HttpSessionEvent event) {
        event.getSession().setMaxInactiveInterval(5 * 60);
    }
}
```

下面是 Spring 安全配置:

```java
protected void configure(HttpSecurity http) throws Exception {
    http 
    ...
        .sessionManagement()
        .invalidSessionUrl("/?invalidSession=true")
        .sessionFixation().none();
}
```

注意:

*   我们将会话超时设置为 5 分钟。
*   当会话到期时，用户将被重定向到登录页面。

## 5。增强注册

接下来，我们将通过添加一些以前缺少的功能来增强注册流程。

我们在这里只说明要点；要深入注册，请查看 **[注册系列](/web/20220813071757/https://www.baeldung.com/spring-security-registration)** 。

### 5.1。注册确认电子邮件

注册时缺少的一个功能是用户不会被提示确认他们的电子邮件。

我们现在让用户在系统中激活之前先确认他们的电子邮件地址:

```java
public void register(HttpServletRequest request, 
  @RequestParam("username") String username, 
  @RequestParam("email") String email, 
  @RequestParam("password") String password) {
    String appUrl = 
      "http://" + request.getServerName() + ":" + 
       request.getServerPort() + request.getContextPath();
    userService.registerNewUser(username, email, password, appUrl);
}
```

服务层也需要做一些工作，基本上是确保用户最初是禁用的:

```java
@Override
public void registerNewUser(String username, String email, String password, String appUrl) {
    ...
    user.setEnabled(false);
    userRepository.save(user);
    eventPublisher.publishEvent(new OnRegistrationCompleteEvent(user, appUrl));
}
```

现在确认一下:

```java
@RequestMapping(value = "/user/regitrationConfirm", method = RequestMethod.GET)
public String confirmRegistration(Model model, @RequestParam("token") String token) {
    String result = userService.confirmRegistration(token);
    if (result == null) {
        return "redirect:/?msg=registration confirmed successfully";
    }
    model.addAttribute("msg", result);
    return "submissionResponse";
}
```

```java
public String confirmRegistration(String token) {
    VerificationToken verificationToken = tokenRepository.findByToken(token);
    if (verificationToken == null) {
        return "Invalid Token";
    }

    Calendar cal = Calendar.getInstance();
    if ((verificationToken.getExpiryDate().getTime() - cal.getTime().getTime()) <= 0) {
        return "Token Expired";
    }

    User user = verificationToken.getUser();
    user.setEnabled(true);
    userRepository.save(user);
    return null;
}
```

### 5.2。触发密码重置

现在，让我们看看如何允许用户在忘记密码的情况下重置自己的密码:

```java
@RequestMapping(value = "/users/passwordReset", method = RequestMethod.POST)
@ResponseStatus(HttpStatus.OK)
public void passwordReset(HttpServletRequest request, @RequestParam("email") String email) {
    String appUrl = "http://" + request.getServerName() + ":" + 
      request.getServerPort() + request.getContextPath();
    userService.resetPassword(email, appUrl);
}
```

现在，服务层只需向用户发送一封电子邮件，其中包含用户可以重置密码的链接:

```java
public void resetPassword(String userEmail, String appUrl) {
    Preference preference = preferenceRepository.findByEmail(userEmail);
    User user = userRepository.findByPreference(preference);
    if (user == null) {
        throw new UserNotFoundException("User not found");
    }

    String token = UUID.randomUUID().toString();
    PasswordResetToken myToken = new PasswordResetToken(token, user);
    passwordResetTokenRepository.save(myToken);
    SimpleMailMessage email = constructResetTokenEmail(appUrl, token, user);
    mailSender.send(email);
}
```

### 5.3。重置密码

一旦用户点击邮件中的链接，他们实际上可以**执行重置密码操作**:

```java
@RequestMapping(value = "/users/resetPassword", method = RequestMethod.GET)
public String resetPassword(
  Model model, 
  @RequestParam("id") long id, 
  @RequestParam("token") String token) {
    String result = userService.checkPasswordResetToken(id, token);
    if (result == null) {
        return "updatePassword";
    }
    model.addAttribute("msg", result);
    return "submissionResponse";
}
```

服务层:

```java
public String checkPasswordResetToken(long userId, String token) {
    PasswordResetToken passToken = passwordResetTokenRepository.findByToken(token);
    if ((passToken == null) || (passToken.getUser().getId() != userId)) {
        return "Invalid Token";
    }

    Calendar cal = Calendar.getInstance();
    if ((passToken.getExpiryDate().getTime() - cal.getTime().getTime()) <= 0) {
        return "Token Expired";
    }

    UserPrincipal userPrincipal = new UserPrincipal(passToken.getUser());
    Authentication auth = new UsernamePasswordAuthenticationToken(
      userPrincipal, null, userPrincipal.getAuthorities());
    SecurityContextHolder.getContext().setAuthentication(auth);
    return null;
}
```

最后，下面是更新密码的实现:

```java
@RequestMapping(value = "/users/updatePassword", method = RequestMethod.POST)
@ResponseStatus(HttpStatus.OK)
public void changeUserPassword(@RequestParam("password") String password) {
    userService.changeUserPassword(userService.getCurrentUser(), password);
}
```

### 5.4。更改密码

接下来，我们将实现类似的功能—在内部更改您的密码:

```java
@RequestMapping(value = "/users/changePassword", method = RequestMethod.POST)
@ResponseStatus(HttpStatus.OK)
public void changeUserPassword(@RequestParam("password") String password, 
  @RequestParam("oldpassword") String oldPassword) {
    User user = userService.getCurrentUser();
    if (!userService.checkIfValidOldPassword(user, oldPassword)) {
        throw new InvalidOldPasswordException("Invalid old password");
    }
    userService.changeUserPassword(user, password);
}
```

```java
public void changeUserPassword(User user, String password) {
    user.setPassword(passwordEncoder.encode(password));
    userRepository.save(user);
}
```

## 6。启动项目

接下来，让我们将项目转换/升级到 Spring Boot；首先，我们将修改`pom.xml`:

```java
...
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.2.5.RELEASE</version>
</parent>

<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <dependency>
       <groupId>org.aspectj</groupId>
       <artifactId>aspectjweaver</artifactId>
     </dependency>
...
```

并且还为**提供了一个简单的启动应用程序用于启动**:

```java
@SpringBootApplication
public class Application {

    @Bean
    public SessionListener sessionListener() {
        return new SessionListener();
    }

    @Bean
    public RequestContextListener requestContextListener() {
        return new RequestContextListener();
    }

    public static void main(String... args) {
        SpringApplication.run(Application.class, args);
    }
}
```

注意**新的基础 URL** 现在将是`http://localhost:8080`而不是旧的`http://localhost:8080/reddit-scheduler`。

## 7 .**。外部化属性**

现在我们已经启动了，我们可以使用`@ConfigurationProperties`来具体化我们的 Reddit 属性:

```java
@ConfigurationProperties(prefix = "reddit")
@Component
public class RedditProperties {

    private String clientID;
    private String clientSecret;
    private String accessTokenUri;
    private String userAuthorizationUri;
    private String redirectUri;

    public String getClientID() {
        return clientID;
    }

    ...
}
```

我们现在可以以类型安全的方式干净地使用这些属性:

```java
@Autowired
private RedditProperties redditProperties;

@Bean
public OAuth2ProtectedResourceDetails reddit() {
    AuthorizationCodeResourceDetails details = new AuthorizationCodeResourceDetails();
    details.setClientId(redditProperties.getClientID());
    details.setClientSecret(redditProperties.getClientSecret());
    details.setAccessTokenUri(redditProperties.getAccessTokenUri());
    details.setUserAuthorizationUri(redditProperties.getUserAuthorizationUri());
    details.setPreEstablishedRedirectUri(redditProperties.getRedirectUri());
    ...
    return details;
}
```

## 8。结论

这一轮的改进对于应用程序来说是一个很好的进步。

我们不再添加任何主要特性，这使得架构改进成为下一个合乎逻辑的步骤——这就是本文所要讨论的。