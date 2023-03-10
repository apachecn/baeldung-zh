# Spring Mobile 指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-mobile>

## 1。概述

Spring Mobile 是流行的`Spring Web MVC`框架的现代扩展，它有助于简化 web 应用程序的开发，web 应用程序需要完全或部分兼容跨设备平台，只需最少的工作和较少的样板代码。

在本文中，我们将了解 Spring Mobile 项目，并构建一个示例项目来突出 Spring Mobile 的用途。

## 2。弹簧手机的特点

*   **自动设备检测:** Spring Mobile 内置了服务器端设备解析器抽象层。这将分析所有传入的请求并检测发送方设备信息，例如设备类型、操作系统等
*   **站点偏好管理:**使用站点偏好管理，Spring Mobile 允许用户选择移动/平板/普通查看网站。相对来说，这是一种不推荐的技术，因为通过使用`DeviceDelegatingViewresolver` 我们可以根据设备类型持久化视图层，而不需要用户方的任何输入
*   **Site Switcher**:Site Switcher 能够根据用户的设备类型(如手机、桌面等)自动将用户切换到最合适的视图。)
*   **设备感知视图管理器**:通常，根据设备类型，我们会将用户请求转发到特定站点，以处理特定设备。Spring Mobile 的`View Manager`让开发者可以灵活地将所有视图放入预定义的格式中，Spring Mobile 会根据设备类型自动管理不同的视图

## 3。构建应用程序

现在让我们使用 Spring Mobile 和 Spring Boot 和`Freemarker Template Engine`创建一个演示应用程序，并尝试用最少的代码量捕获设备细节。

### 3.1。Maven 依赖关系

在开始之前，我们需要在`pom.xml`中添加以下 Spring Mobile 依赖项:

```java
<dependency>
    <groupId>org.springframework.mobile</groupId>
    <artifactId>spring-mobile-device</artifactId>
    <version>2.0.0.M3</version>
</dependency>
```

请注意，最新的依赖项在 Spring Milestones 存储库中可用，所以让我们也将它添加到我们的`pom.xml`中:

```java
<repositories>
    <repository>
        <id>spring-milestones</id>
        <name>Spring Milestones</name>
        <url>https://repo.spring.io/libs-milestone</url>
        <snapshots>
            <enabled>false</enabled>
        </snapshots>
    </repository>
</repositories>
```

### 3.2。创建 Freemarker 模板

首先，让我们使用 Freemarker 创建我们的索引页面。不要忘记添加必要的依赖项来启用 Freemarker 的自动配置。

因为我们试图检测发送设备并相应地路由请求，所以我们需要创建三个独立的 Freemarker 文件来解决这个问题；一个用于处理移动请求，另一个用于处理平板电脑，最后一个(默认)用于处理普通浏览器请求。

我们需要在`src/main/resources/templates`下创建两个名为`mobile`和`tablet`的文件夹，并相应地放置 Freemarker 文件。最终的结构应该是这样的:

```java
└── src
    └── main
        └── resources
            └── templates
                └── index.ftl
                └── mobile
                    └── index.ftl
                └── tablet
                    └── index.ftl
```

现在，让我们将下面的`HTML`放到`index.ftl`文件中:

```java
<h1>You are into browser version</h1>
```

根据设备类型，我们将改变`<h1>` 标签中的内容，

### 3.3。启用`DeviceDelegatingViewresolver`

为了启用 Spring Mobile `DeviceDelegatingViewresolver`服务，我们需要在 `application.properties:`中放置以下属性

```java
spring.mobile.devicedelegatingviewresolver.enabled: true 
```

当您包含 Spring Mobile starter 时，默认情况下在 Spring Boot 启用站点首选项功能。但是，可以通过将以下属性设置为 false 来禁用它:

```java
spring.mobile.sitepreference.enabled: true
```

### 3.4。添加 Freemarker 属性

为了让 Spring Boot 能够找到并呈现我们的模板，我们需要将以下内容添加到我们的`application.properties`:

```java
spring.freemarker.template-loader-path: classpath:/templates
spring.freemarker.suffix: .ftl
```

### 3.5。创建一个控制器

现在我们需要创建一个`Controller`类来处理传入的请求。我们将使用简单的`@GetMapping`注释来处理请求:

```java
@Controller
public class IndexController {

    @GetMapping("/")
    public String greeting(Device device) {

        String deviceType = "browser";
        String platform = "browser";
        String viewName = "index";

        if (device.isNormal()) {
            deviceType = "browser";
        } else if (device.isMobile()) {
            deviceType = "mobile";
            viewName = "mobile/index";
        } else if (device.isTablet()) {
            deviceType = "tablet";
            viewName = "tablet/index";
        }

        platform = device.getDevicePlatform().name();

        if (platform.equalsIgnoreCase("UNKNOWN")) {
            platform = "browser";
        }

        return viewName;
    }
}
```

这里需要注意一些事情:

*   在处理程序映射方法中，我们传递的是`org.springframework.mobile.device.Device`。这是每个请求注入的设备信息。这是由我们在`apllication.properties`中启用的`DeviceDelegatingViewresolver`完成的
*   `org.springframework.mobile.device.Device`有几个内置的方法，比如`isMobile(), isTablet(), getDevicePlatform()`等等。使用这些，我们可以捕获所有我们需要的设备信息并加以利用

### 3.6。Java 配置

为了在 Spring web 应用程序中启用设备检测，我们还需要添加一些配置:

```java
@Configuration
public class AppConfig implements WebMvcConfigurer {

    @Bean
    public DeviceResolverHandlerInterceptor deviceResolverHandlerInterceptor() { 
        return new DeviceResolverHandlerInterceptor(); 
    }

    @Bean
    public DeviceHandlerMethodArgumentResolver deviceHandlerMethodArgumentResolver() { 
        return new DeviceHandlerMethodArgumentResolver(); 
    }

    @Override
    public void addInterceptors(InterceptorRegistry registry) { 
        registry.addInterceptor(deviceResolverHandlerInterceptor()); 
    }

    @Override
    public void addArgumentResolvers(List<HandlerMethodArgumentResolver> argumentResolvers) {
        argumentResolvers.add(deviceHandlerMethodArgumentResolver()); 
    }
}
```

我们差不多完成了。最后要做的一件事是构建一个 Spring Boot 配置类来启动应用程序:

```java
@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

## 4。测试应用程序

一旦我们启动应用程序，它将在`http://localhost:8080`运行。

**我们将使用`Google Chrome's Developer Console`来模拟不同种类的设备。**我们可以通过按下`ctrl + shift + i` 或 `F12.`来启用它

默认情况下，如果我们打开主页，我们可以看到 Spring Web 将该设备检测为桌面浏览器。我们应该会看到以下结果:

[![browser-300x32](img/427ac62f89d8fcd87c36ed22dc64264a.png)](/web/20220815040239/https://www.baeldung.com/wp-content/uploads/2017/01/browser.png)

现在，在控制台面板上，我们单击左上角的第二个图标。它将启用浏览器的移动视图。

我们可以在浏览器的左上角看到一个下拉菜单。在下拉列表中，我们可以选择不同种类的设备类型。要模拟移动设备，让我们选择 Nexus 6P 并刷新页面。

当我们刷新页面时，我们会注意到页面的**内容发生了变化，因为`DeviceDelegatingViewresolver`已经检测到最后一个请求来自移动设备。**因此，它传递了模板中移动文件夹中的`index.ftl`文件。

结果如下:

[![mobile](img/3bf36255c2571ed2debbcdb033055650.png)](/web/20220815040239/https://www.baeldung.com/wp-content/uploads/2017/01/mobile.png)

同样，我们将模拟一个平板电脑版本。让我们像上次一样从下拉列表中选择 iPad 并刷新页面。内容会发生变化，应该被视为平板电脑视图:

[![tablet](img/712734e73b88baf9513743c5b0ed7070.png)](/web/20220815040239/https://www.baeldung.com/wp-content/uploads/2017/01/tablet.png)

现在，我们将看看站点首选项功能是否按预期工作。

要模拟一个实时场景，用户希望以一种移动友好的方式查看网站，只需在默认 URL 的末尾添加以下 URL 参数:

```java
?site_preference=mobile
```

刷新后，视图应自动移动到移动视图，即显示以下文本“您正在使用移动版本”。

以同样的方式模拟平板电脑偏好，只需在默认 URL 的末尾添加以下 URL 参数:

```java
?site_preference=tablet
```

而且就像上次一样，视图应该会自动刷新为平板视图。

请注意，默认 URL 将保持不变，如果用户再次访问默认 URL，用户将被重定向到基于设备类型的相应视图。

## 5。结论

我们刚刚创建了一个 web 应用程序，并实现了跨平台功能。从生产力的角度来看，这是一个巨大的性能提升。Spring Mobile 消除了许多前端脚本来处理跨浏览器行为，从而减少了开发时间。

像往常一样，更新的源代码在 GitHub 上[。](https://web.archive.org/web/20220815040239/https://github.com/eugenp/tutorials/tree/master/spring-mobile)