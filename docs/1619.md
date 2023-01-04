# 使用 Spring Security 跟踪登录的用户

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-security-track-logged-in-users>

## 1。概述

在这个快速教程中，我们将展示一个例子，说明如何使用 Spring Security 在应用程序中**跟踪当前登录的用户。**

为此，我们将通过在用户登录时添加用户并在他们注销时删除用户来跟踪登录用户的列表。

我们将利用`HttpSessionBindingListener`来更新登录用户的列表，无论用户信息是基于用户登录到系统还是从系统注销而添加到会话还是从会话中删除。

## 2。活跃用户存储

为了简单起见，我们将为登录用户定义一个充当内存存储的类: 

```
public class ActiveUserStore {

    public List<String> users;

    public ActiveUserStore() {
        users = new ArrayList<String>();
    }

    // standard getter and setter
}
```

我们将把它定义为 Spring 上下文中的标准 bean:

```
@Bean
public ActiveUserStore activeUserStore(){
    return new ActiveUserStore();
}
```

## 3。`HTTPSessionBindingListener`

现在，我们将利用`HTTPSessionBindingListener`接口并创建一个包装类来表示当前登录的用户。

这将基本上侦听类型为`HttpSessionBindingEvent`的事件，每当设置或删除值时，或者换句话说，绑定或解除绑定到 HTTP 会话时，都会触发这些事件:

```
@Component
public class LoggedUser implements HttpSessionBindingListener, Serializable {

    private static final long serialVersionUID = 1L;
    private String username; 
    private ActiveUserStore activeUserStore;

    public LoggedUser(String username, ActiveUserStore activeUserStore) {
        this.username = username;
        this.activeUserStore = activeUserStore;
    }

    public LoggedUser() {}

    @Override
    public void valueBound(HttpSessionBindingEvent event) {
        List<String> users = activeUserStore.getUsers();
        LoggedUser user = (LoggedUser) event.getValue();
        if (!users.contains(user.getUsername())) {
            users.add(user.getUsername());
        }
    }

    @Override
    public void valueUnbound(HttpSessionBindingEvent event) {
        List<String> users = activeUserStore.getUsers();
        LoggedUser user = (LoggedUser) event.getValue();
        if (users.contains(user.getUsername())) {
            users.remove(user.getUsername());
        }
    }

    // standard getter and setter
}
```

监听器有两个需要实现的方法，`valueBound()`和`valueUnbound()`用于触发它正在监听的事件的两种类型的动作。每当在会话中设置或移除实现侦听器的类型的值，或者会话失效时，都会调用这两个方法。

在我们的例子中，当用户登录时将调用`valueBound()`方法，当用户注销或会话到期时将调用`valueUnbound()`方法。

在每个方法中，我们检索与事件相关联的值，然后根据该值是否与会话绑定，在登录用户列表中添加或删除用户名。

## 4。 **跟踪登录和注销**

现在我们需要跟踪用户成功登录或注销的时间，以便我们可以在会话中添加或删除一个活动用户。在 Spring 安全应用程序中，这可以通过实现`AuthenticationSuccessHandler`和`LogoutSuccessHandler`接口来实现。

### 4.1。实施`AuthenticationSuccessHandler`

对于登录操作，我们将通过覆盖为我们提供对`session`和`authentication`对象的访问的`onAuthenticationSuccess()`方法，将登录用户的用户名设置为会话的一个属性:

```
@Component("myAuthenticationSuccessHandler")
public class MySimpleUrlAuthenticationSuccessHandler implements AuthenticationSuccessHandler {

    @Autowired
    ActiveUserStore activeUserStore;

    @Override
    public void onAuthenticationSuccess(HttpServletRequest request, 
      HttpServletResponse response, Authentication authentication) 
      throws IOException {
        HttpSession session = request.getSession(false);
        if (session != null) {
            LoggedUser user = new LoggedUser(authentication.getName(), activeUserStore);
            session.setAttribute("user", user);
        }
    }
}
```

### 4.2。实施`LogoutSuccessHandler`

对于注销操作，我们将通过覆盖`LogoutSuccessHandler`接口的`onLogoutSuccess()`方法来删除用户属性:

```
@Component("myLogoutSuccessHandler")
public class MyLogoutSuccessHandler implements LogoutSuccessHandler{
    @Override
    public void onLogoutSuccess(HttpServletRequest request, 
      HttpServletResponse response, Authentication authentication)
      throws IOException, ServletException {
        HttpSession session = request.getSession();
        if (session != null){
            session.removeAttribute("user");
        }
    }
}
```

## 5。控制器和视图

为了查看上面的所有操作，我们将为 URL `“/users”`创建一个控制器映射，该映射将检索用户列表，将其添加为模型属性并返回`users.html`视图:

### 5.1。控制器

```
@Controller
public class UserController {

    @Autowired
    ActiveUserStore activeUserStore;

    @GetMapping("/loggedUsers")
    public String getLoggedUsers(Locale locale, Model model) {
        model.addAttribute("users", activeUserStore.getUsers());
        return "users";
    }
}
```

### 5.2。Users.html

```
<html>
<body>
    <h2>Currently logged in users</h2>
    <div th:each="user : ${users}">
        <p th:text="${user}">user</p>
    </div>
</body>
</html> 
```

## 6。使用`Sessionregistry`的替代方法

检索当前登录用户的另一种方法是利用 Spring 的`SessionRegistry`，这是一个管理用户和会话的类。这个类有方法`getAllPrincipals()`来获取用户列表。

对于每个用户，我们可以通过调用方法`getAllSessions()`来查看他们所有会话的列表。为了只获取当前登录的用户，我们必须通过将`getAllSessions()`的第二个参数设置为`false`来排除过期的会话:

```
@Autowired
private SessionRegistry sessionRegistry;

@Override
public List<String> getUsersFromSessionRegistry() {
    return sessionRegistry.getAllPrincipals().stream()
      .filter(u -> !sessionRegistry.getAllSessions(u, false).isEmpty())
      .map(Object::toString)
      .collect(Collectors.toList());
}
```

为了使用`SessionRegistry`类，我们必须定义 bean 并将其应用于会话管理，如下所示:

```
http
  .sessionManagement()
  .maximumSessions(1).sessionRegistry(sessionRegistry())

...

@Bean
public SessionRegistry sessionRegistry() {
    return new SessionRegistryImpl();
}
```

## 7。结论

在本文中，我们展示了如何确定 Spring 安全应用程序中当前登录的用户。

本教程的实现可以在 GitHub 项目中找到——这是一个基于 Maven 的项目，所以它应该很容易导入和运行。