# 春季 MVC 主题

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-mvc-themes>

## 1.概观

当设计一个 web 应用程序时，它的外观是一个关键的组成部分。它影响我们的应用程序的可用性和可访问性，并可以进一步建立我们公司的品牌。

在本教程中，我们将介绍在 Spring MVC 应用程序中配置主题所需的步骤。

## 2.用例

简而言之，主题是一组静态资源，通常是样式表和图像，它们会影响 web 应用程序的视觉风格。

我们可以使用主题来:

*   使用`fixed theme`建立共同的外观
*   用`branding theme`为品牌定制——这在 SAAS 应用程序中很常见，每个客户都想要不同的外观和感觉
*   用一个`usability theme`来解决易访问性问题——例如，我们可能想要一个深色或者高对比度的主题

## 3.Maven 依赖性

所以，首先，让我们添加我们将在本教程的第一部分使用的 Maven 依赖项。

我们将需要 [Spring WebMVC](https://web.archive.org/web/20220701013736/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.springframework%22%20AND%20a%3A%22spring-webmvc%22) 和 [Spring Context](https://web.archive.org/web/20220701013736/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.springframework%22%20AND%20a%3A%22spring-context%22) 依赖关系:

```
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-webmvc</artifactId>
    <version>5.2.1.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>5.2.1.RELEASE</version>
</dependency> 
```

由于我们将在示例中使用 JSP，我们将需要[Java servlet](https://web.archive.org/web/20220701013736/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22javax.servlet%22%20AND%20a%3A%22javax.servlet-api%22)、 [JSP](https://web.archive.org/web/20220701013736/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22javax.servlet.jsp%22%20AND%20a%3A%22javax.servlet.jsp-api%22) 和 [JSTL](https://web.archive.org/web/20220701013736/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22javax.servlet%22%20AND%20a%3A%22jstl%22) :

```
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>javax.servlet-api</artifactId>
    <version>4.0.1</version>
</dependency>
<dependency>
     <groupId>javax.servlet.jsp</groupId>
     <artifactId>javax.servlet.jsp-api</artifactId>
     <version>2.3.3</version>
</dependency>
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>jstl</artifactId>
    <version>1.2</version>
</dependency>
```

## 4。配置 Spring 主题

### 4.1.主题属性

现在，让我们为我们的应用程序配置亮和暗的主题。

对于黑暗主题，让我们创建`dark.properties`:

```
styleSheet=themes/black.css
background=black
```

而对于光的主题，`light.properties`:

```
styleSheet=themes/white.css
background=white
```

从上面的属性中，我们注意到一个引用了 CSS 文件，另一个引用了 CSS 样式。一会儿我们会看到这些是如何在我们的视野中显现的。

### 4.2.`ResourceHandler`

阅读上面的属性，文件`black.css`和 `white.css`必须放在名为`/themes`的目录中。

而且，我们必须配置一个`ResourceHandler`来使 Spring MVC 在被请求时正确定位文件:

```
@Override 
public void addResourceHandlers(ResourceHandlerRegistry registry) {
    registry.addResourceHandler("/themes/**").addResourceLocations("classpath:/themes/");
}
```

### 4.3.`ThemeSource`

我们可以管理这些特定主题。`properties`文件为 [`ResourceBundle` s](/web/20220701013736/https://www.baeldung.com/java-resourcebundle) 经`ResourceBundleThemeSource`:

```
@Bean
public ResourceBundleThemeSource resourceBundleThemeSource() {
    return new ResourceBundleThemeSource();
}
```

### 4.4.`ThemeResolver`年代

接下来，我们需要一个 *ThemeResolver* 来为应用程序解析正确的主题。根据我们的设计需求，我们可以在现有的实现中进行选择，或者创建我们自己的实现。

对于我们的例子，让我们如名称所示配置`CookieThemeResolver. `,这将从浏览器 cookie 中解析主题信息，或者如果该信息不可用，则返回到缺省值:

```
@Bean
public ThemeResolver themeResolver() {
    CookieThemeResolver themeResolver = new CookieThemeResolver();
    themeResolver.setDefaultThemeName("light");
    return themeResolver;
}
```

框架附带的`ThemeResolver `的其他变体有:

*   `FixedThemeResolver`:应用程序有固定主题时使用
*   `SessionThemeResolver`:用于允许用户切换当前会话的主题

### 4.5.视角

为了将主题应用到我们的视图中，我们必须配置一种机制来查询资源包。

我们将只保留 JSP 的范围，尽管也可以为其他视图呈现引擎配置类似的查找机制。

对于 JSP，我们可以导入一个标记库来完成这项工作:

```
<%@ taglib prefix="spring" uri="http://www.springframework.org/tags"%>
```

然后我们可以引用任何指定了适当属性名的属性:

```
<link rel="stylesheet" href="<spring:theme code='styleSheet'/>"/>
```

或者:

```
<body bgcolor="<spring:theme code='background'/>">
```

因此，现在让我们将一个名为`index.jsp`的视图添加到我们的应用程序中，并将其放在`WEB-INF/`目录中:

```
<%@ taglib prefix="spring" uri="http://www.springframework.org/tags"%>
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c" %>
<html>
    <head>
        <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
        <link rel="stylesheet" href="<spring:theme code='styleSheet'/>"/>
        <title>Themed Application</title>
    </head>
    <body>
        <header>
            <h1>Themed Application</h1>
            <hr />
        </header>
        <section>
            <h2>Spring MVC Theme Demo</h2>
            <form action="<c:url value='/'/>" method="POST" name="themeChangeForm" id="themeChangeForm">
                <div>
                    <h4>
                        Change Theme
                    </h4>
                </div>
                <select id="theme" name="theme" onChange="submitForm()">
                    <option value="">Reset</option>
                    <option value="light">Light</option>
                    <option value="dark">Dark</option>
                </select>
            </form>
        </section>

        <script type="text/javascript">
            function submitForm() {
                document.themeChangeForm.submit();
            }
        </script>
    </body>
</html>
```

事实上，我们的应用程序将在这一点上工作，总是选择我们的光主题。

让我们看看如何让用户改变他们的主题。

### 4.6.`ThemeChangeInterceptor`

`ThemeChangeInterceptor`的工作是理解主题变更请求。

现在让我们添加一个`ThemeChangeInterceptor`并配置它来寻找一个`theme`请求参数:

```
@Override
public void addInterceptors(InterceptorRegistry registry) {
    registry.addInterceptor(themeChangeInterceptor());
}

@Bean
public ThemeChangeInterceptor themeChangeInterceptor() {
    ThemeChangeInterceptor interceptor = new ThemeChangeInterceptor();
    interceptor.setParamName("theme");
    return interceptor;
}
```

## 5.进一步的依赖性

接下来，让我们实现自己的`ThemeResolver` ,将用户的偏好存储到数据库中。

为了实现这一点，我们将需要 [Spring Security](https://web.archive.org/web/20220701013736/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.springframework.security%22%20AND%20a%3A%22spring-security-web%22) 来识别用户:

```
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-web</artifactId>
    <version>5.2.1.RELEASE</version>
</dependency>

<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-config</artifactId>
    <version>5.2.1.RELEASE</version>
</dependency>

<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-taglibs</artifactId>
    <version>5.2.1.RELEASE</version>
</dependency>
```

以及用于存储用户偏好的[春季数据](https://web.archive.org/web/20220701013736/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.springframework.data%22%20AND%20a%3A%22spring-data-jpa%22)、[休眠、](https://web.archive.org/web/20220701013736/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.hibernate%22%20AND%20a%3A%22hibernate-core%22)和 [HSQLDB](https://web.archive.org/web/20220701013736/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.hsqldb%22%20AND%20a%3A%22hsqldb%22) :

```
<dependency>
    <groupId>org.springframework.data</groupId>
    <artifactId>spring-data-jpa</artifactId>
    <version>2.2.2.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.hibernate</groupId>
    <artifactId>hibernate-core</artifactId>
    <version>5.4.9.Final</version>
</dependency>

<dependency>
    <groupId>org.hsqldb</groupId>
    <artifactId>hsqldb</artifactId>
    <version>2.5.0</version>
</dependency> 
```

## 6.自定义`ThemeResolver`

现在让我们更深入地研究`ThemeResolver `并实现我们自己的一个。这个自定义的`ThemeResolver` 会将用户的主题偏好保存到数据库中。

为了实现这一点，让我们首先添加一个`UserPreference `实体:

```
@Entity
@Table(name = "preferences")
public class UserPreference {
    @Id
    private String username;

    private String theme;
}
```

接下来，我们将创建`UserPreferenceThemeResolver`，它必须实现`ThemeResolver `接口。它的主要职责是解析和保存主题信息。

让我们首先通过实现`UserPreferenceThemeResolver#resolveThemeName`来解决名称解析问题:

```
@Override
public String resolveThemeName(HttpServletRequest request) {
    String themeName = findThemeFromRequest(request)
      .orElse(findUserPreferredTheme().orElse(getDefaultThemeName()));
    request.setAttribute(THEME_REQUEST_ATTRIBUTE_NAME, themeName);
    return themeName;
}

private Optional<String> findUserPreferredTheme() {
    Authentication authentication = SecurityContextHolder.getContext()
            .getAuthentication();
    UserPreference userPreference = getUserPreference(authentication).orElse(new UserPreference());
    return Optional.ofNullable(userPreference.getTheme());
}

private Optional<String> findThemeFromRequest(HttpServletRequest request) {
    return Optional.ofNullable((String) request.getAttribute(THEME_REQUEST_ATTRIBUTE_NAME));
}

private Optional<UserPreference> getUserPreference(Authentication authentication) {
    return isAuthenticated(authentication) ? 
      userPreferenceRepository.findById(((User) authentication.getPrincipal()).getUsername()) : 
      Optional.empty();
}
```

现在我们可以在`UserPreferenceThemeResolver#setThemeName`中编写保存主题的实现了:

```
@Override
public void setThemeName(HttpServletRequest request, HttpServletResponse response, String theme) {
    Authentication authentication = SecurityContextHolder.getContext()
        .getAuthentication();
    if (isAuthenticated(authentication)) {
        request.setAttribute(THEME_REQUEST_ATTRIBUTE_NAME, theme);
        UserPreference userPreference = getUserPreference(authentication).orElse(new UserPreference());
        userPreference.setUsername(((User) authentication.getPrincipal()).getUsername());
        userPreference.setTheme(StringUtils.hasText(theme) ? theme : null);
        userPreferenceRepository.save(userPreference);
    }
}
```

最后，现在让我们改变应用程序中的`ThemeResolver `:

```
@Bean 
public ThemeResolver themeResolver() { 
    return new UserPreferenceThemeResolver();
}
```

现在，用户的主题偏好保存在数据库中，而不是作为 cookie 保存。

保存用户偏好的另一种方法是通过 Spring MVC 控制器和单独的 API。

## 7.结论

在本文中，我们学习了配置 Spring MVC 主题的步骤。

我们也可以在 GitHub 上找到完整的代码[。](https://web.archive.org/web/20220701013736/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-mvc-views)