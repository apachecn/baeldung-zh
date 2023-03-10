# 在百里香叶中使用数组

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/thymeleaf-arrays>

## 1.概观

在这个快速教程中，我们将看到如何在百里香叶中使用数组。为了便于设置，我们将使用一个 spring-boot 初始化器来引导我们的应用程序。

Spring MVC 和百里香 leaf 的基础可以在[这里](/web/20221208143909/https://www.baeldung.com/thymeleaf-in-spring-mvc)找到。

## 2.百里香叶依赖性

在我们的`pom.xml`文件中，我们需要添加的唯一依赖项是 SpringMVC 和 Thymeleaf:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

## 3.控制器

为了简单起见，让我们使用一个只有一种方法的控制器来处理`GET`请求。

这通过向模型对象传递一个数组来响应，这将使它可被视图访问:

```java
@Controller
public class ThymeleafArrayController {

    @GetMapping("/arrays")
    public String arrayController(Model model) {
        String[] continents = {
          "Africa", "Antarctica", "Asia", "Australia", 
          "Europe", "North America", "Sourth America"
        };

        model.addAttribute("continents", continents);

        return "continents";
    }
}
```

## 4.景色

在视图页面中，我们将通过从上方的控制器传递的名称(大陆)来访问数组`continents`。

### 4.1.属性和索引

我们要检查的第一个属性是数组的长度。我们可以这样检查它:

```java
<p>...<span th:text="${continents.length}"></span>...</p>
```

查看上面来自视图页面的代码片段，我们应该注意到关键字`th:text`的使用。我们用它来打印花括号内的变量值，在本例中，是数组的长度。

因此，**我们通过索引访问数组`continents`中每个元素的值，就像我们在普通的 Java 代码**中所做的一样:

```java
<ol>
    <li th:text="${continents[2]}"></li>
    <li th:text="${continents[0]}"></li>
    <li th:text="${continents[4]}"></li>
    <li th:text="${continents[5]}"></li>
    <li th:text="${continents[6]}"></li>
    <li th:text="${continents[3]}"></li>
    <li th:text="${continents[1]}"></li>
</ol>
```

正如我们在上面的代码片段中看到的，每个元素都可以通过它的索引来访问。我们可以去[这里](/web/20221208143909/https://www.baeldung.com/spring-thymeleaf-3-expressions)了解更多关于 T 中的表情。

### 4.2.循环

类似地，**我们可以依次迭代数组中的元素**。

在百里香，我们可以这样做:

```java
<ul th:each="continet : ${continents}">
    <li th:text="${continent}"></li>
</ul>
```

当使用 **`th:each`关键字迭代数组**的元素时，我们并不仅限于使用列表标签。我们可以使用任何能够在页面上显示文本的 HTML 标签。例如:

```java
<h4 th:each="continent : ${continents}" th:text="${continent}"></h4>
```

在上面的代码片段中，每个元素都将显示在自己单独的`<h4></h4>`标签上。

### 4.3.效用函数

最后，我们将使用实用类函数来检查数组的其他一些属性。

让我们来看看这个:

```java
<p>The greatest <span th:text="${#arrays.length(continents)}"></span> continents.</p>

<p>Europe is a continent: <span th:text="${#arrays.contains(continents, 'Europe')}"></span>.</p>

<p>Array of continents is empty <span th:text="${#arrays.isEmpty(continents)}"></span>.</p>
```

我们首先查询数组的长度，然后检查`Europe`是否是数组`continents.`的元素

最后，我们检查数组`continents `是否为空。

## 5.结论

在本文中，我们学习了如何在 T hymeleaf 中使用一个数组，方法是检查它的长度并使用索引访问它的元素。我们还学习了如何在百里香叶中迭代它的元素。

最后，我们看到了使用效用函数来检查数组的其他属性。

和往常一样，本文的完整源代码可以在 Github 上找到[。](https://web.archive.org/web/20221208143909/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-thymeleaf-2)