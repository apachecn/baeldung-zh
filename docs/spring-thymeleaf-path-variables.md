# 百里香叶的春季路径变量

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-thymeleaf-path-variables>

## 1.介绍

在这个简短的教程中，我们将学习如何使用 Spring path 变量使用[百里香叶](/web/20220728105348/https://www.baeldung.com/thymeleaf-in-spring-mvc)创建 URL。

当我们想要传递一个值作为 URL 的一部分时，我们使用路径变量。在弹簧控制器中，我们使用 [`@PathVariable`](https://web.archive.org/web/20220728105348/https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/web/bind/annotation/PathVariable.html) 注释来访问这些值。

## 2.使用路径变量

首先，让我们通过创建一个简单的`Item`类来设置我们的示例:

```java
public class Item {
    private int id;
    private String name;

    // Constructor and standard getters and setters
}
```

现在，让我们创建我们的控制器:

```java
@Controller
public class PathVariablesController {

    @GetMapping("/pathvars")
    public String start(Model model) {
        List<Item> items = new ArrayList<Item>();
        items.add(new Item(1, "First Item"));
        items.add(new Item(2, "Second Item"));
        model.addAttribute("items", items);
        return "pathvariables/index";
    }

    @GetMapping("/pathvars/single/{id}")
    public String singlePathVariable(@PathVariable("id") int id, Model model) {
        if (id == 1) {
            model.addAttribute("item", new Item(1, "First Item"));
        } else {
            model.addAttribute("item", new Item(2, "Second Item"));
        }

        return "pathvariables/view";
    }
}
```

在我们的`index.html`模板中，让我们遍历我们的项目并创建调用`singlePathVariable`方法的链接:

```java
<div th:each="item : ${items}">
    <a th:href="@{/pathvars/single/{id}(id = ${item.id})}">
        <span th:text="${item.name}"></span>
    </a>
</div>
```

我们刚刚创建的代码生成了这样的 URL:

```java
http://localhost:8080/pathvars/single/1
```

这是在 URL 中使用表达式的标准百里香语法。

我们也可以使用串联来达到相同的结果:

```java
<div th:each="item : ${items}">
    <a th:href="@{'/pathvars/single/' + ${item.id}}">
        <span th:text="${item.name}"></span>
    </a>
</div>
```

## 3.使用多个路径变量

现在我们已经介绍了在百里香叶中创建路径变量 URL 的基础知识，让我们快速介绍使用多个。

首先，我们将创建一个`Detail`类，并修改我们的`Item`类以包含它们的列表:

```java
public class Detail {
    private int id;
    private String description;

    // constructor and standard getters and setters
}
```

接下来，我们来添加一个`Detail`到`Item`的列表:

```java
private List<Detail> details;
```

现在，让我们更新控制器，添加一个使用多个`@PathVariable`注释的方法:

```java
@GetMapping("/pathvars/item/{itemId}/detail/{dtlId}")
public String multiplePathVariable(@PathVariable("itemId") int itemId, 
  @PathVariable("dtlId") int dtlId, Model model) {
    for (Item item : items) {
        if (item.getId() == itemId) {
            model.addAttribute("item", item);
            for (Detail detail : item.getDetails()) {
                if (detail.getId() == dtlId) {
                    model.addAttribute("detail", detail);
                }
            }
        }
    }
    return "pathvariables/view";
}
```

最后，让我们修改我们的`index.html`模板，为每个详细记录创建 URL:

```java
<ul>
    <li th:each="detail : ${item.details}">
        <a th:href="@{/pathvars/item/{itemId}/detail/{dtlId}(itemId = ${item.id}, dtlId = ${dtl.id})}">
            <span th:text="${detail.description}"></span>
        </a>
    </li>
</ul>
```

## 4.结论

在这个快速教程中，我们学习了如何使用百里香创建带有路径变量的 URL。我们从创建一个只有一个 URL 的简单 URL 开始。后来，我们扩展了我们的例子，使用多个路径变量。

GitHub 上的[提供了示例代码。](https://web.archive.org/web/20220728105348/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-thymeleaf-2)