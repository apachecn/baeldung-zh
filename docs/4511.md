# JavaServer Faces (JSF)与 Spring

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-jsf>

## 1。概述

在本文中，我们将研究从 JSF 管理的 bean 和 JSF 页面中访问 Spring 中定义的 bean 的方法，目的是将业务逻辑的执行委托给 Spring beans。

这篇文章假设读者已经分别了解了 JSF 和春天。这篇文章基于 JSF 的 Mojarra 实现。

## 2。春天

让我们在 Spring 中定义下面的 bean。`UserManagementDAO` bean 将用户名添加到内存存储中，它由以下接口定义:

```
public interface UserManagementDAO {
    boolean createUser(String newUserData);
}
```

使用以下 Java 配置来配置 bean 的实现:

```
public class SpringCoreConfig {
    @Bean
    public UserManagementDAO userManagementDAO() {
        return new UserManagementDAOImpl();
    }
}
```

或者使用以下 XML 配置:

```
<bean class="org.springframework.context.annotation.CommonAnnotationBeanPostProcessor" />
<bean class="com.baeldung.dao.UserManagementDAOImpl" id="userManagementDAO"/>
```

我们在 XML 中定义 bean，并注册`CommonAnnotationBeanPostProcessor`以确保获得`@PostConstruct`注释。

## 3。配置

以下部分解释了支持 Spring 和 JSF 上下文集成的配置项。

### 3.1。Java 配置不带`web.xml`

通过实现`WebApplicationInitializer`,我们能够编程配置`ServletContext.` 。下面是`MainWebAppInitializer`类中的`onStartup()`实现:

```
public void onStartup(ServletContext sc) throws ServletException {
    AnnotationConfigWebApplicationContext root = new AnnotationConfigWebApplicationContext();
    root.register(SpringCoreConfig.class);
    sc.addListener(new ContextLoaderListener(root));
}
```

`AnnotationConfigWebApplicationContext`引导 Spring'g 上下文并通过注册`SpringCoreConfig`类添加 beans。

类似地，在 Mojarra 实现中有一个配置`FacesServlet.` 的`FacesInitializer`类，使用这个配置就足以扩展`FacesInitializer.` 。`MainWebAppInitializer,` 的完整实现如下:

```
public class MainWebAppInitializer extends FacesInitializer implements WebApplicationInitializer {
    public void onStartup(ServletContext sc) throws ServletException {
        AnnotationConfigWebApplicationContext root = new AnnotationConfigWebApplicationContext();
        root.register(SpringCoreConfig.class);
        sc.addListener(new ContextLoaderListener(root));
    }
}
```

### 3.2。`web.xml`同

我们将从配置应用程序的`web.xml`文件中的`ContextLoaderListener`开始:

```
<listener>
    <listener-class>
        org.springframework.web.context.ContextLoaderListener
    </listener-class>
</listener>
```

这个监听器负责在 web 应用程序启动时启动 Spring 应用程序上下文。默认情况下，这个监听器将查找名为`applicationContext.xml`的 spring 配置文件。

### 3.3。`faces-config.xml`

我们现在在`face-config.xml`文件中配置`SpringBeanFacesELResolver`:

```
<el-resolver>org.springframework.web.jsf.el.SpringBeanFacesELResolver</el-resolver>
```

EL 解析器是 JSF 框架支持的可插拔组件，允许我们在评估表达式语言(EL)表达式时自定义 JSF 运行时的行为。这个 EL resolver 将允许 JSF 运行时通过 JSF 定义的 EL 表达式访问 Spring 组件。

## 4。在 JSF 访问春豆

此时，我们的 JSF web 应用程序已经准备好从 JSF 后台 bean 或 JSF 页面访问我们的 Spring bean。

### 4.1。来自一个撑腰豆 JSF 2.0

现在可以从 JSF 支持 bean 中访问 Spring bean。根据您运行的 JSF 版本，有两种可能的方法。在 JSF 2.0 中，您可以在 JSF 管理的 bean 上使用`@ManagedProperty`注释。

```
@ManagedBean(name = "registration")
@RequestScoped
public class RegistrationBean implements Serializable {
    @ManagedProperty(value = "#{userManagementDAO}")
    transient private IUserManagementDAO theUserDao;

    private String userName;
```

```
 // getters and setters
}
```

请注意，现在使用`@ManagedProperty.`
时，getter 和 setter 是强制的——为了从受管 bean 断言 Spring bean 的可访问性，我们将添加`createNewUser()`方法:

```
public void createNewUser() {
    FacesContext context = FacesContext.getCurrentInstance();
    boolean operationStatus = userDao.createUser(userName);
    context.isValidationFailed();
    if (operationStatus) {
        operationMessage = "User " + userName + " created";
    }
} 
```

该方法的要点是使用`userDao` Spring bean，并访问其功能。

### 4.2。来自 JSF 2.2 的一个后援豆

另一种方法，只在 JSF2.2 和更高版本中有效，是使用 CDI 的`@Inject`注释。这适用于 JSF 管理的 bean(带有`@ManagedBean`注释)和 CDI 管理的 bean(带有`@Named`注释)。

事实上，使用 CDI 注释，这是注入 bean 的唯一有效方法:

```
@Named( "registration")
@RequestScoped
public class RegistrationBean implements Serializable {
    @Inject
    UserManagementDAO theUserDao;
}
```

使用这种方法，getter 和 setter 是不必要的。还要注意没有 EL 表达式。

### 4.3。从 JSF 的角度来看

从下面的 JSF 页面将触发`createNewUser()`方法:

```
<h:form>
    <h:panelGrid id="theGrid" columns="3">
        <h:outputText value="Username"/>
        <h:inputText id="firstName" binding="#{userName}" required="true"
          requiredMessage="#{msg['message.valueRequired']}" value="#{registration.userName}"/>
        <h:message for="firstName" style="color:red;"/>
        <h:commandButton value="#{msg['label.saveButton']}" action="#{registration.createNewUser}"
          process="@this"/>
        <h:outputText value="#{registration.operationMessage}" style="color:green;"/>
    </h:panelGrid>
</h:form> 
```

要呈现页面，请启动服务器并导航到:

```
http://localhost:8080/jsf/index.jsf
```

我们还可以在 JSF 视图中使用 EL 来访问 Spring bean。要测试它，只需将前面介绍的 JSF 页面中的第 7 行改为:

```
<h:commandButton value="Save"
  action="#{registration.userDao.createUser(userName.value)}"/>
```

这里，我们直接在 Spring DAO 上调用`createUser`方法，从 JSF 页面内部将`userName` 的绑定值传递给该方法，绕过了所有受管 bean。

## 5。结论

我们研究了 Spring 和 JSF 上下文之间的基本集成，其中我们能够访问 JSF bean 和页面中的 Spring bean。

值得注意的是，虽然 JSF 运行时提供了可插拔的架构，使 Spring 框架能够提供集成组件，但是 Spring 框架的注释不能在 JSF 环境中使用，反之亦然。

这意味着你将不能使用像`@Autowired`或`@Component`这样的注释。在 JSF 管理的 bean 中，或者在 Spring 管理的 bean 上使用`@ManagedBean`注释。但是，您可以在 JSF 2.2+托管 bean 和 Spring bean 中使用`@Inject`注释(因为 Spring 支持 JSR-330)。

本文附带的源代码可以从 [GitHub](https://web.archive.org/web/20220118052106/https://github.com/eugenp/tutorials/tree/master/jsf) 获得。