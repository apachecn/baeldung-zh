# 在百里香叶中使用选择和选项

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/thymeleaf-select-option>

## 1。概述

百里香叶是与 Spring Boot 捆绑在一起的一个流行的模板引擎。我们已经发表了几篇关于它的文章，我们强烈推荐阅读 Baeldung 的百里香系列。

在本教程中，我们将看看如何在百里香叶中使用`select`和`option`标签。

## 2。HTML 基础知识

在 HTML 中，我们可以构建一个包含多个值的下拉列表:

```java
<select>
    <option value="apple">Apple</option>
    <option value="banana">Banana</option>
    <option value="orange">Orange</option>
    <option value="pear">Pear</option>
</select>
```

每个列表由`select`和嵌套的`option`标签组成。**默认情况下，网络浏览器会呈现一个带有第一个预选选项的列表**。

我们可以通过使用`selected`属性来控制选择哪个值:

```java
<option value="orange" selected>Orange</option>
```

此外，我们可以通过使用`disabled`属性来指定一个选项是不可选择的:

```java
<option disabled>Please select...</option>
```

## 3。百里香叶

在百里香叶中，我们可以使用`th:field`属性来绑定视图和模型:

```java
<select th:field="*{gender}">
    <option th:value="'M'" th:text="Male"></option>
    <option th:value="'F'" th:text="Female"></option>
</select>
```

虽然上面的例子不需要使用模板引擎，但在后面更高级的例子中，我们将看到百里香的威力。

### 3.1。`Option`没有选择

如果我们考虑一个有更多选项可供选择的场景，一个干净整洁的显示所有选项的方法是将`th:each`属性与`th:value`和`th:text`一起使用:

```java
<select th:field="*{percentage}">
    <option th:each="i : ${#numbers.sequence(0, 100)}" th:value="${i}" th:text="${i}">
    </option>
</select>
```

在上面的例子中，我们使用了从 0 到 100 的数字序列。我们将每个数字`i`的值分配给`option`标签的`value`属性，并且我们使用与显示值相同的数字。

百里香叶代码将在浏览器中呈现为:

```java
<select id="percentage" name="percentage">
    <option value="0">0</option>
    <option value="1">1</option>
    <option value="2">2</option>
    ...
    <option value="100">100</option>
</select>
```

**让我们把这个例子想象成`create`** ，也就是说，我们从一个新的表单开始，**百分比值不需要预先选择**。

### 3.2。选中`Option`

**假设我们现在想用一个`update`功能扩展我们的表单。在这种情况下，**，也就是说，我们回到先前创建的记录，我们想用现有数据填充表单，然后**需要选择选项**。

我们可以通过添加`th:selected`属性和一些条件来实现:

```java
<select th:field="*{percentage}">
    <option th:each="i : ${#numbers.sequence(0, 100)}" th:value="${i}" th:text="${i}" 
      th:selected="${i==75}"></option>
</select>
```

在上面的例子中，我们想通过检查`i`是否等于 75 来预选 75 的值。

然而，**这段代码不起作用，**呈现的 HTML 将是:

```java
<select id="percentage" name="percentage">
    <option value="0">0</option>
    ...
    <option value="74">74</option>
    <option value="75">75</option>
    <option value="76">76</option>
    ...
    <option value="100">100</option>
</select>
```

**要修复它，我们需要移除`th:field`并替换为`name`和`id`属性:**

```java
<select id="percentage" name="percentage">
```

最终，我们会得到:

```java
<select id="percentage" name="percentage">
    <option value="0">0</option>
    ...
    <option value="74">74</option>
    <option value="75" selected="selected">75</option>
    <option value="76">76</option>
    ...
    <option value="100">100</option>
</select>
```

## 4。用列表填充下拉列表

让我们看看如何在百里香中填充一个下拉列表。为此，我们将在控制器中创建一个`String`列表，并在视图中显示它。

首先，让我们用一个初始化字符串列表的方法创建一个控制器。其次，我们将使用模型属性来保存我们的列表，以便在视图中进行渲染。

```java
@RequestMapping(value = "/populateDropDownList", method = RequestMethod.GET) 
public String populateList(Model model) {
    List<String> options = new ArrayList<String>();
    options.add("option 1");
    options.add("option 2");
    options.add("option 3");
    model.addAttribute("options", options);
    return "dropDownList/dropDownList.html";
}
```

最后，我们可以引用我们的列表模型属性，并对其进行循环，以将每个列表元素显示为下拉列表的一个选项。

```java
<select class="form-control" id="dropDownList">
    <option value="0">select option</option>
    <option th:each="option : ${options}" th:value="${option}" th:text="${option}"></option>
</select>
```

## 5。结论

在这篇短文中，我们检查了如何在百里香中使用下拉/列表选择器。预选值有一个常见的缺陷，我们已经给出了解决方案。

和往常一样，讨论中使用的代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220728105348/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-thymeleaf-3)