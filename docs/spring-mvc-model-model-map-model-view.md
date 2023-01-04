# Spring MVC 中的模型、模型图和模型视图

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-mvc-model-model-map-model-view>

## 1。概述

在本文中，我们将看看 Spring MVC 提供的核心`org.springframework.ui.Model` **、**、`org.springframework.ui.ModelMap`和`org.springframework.web.servlet.ModelAndView`的用法。

## 2。Maven 依赖关系

让我们从`pom.xml`文件中的`spring-context`依赖项开始:

```
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>5.2.2.RELEASE</version>
</dependency>
```

spring-context 依赖的最新版本可以在这里找到[。](https://web.archive.org/web/20220526052619/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22spring-context%22)

对于`ModelAndView`，需要`spring-web`依赖关系:

```
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-web</artifactId>
    <version>5.2.2.RELEASE</version>
</dependency>
```

spring-web dependency 的最新版本可以在这里找到。

而且，如果我们使用百里香叶作为我们的视图，我们应该将这个依赖项添加到 pom.xml:

```
<dependency>
    <groupId>org.thymeleaf</groupId>
    <artifactId>thymeleaf-spring5</artifactId>
    <version>3.0.11.RELEASE</version>
</dependency>
```

百里香依赖的最新版本可以在[这里](https://web.archive.org/web/20220526052619/https://search.maven.org/search?q=a:thymeleaf-spring5%20AND%20g:org.thymeleaf)找到。

## 3。`Model`

让我们从最基本的概念开始——`Model`。

简单地说，模型可以提供用于呈现视图的属性。

为了给视图提供可用的数据，我们只需将这些数据添加到它的`Model` 对象中。此外，带有属性的地图可以与`Model`实例合并:

```
@GetMapping("/showViewPage")
public String passParametersWithModel(Model model) {
    Map<String, String> map = new HashMap<>();
    map.put("spring", "mvc");
    model.addAttribute("message", "Baeldung");
    model.mergeAttributes(map);
    return "viewPage";
}
```

## 4。`ModelMap`

就像上面的 `Model`接口一样， `ModelMap`也用于传递值来渲染视图。

`ModelMap`的优点是它让我们能够传递一组值，并将这些值视为在`Map`中:

```
@GetMapping("/printViewPage")
public String passParametersWithModelMap(ModelMap map) {
    map.addAttribute("welcomeMessage", "welcome");
    map.addAttribute("message", "Baeldung");
    return "viewPage";
}
```

## 5。`ModelAndView`

将值传递给视图的最后一个接口是 `ModelAndView`。

这个接口允许我们在一次返回中传递 Spring MVC 所需的所有信息:

```
@GetMapping("/goToViewPage")
public ModelAndView passParametersWithModelAndView() {
    ModelAndView modelAndView = new ModelAndView("viewPage");
    modelAndView.addObject("message", "Baeldung");
    return modelAndView;
} 
```

## 6。视图

我们放在这些模型中的所有数据都由一个视图使用——通常是一个模板化的视图来呈现网页。

如果我们有一个由控制器的方法作为视图的百里香模板文件。通过模型传递的参数可以从百里香 HTML 代码中访问:

```
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <title>Title</title>
</head>
<body>
    <div>Web Application. Passed parameter : th:text="${message}"</div>
</body>
</html>
```

这里传递的参数是通过语法`${message}`使用的，它被称为占位符。百里香模板引擎将用通过模型传递的同名属性的实际值替换该占位符。

## 7。结论

在这个快速教程中，我们讨论了 Spring MVC 中的三个核心概念——`Model`、`ModelMap`和`ModelAndView`。我们还看了视图如何利用这些值的例子。

和往常一样，所有这些例子和代码片段的实现都可以在 Github 上找到[。](https://web.archive.org/web/20220526052619/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-mvc-basics-4)