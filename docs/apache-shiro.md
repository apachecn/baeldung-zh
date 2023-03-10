# 阿帕奇·希罗简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/apache-shiro>

 ![announcement-icon.png](img/f941b97c607d3bb0797a8d2dc0b8ccf0.png)

It’s just plain hard to get true, **real-time visibility into a running auth flow.**

Parts of the process can be completely hidden from us; if the complete authorization process requires a redirect from a remote OAuth production server, then every debugging effort must go through the production server.

It’s practically unfeasible to debug this locally. There’s no way to reproduce the exact state and no way to inspect what is actually happening under the hood. Not ideal.

Knowing these types of challenges, we built Lightrun - a real-time production debugging tool - to allow you to understand complicated flows with code-level information. Add logs, take snapshots (virtual breakpoints), and instrument metrics without a remote debugger, without stopping the running service, and, most importantly - **in real-time and without side effects**.

**Learn more with this 5-minute tutorial** focused on debugging these kinds of scenarios using Lightrun:

[>> Debugging Authentication and Authorization Using Lightrun](/web/20220524130132/https://www.baeldung.com/lightrun-n-security)

## 1。概述

在本文中，我们将看看[阿帕奇·希罗](https://web.archive.org/web/20220524130132/https://shiro.apache.org/)，一个通用的 Java 安全框架。

该框架是高度可定制和模块化的，因为它提供了认证、授权、加密和会话管理。

## 2。依赖性

阿帕奇·希罗有许多[模块](https://web.archive.org/web/20220524130132/https://shiro.apache.org/download.html)。然而，在本教程中，我们只使用了`shiro-core`工件。

让我们把它添加到我们的`pom.xml`:

```java
<dependency>
    <groupId>org.apache.shiro</groupId>
    <artifactId>shiro-core</artifactId>
    <version>1.4.0</version>
</dependency>
```

最新版本的阿帕奇·希罗模块可以在 Maven Central 上找到。

## 3。配置安全管理器

`SecurityManager`是阿帕奇·希罗框架的中心部分。应用程序通常只有一个运行的实例。

在本教程中，我们将在桌面环境中探索该框架。要配置框架，我们需要在资源文件夹中创建一个`shiro.ini`文件，内容如下:

```java
[users]
user = password, admin
user2 = password2, editor
user3 = password3, author

[roles]
admin = *
editor = articles:*
author = articles:compose,articles:save
```

`shiro.ini`配置文件的`[users]`部分定义了被`SecurityManager`识别的用户凭证。`p`格式为:`rincipal (username) = password, role1, role2, …, role`。

角色及其相关权限在`[roles]`部分声明。`admin` 角色被授予对应用程序每个部分的权限和访问权。这由通配符`(*)`符号表示。

`editor`角色拥有与`articles` 相关的所有权限，而`author`角色只能拥有`compose` 和`save` 一篇文章。

`SecurityManager`用于配置`SecurityUtils`类。从`SecurityUtils`中，我们可以获得与系统交互的当前用户，并执行认证和授权操作。

让我们使用`IniRealm`从`shiro.ini`文件中加载我们的用户和角色定义，然后使用它来配置`DefaultSecurityManager`对象:

```java
IniRealm iniRealm = new IniRealm("classpath:shiro.ini");
SecurityManager securityManager = new DefaultSecurityManager(iniRealm);

SecurityUtils.setSecurityManager(securityManager);
Subject currentUser = SecurityUtils.getSubject();
```

现在我们有了一个知道在`shiro.ini`文件中定义的用户凭证和角色的`SecurityManager`，让我们继续进行用户认证和授权。

## 4。认证

在阿帕奇·希罗的术语中，`Subject`是任何与系统交互的实体。它可能是一个人，一个脚本，或者一个 REST 客户机。

调用`SecurityUtils.getSubject()`返回当前`Subject`的一个实例，即 `currentUser`。

现在我们有了`currentUser`对象，我们可以对提供的凭证执行认证:

```java
if (!currentUser.isAuthenticated()) {               
  UsernamePasswordToken token                       
    = new UsernamePasswordToken("user", "password");
  token.setRememberMe(true);                        
  try {                                             
      currentUser.login(token);                       
  } catch (UnknownAccountException uae) {           
      log.error("Username Not Found!", uae);        
  } catch (IncorrectCredentialsException ice) {     
      log.error("Invalid Credentials!", ice);       
  } catch (LockedAccountException lae) {            
      log.error("Your Account is Locked!", lae);    
  } catch (AuthenticationException ae) {            
      log.error("Unexpected Error!", ae);           
  }                                                 
}
```

首先，我们检查当前用户是否还没有被认证。然后，我们用用户的主体`(username)`和凭证`(password).`创建一个认证令牌

接下来，我们尝试使用令牌登录。如果所提供的凭证是正确的，那么一切都会好的。

不同的情况有不同的例外。也可以抛出一个更适合应用程序需求的自定义异常。这可以通过子类化`AccountException`类来实现。

## 5。授权

身份验证试图验证用户的身份，而授权试图控制对系统中某些资源的访问。

回想一下，我们为在`shiro.ini`文件中创建的每个用户分配了一个或多个角色。此外，在角色部分，我们为每个角色定义了不同的权限或访问级别。

现在让我们看看如何在我们的应用程序中使用它来加强用户访问控制。

在`shiro.ini`文件中，我们给予管理员对系统每个部分的完全访问权。

编辑者可以完全访问关于`articles`的每一个资源/操作，而作者只能编写和保存`articles`。

让我们欢迎基于角色的当前用户:

```java
if (currentUser.hasRole("admin")) {       
    log.info("Welcome Admin");              
} else if(currentUser.hasRole("editor")) {
    log.info("Welcome, Editor!");           
} else if(currentUser.hasRole("author")) {
    log.info("Welcome, Author");            
} else {                                  
    log.info("Welcome, Guest");             
}
```

现在，让我们看看当前用户在系统中被允许做什么:

```java
if(currentUser.isPermitted("articles:compose")) {            
    log.info("You can compose an article");                    
} else {                                                     
    log.info("You are not permitted to compose an article!");
}                                                            

if(currentUser.isPermitted("articles:save")) {               
    log.info("You can save articles");                         
} else {                                                     
    log.info("You can not save articles");                   
}                                                            

if(currentUser.isPermitted("articles:publish")) {            
    log.info("You can publish articles");                      
} else {                                                     
    log.info("You can not publish articles");                
}
```

## 6。领域配置

在实际应用中，我们需要一种从数据库而不是从`shiro.ini`文件中获取用户凭证的方法。这就是境界这个概念发挥作用的地方。

在阿帕奇·希罗的术语中，[领域](https://web.archive.org/web/20220524130132/https://shiro.apache.org/realm.html)是一个 DAO，它指向认证和授权所需的用户凭证的存储。

要创建一个领域，我们只需要实现`Realm`接口。那可能是乏味的；然而，该框架带有默认的实现，我们可以从中派生出子类。其中一个实现就是`JdbcRealm`。

我们创建一个自定义领域实现，它扩展了`JdbcRealm`类并覆盖了以下方法:`doGetAuthenticationInfo()`、`doGetAuthorizationInfo()`、`getRoleNamesForUser()`和`getPermissions()`。

让我们通过子类化`JdbcRealm`类来创建一个领域:

```java
public class MyCustomRealm extends JdbcRealm {
    //...
}
```

为了简单起见，我们使用`java.util.Map`来模拟一个数据库:

```java
private Map<String, String> credentials = new HashMap<>();
private Map<String, Set<String>> roles = new HashMap<>();
private Map<String, Set<String>> perm = new HashMap<>();

{
    credentials.put("user", "password");
    credentials.put("user2", "password2");
    credentials.put("user3", "password3");

    roles.put("user", new HashSet<>(Arrays.asList("admin")));
    roles.put("user2", new HashSet<>(Arrays.asList("editor")));
    roles.put("user3", new HashSet<>(Arrays.asList("author")));

    perm.put("admin", new HashSet<>(Arrays.asList("*")));
    perm.put("editor", new HashSet<>(Arrays.asList("articles:*")));
    perm.put("author", 
      new HashSet<>(Arrays.asList("articles:compose", 
      "articles:save")));
}
```

让我们继续并覆盖`doGetAuthenticationInfo()`:

```java
protected AuthenticationInfo 
  doGetAuthenticationInfo(AuthenticationToken token)
  throws AuthenticationException {

    UsernamePasswordToken uToken = (UsernamePasswordToken) token;

    if(uToken.getUsername() == null
      || uToken.getUsername().isEmpty()
      || !credentials.containsKey(uToken.getUsername())) {
          throw new UnknownAccountException("username not found!");
    }

    return new SimpleAuthenticationInfo(
      uToken.getUsername(), 
      credentials.get(uToken.getUsername()), 
      getName()); 
}
```

我们首先将提供的`AuthenticationToken`转换为`UsernamePasswordToken`。从`uToken`中，我们提取用户名(`uToken.getUsername()`，并使用它从数据库中获取用户凭证(密码)。

如果没有找到记录——我们抛出一个`UnknownAccountException`,否则我们使用凭证和用户名构造一个从方法返回的`SimpleAuthenticatioInfo`对象。

如果用户凭证用 salt 散列，我们需要返回一个带有相关 salt 的`SimpleAuthenticationInfo`:

```java
return new SimpleAuthenticationInfo(
  uToken.getUsername(), 
  credentials.get(uToken.getUsername()), 
  ByteSource.Util.bytes("salt"), 
  getName()
);
```

我们还需要覆盖`doGetAuthorizationInfo()`，以及`getRoleNamesForUser()`和`getPermissions()`。

最后，让我们将自定义领域插入到`securityManager`中。我们需要做的就是用我们的自定义领域替换上面的`IniRealm` ，并将其传递给`DefaultSecurityManager`的构造函数:

```java
Realm realm = new MyCustomRealm();
SecurityManager securityManager = new DefaultSecurityManager(realm);
```

代码的其他部分都和以前一样。这就是我们用自定义领域正确配置`securityManager`所需的全部内容。

现在的问题是——框架如何匹配凭证？

默认情况下，`JdbcRealm`使用`SimpleCredentialsMatcher`，它只是通过比较`AuthenticationToken`和`AuthenticationInfo`中的凭证来检查是否相等。

如果我们散列密码，我们需要通知框架使用一个`HashedCredentialsMatcher`来代替。带散列密码的 INI 配置可以在[这里](https://web.archive.org/web/20220524130132/https://shiro.apache.org/realm.html#Realm-HashingCredentials)找到。

## 7。注销

现在我们已经验证了用户，是时候实现注销了。只需调用一个方法就可以做到这一点，该方法会使用户会话无效并注销用户:

```java
currentUser.logout();
```

## 8。会话管理

该框架自然带有其会话管理系统。如果在 web 环境中使用，它默认为`HttpSession` 实现。

对于独立的应用程序，它使用其企业会话管理系统。好处是，即使在桌面环境中，您也可以像在典型的 web 环境中一样使用会话对象。

让我们看一个简单的例子，并与当前用户的会话进行交互:

```java
Session session = currentUser.getSession();                
session.setAttribute("key", "value");                      
String value = (String) session.getAttribute("key");       
if (value.equals("value")) {                               
    log.info("Retrieved the correct value! [" + value + "]");
}
```

## 9。使用 Spring 的 Web 应用程序 Shiro】

到目前为止，我们已经概述了阿帕奇·希罗的基本结构，并在桌面环境中实现了它。让我们继续将框架集成到 Spring Boot 应用程序中。

注意，这里主要关注的是 Shiro，而不是 Spring 应用程序——我们只打算用它来驱动一个简单的示例应用程序。

### 9.1。依赖性

首先，我们需要将 Spring Boot 父依赖项添加到我们的`pom.xml`:

```java
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.6.1</version>
</parent>
```

接下来，我们必须将以下依赖项添加到同一个`pom.xml`文件中:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-freemarker</artifactId>
</dependency>
<dependency>
    <groupId>org.apache.shiro</groupId>
    <artifactId>shiro-spring-boot-web-starter</artifactId>
    <version>${apache-shiro-core-version}</version>
</dependency>
```

### 9.2。配置

将`shiro-spring-boot-web-starter`依赖项添加到我们的`pom.xml`中将默认配置阿帕奇·希罗应用程序的一些特性，比如`SecurityManager`。

然而，我们仍然需要配置`Realm`和 Shiro 安全过滤器。我们将使用上面定义的相同的自定义领域。

因此，在运行 Spring Boot 应用程序的主类中，让我们添加以下`Bean`定义:

```java
@Bean
public Realm realm() {
    return new MyCustomRealm();
}

@Bean
public ShiroFilterChainDefinition shiroFilterChainDefinition() {
    DefaultShiroFilterChainDefinition filter
      = new DefaultShiroFilterChainDefinition();

    filter.addPathDefinition("/secure", "authc");
    filter.addPathDefinition("/**", "anon");

    return filter;
}
```

在`ShiroFilterChainDefinition`中，我们对`/secure`路径应用了`authc`滤镜，并使用蚂蚁模式对其他路径应用了`anon`滤镜。

默认情况下，web 应用程序会附带`authc` 和`anon`过滤器。其他默认过滤器可以在[这里](https://web.archive.org/web/20220524130132/https://shiro.apache.org/web.html#Web-DefaultFilters)找到。

如果我们没有定义`Realm` bean，默认情况下，`ShiroAutoConfiguration`将提供一个`IniRealm`实现，期望在`src/main/resources`或`src/main/resources/META-INF.`中找到一个`shiro.ini`文件

如果我们没有定义一个`ShiroFilterChainDefinition` bean，框架会保护所有路径，并将登录 URL 设置为`login.jsp`。

我们可以通过向我们的`application.properties`添加以下条目来更改这个默认登录 URL 和其他默认设置:

```java
shiro.loginUrl = /login
shiro.successUrl = /secure
shiro.unauthorizedUrl = /login
```

既然`authc`过滤器已经被应用到了`/secure`，那么对该路由的所有请求都将需要表单认证。

### 9.3。认证和授权

让我们用下面的路径映射创建一个`ShiroSpringController`:`/index`、`/login, /logout` 和`/secure.`

`login()`方法是我们如上所述实现实际用户认证的地方。如果身份验证成功，用户将被重定向到安全页面:

```java
Subject subject = SecurityUtils.getSubject();

if(!subject.isAuthenticated()) {
    UsernamePasswordToken token = new UsernamePasswordToken(
      cred.getUsername(), cred.getPassword(), cred.isRememberMe());
    try {
        subject.login(token);
    } catch (AuthenticationException ae) {
        ae.printStackTrace();
        attr.addFlashAttribute("error", "Invalid Credentials");
        return "redirect:/login";
    }
}

return "redirect:/secure";
```

现在在`secure()`实现中，`currentUser`是通过调用`SecurityUtils.getSubject().`获得的，用户的角色和权限以及用户的主体都被传递到安全页面:

```java
Subject currentUser = SecurityUtils.getSubject();
String role = "", permission = "";

if(currentUser.hasRole("admin")) {
    role = role  + "You are an Admin";
} else if(currentUser.hasRole("editor")) {
    role = role + "You are an Editor";
} else if(currentUser.hasRole("author")) {
    role = role + "You are an Author";
}

if(currentUser.isPermitted("articles:compose")) {
    permission = permission + "You can compose an article, ";
} else {
    permission = permission + "You are not permitted to compose an article!, ";
}

if(currentUser.isPermitted("articles:save")) {
    permission = permission + "You can save articles, ";
} else {
    permission = permission + "\nYou can not save articles, ";
}

if(currentUser.isPermitted("articles:publish")) {
    permission = permission  + "\nYou can publish articles";
} else {
    permission = permission + "\nYou can not publish articles";
}

modelMap.addAttribute("username", currentUser.getPrincipal());
modelMap.addAttribute("permission", permission);
modelMap.addAttribute("role", role);

return "secure";
```

我们完成了。这就是我们如何将阿帕奇·希罗集成到 Spring Boot 应用中的方法。

另外，请注意，框架提供了额外的[注释](https://web.archive.org/web/20220524130132/https://shiro.apache.org/spring.html)，可以与过滤器链定义一起使用，以保护我们的应用程序。

## 10.JEE 一体化

将阿帕奇·希罗集成到 JEE 应用程序中只是配置`web.xml`文件的问题。像往常一样，配置期望`shiro.ini`在类路径中。详细的配置示例可在[这里](https://web.archive.org/web/20220524130132/https://shiro.apache.org/web.html#Web-%7B%7Bweb.xml%7D%7D)找到。此外，JSP 标签可以在[这里](https://web.archive.org/web/20220524130132/https://shiro.apache.org/web.html#Web-JSP%2FGSPTagLibrary)找到。

## 11。结论

在本教程中，我们研究了阿帕奇·希罗的认证和授权机制。我们还关注了如何定义一个自定义领域并将其插入到`SecurityManager`中。

和往常一样，完整的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220524130132/https://github.com/eugenp/tutorials/tree/master/apache-shiro)