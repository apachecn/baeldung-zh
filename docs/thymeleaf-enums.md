# 在百里香叶中使用枚举

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/thymeleaf-enums>

## 1.介绍

在这个快速教程中，我们将学习如何在[百里香叶](/web/20220728105348/https://www.baeldung.com/thymeleaf-in-spring-mvc)中使用枚举。

我们将从在下拉列表中列出枚举值开始。之后，我们将看看如何在模板中使用 enum 进行流控制。

## 2.设置

让我们首先将百里香的 [Spring Boot 发酵剂添加到我们的`pom.xml`文件中:](https://web.archive.org/web/20220728105348/https://search.maven.org/search?q=a:spring-boot-starter-thymeleaf%20AND%20g:org.springframework.boot)

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
    <versionId>2.2.2.RELEASE</versionId>
</dependency>
```

我们将使用有几种颜色选择的小部件，所以让我们定义我们的`Color`枚举:

```java
public enum Color {
    BLACK, BLUE, RED, YELLOW, GREEN, ORANGE, PURPLE, WHITE
}
```

现在，让我们创建我们的`Widget`类:

```java
public class Widget {
    private String name;
    private Color color;

    // Standard getters/setters
}
```

## 3.在下拉菜单中显示枚举

让我们使用[选择和选项](/web/20220728105348/https://www.baeldung.com/thymeleaf-select-option)来创建一个使用我们的`Color`枚举的下拉列表:

```java
<select name="color">
    <option th:each="colorOpt : ${T(com.baeldung.thymeleaf.model.Color).values()}" 
        th:value="${colorOpt}" th:text="${colorOpt}"></option>
</select>
```

**`T`操作符是 [Spring 表达式语言](/web/20220728105348/https://www.baeldung.com/spring-expression-language)** 的一部分，用于指定一个类的实例或访问静态方法。

如果我们运行我们的应用程序并导航到我们的小部件入口页面，我们将在`Color`下拉列表中看到我们所有的颜色:

[![entryenum dd1](img/109ba48a1c7c10c57d89d6b8602020c6.png)](/web/20220728105348/https://www.baeldung.com/wp-content/uploads/2019/07/entry_enum_dd1.jpg)

当我们提交表单时，**我们的`Widget`对象将用所选的颜色填充:**

[![view_1-1](img/8dd2d16391cad75c2628b5e23f2f1e5e.png)](/web/20220728105348/https://www.baeldung.com/wp-content/uploads/2019/07/view_1-1.jpg)

## 4.使用显示名称

因为所有的大写字母可能有点不协调，所以让我们扩展一下我们的例子，提供更多用户友好的下拉标签。

我们将首先修改我们的`Color`枚举来提供一个显示名称:

```java
public enum Color {
    BLACK("Black"), 
    BLUE("Blue"), 
    RED("Red"), 
    YELLOW("Yellow"), 
    GREEN("Green"),
    ORANGE("Orange"), 
    PURPLE("Purple"), 
    WHITE("White");

    private final String displayValue;

    private Color(String displayValue) {
        this.displayValue = displayValue;
    }

    public String getDisplayValue() {
        return displayValue;
    }
}
```

接下来，让我们来看看我们的百里香模板，并更新我们的下拉列表，以使用新的`displayValue`:

```java
<select name="color">
    <option th:each="colorOpt : ${T(com.baeldung.thymeleaf.model.Color).values()}" 
        th:value="${colorOpt}" th:text="${colorOpt.displayValue}"></option>
</select>
```

我们的下拉菜单现在显示更易读的颜色名称:

[![entry enum dd2](img/4e4594b6bfab5e7e91f3fbbc5fc9ec43.png)](/web/20220728105348/https://www.baeldung.com/wp-content/uploads/2019/07/entry_enum_dd2.jpg)

## 5.If 语句

有时，我们可能希望根据枚举的值来改变显示的内容。我们可以将我们的`Color`枚举与[百里香条件语句](/web/20220728105348/https://www.baeldung.com/spring-thymeleaf-conditionals)一起使用。

假设我们对一些可能的小部件颜色有自己的看法。

我们可以使用百里香叶`if`语句和我们的`Color`枚举来有条件地显示文本:

```java
<div th:if="${widget.color == T(com.baeldung.thymeleaf.model.Color).RED}">
    This color screams danger.
</div>
```

另一种选择是使用`String`比较:

```java
<div th:if="${widget.color.name() == 'GREEN'}">
    Green is for go.
</div>
```

## 6.Switch-Case 语句

**除了`if`语句，百里香还支持`switch-case`语句。**

让我们使用一个带有`Color`枚举的`switch-case`语句:

```java
<div th:switch="${widget.color}">
    <span th:case="${T(com.baeldung.thymeleaf.model.Color).RED}"
      style="color: red;">Alert</span>
    <span th:case="${T(com.baeldung.thymeleaf.model.Color).ORANGE}"
      style="color: orange;">Warning</span>
    <span th:case="${T(com.baeldung.thymeleaf.model.Color).YELLOW}"
      style="color: yellow;">Caution</span>
    <span th:case="${T(com.baeldung.thymeleaf.model.Color).GREEN}"
      style="color: green;">All Good</span>
</div>
```

如果我们输入一个橙色小部件，我们应该会看到我们的警告声明:

[![view_2](img/1a13c7c10098beed7640582c057d0fb5.png)](/web/20220728105348/https://www.baeldung.com/wp-content/uploads/2019/07/view_2.jpg)

## 7.结论

在本教程中，我们首先使用我们在应用程序中定义的`Color`枚举来创建一个下拉列表。从那里，我们学习了如何使下拉显示值更具可读性。

在用下拉菜单检查了输入端之后，我们继续学习输出端，学习如何在控制结构中使用枚举。我们同时使用了`if`和`switch-case`语句来根据我们的`Color`枚举条件化一些元素。

GitHub 上的[提供了完整的示例。](https://web.archive.org/web/20220728105348/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-thymeleaf-2)