# 百里香叶的 Spring 请求参数

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-thymeleaf-request-parameters>

## 1.介绍

在我们的文章[春天使用百里香简介](/web/20220728105348/https://www.baeldung.com/thymeleaf-in-spring-mvc)中，我们看到了如何将用户输入绑定到对象。

我们在百里香模板中使用了`th:object` 和 `th:field`，在控制器中使用了`@ModelAttribute`将数据绑定到一个 Java 对象。**在本文中，我们将看看如何结合百里香叶使用 Spring 注释`@RequestParam`。**

## 2.表单中的参数

让我们首先创建一个简单的控制器，它接受四个可选的请求参数:

```
@Controller
public class MainController {
    @RequestMapping("/")
    public String index(
        @RequestParam(value = "participant", required = false) String participant,
        @RequestParam(value = "country", required = false) String country,
        @RequestParam(value = "action", required = false) String action,
        @RequestParam(value = "id", required = false) Integer id,
        Model model
    ) {
        model.addAttribute("id", id);
        List<Integer> userIds = asList(1,2,3,4);
        model.addAttribute("userIds", userIds);
        return "index";
    }
}
```

我们的百里香模板的名字是`index.html`。**在接下来的三个部分中，我们将使用不同的 HTML 表单元素为用户向控制器传递数据。**

### 2.1.输入元素

首先，让我们创建一个简单的表单，带有一个文本输入字段和一个提交表单的按钮:

```
<form th:action="@{/}">
<input type="text" th:name="participant"/> 
<input type="submit"/> 
</form>
```

属性`th:name=”participant”`将输入字段的值绑定到控制器的参数`participant`。**要做到这一点，我们需要用`@RequestParam(value = “participant”)`来标注参数。**

### 2.2.选择元素

同样，对于 HTML select 元素:

```
<form th:action="@{/}">
    <input type="text" th:name="participant"/>
    <select th:name="country">
        <option value="de">Germany</option>
        <option value="nl">Netherlands</option>
        <option value="pl">Poland</option>
        <option value="lv">Latvia</option>
    </select>
</form>
```

所选选项的值被绑定到参数`country`，用`@RequestParam(value = “country”)`标注。

### 2.3.按钮元素

我们可以使用`th:name`的另一个元素是按钮元素:

```
<form th:action="@{/}">
    <button type="submit" th:name="action" th:value="in">check-in</button>
    <button type="submit" th:name="action" th:value="out">check-out</button>
</form>
```

根据是按下第一个按钮还是第二个按钮来提交表单，参数`action`的值将是`check-in`或`check-out`。

## 3.超链接中的参数

**将请求参数传递给控制器的另一种方式是通过超链接:**

```
<a th:href="@{/index}">
```

我们可以在括号中添加参数:

```
<a th:href="@{/index(param1='value1',param2='value2')}"> 
```

百里香叶对上述内容进行了评估:

```
<a href="/index?param1=value1&param2;=value2">
```

如果我们想要基于变量分配参数值，使用百里香叶表达式生成超链接特别有用。例如，让我们为每个用户 ID 生成一个超链接:

```
<th:block th:each="userId: ${userIds}">
    <a th:href="@{/(id=${userId})}"> User [[${userId}]]</a> <br/>
</th:block>
```

我们可以将用户 id 列表作为属性传递给模板:

```
List<Integer> userIds = asList(1,2,3);
model.addAttribute("userIds", userIds);
```

产生的 HTML 将是:

```
<a th:href="/?id=1"> User 1</a> <br/>
<a th:href="/?id=2"> User 2</a> <br/>
<a th:href="/?id=3"> User 3</a> <br/> 
```

超链接中的参数`id`被绑定到参数`id`，用`@RequestParam(value = “id”)`标注。

## 4.摘要

在这篇短文中，我们看到了如何结合百里香叶使用 Spring 请求参数。

首先，我们创建了一个接受请求参数的简单控制器。其次，我们看了如何使用百里香叶生成一个可以调用我们的控制器的 HTML 页面。

本文中所有示例的完整源代码可以在 GitHub 上找到。