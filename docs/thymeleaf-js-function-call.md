# 用百里香叶进行 JavaScript 函数调用

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/thymeleaf-js-function-call>

## 1.概观

在本教程中，我们将在[百里香](/web/20220813062744/https://www.baeldung.com/thymeleaf-in-spring-mvc)模板中调用 [JavaScript](https://web.archive.org/web/20220813062744/https://developer.mozilla.org/en-US/docs/Web/JavaScript) 函数。

我们将从设置依赖项开始。然后，我们将添加我们的 Spring 控制器和百里香模板。最后，我们将展示基于输入调用 JavaScript 函数的方法。

## 2.设置

为了在我们的应用程序中使用百里香叶，让我们将[百里香叶弹簧 5](https://web.archive.org/web/20220813062744/https://search.maven.org/search?q=a:thymeleaf-spring5%20AND%20g:org.thymeleaf) 依赖项添加到我们的 Maven 配置中:

```java
<dependency>
    <groupId>org.thymeleaf</groupId>
    <artifactId>thymeleaf-spring5</artifactId>
    <version>3.0.11.RELEASE</version>
</dependency>
```

然后，让我们基于我们的 [`Student`](/web/20220813062744/https://www.baeldung.com/thymeleaf-in-spring-mvc#2-collection-attributes) 模型将它添加到我们的弹簧控制器中:

```java
@Controller
public class FunctionCallController {

    @RequestMapping(value = "/function-call", method = RequestMethod.GET)
    public String getExampleHTML(Model model) {
        model.addAttribute("totalStudents", StudentUtils.buildStudents().size());
        model.addAttribute("student", StudentUtils.buildStudents().get(0));
        return "functionCall.html";
    }
}
```

最后，我们将这两个 JavaScript 函数添加到`src/main/webapp/WEB-INF/views`下的`functionCall.html`模板中:

```java
<script  th:inline="javascript">
    function greetWorld() {
        alert("hello world")
    }

    function salute(name) {
        alert("hello: " + name)
    }
</script>
```

在下一节中，我们将使用这两个函数来说明我们的例子。

如果有任何麻烦，我们可以随时查看[如何在百里香](/web/20220813062744/https://www.baeldung.com/spring-thymeleaf-css-js)中添加 JavaScript。

## 3.在百里香叶内部调用 JavaScript 函数

### 3.1.使用没有输入的函数

下面是我们如何调用上面的`greetWorld`函数:

```java
<button th:onclick="greetWorld()">using no variable</button>
```

它适用于任何自定义或内置的 JavaScript 函数。

### 3.2.使用具有静态输入的函数

如果我们在 JavaScript 函数中不需要任何动态变量，这就是如何调用它:

```java
<button th:onclick="'alert(\'static variable used here.\');'">using static variable</button> 
```

这避开了单引号，并且不需要 [SpringEL](/web/20220813062744/https://www.baeldung.com/spring-expression-language) 。

### 3.3.使用具有动态输入的函数

使用变量调用 JavaScript 函数有四种方式。

插入变量的第一种方法是使用行内变量:

```java
<button th:onclick="'alert(\'There are exactly '  + ${totalStudents} +  ' students\');'">using inline dynamic variable</button> 
```

另一个选择是通过调用`javascript:function`:

```java
<button th:onclick="'javascript:alert(\'There are exactly ' + ${totalStudents} + ' students\');'">using javascript:function</button>
```

第三种方式是使用[数据属性](/web/20220813062744/https://www.baeldung.com/thymeleaf-custom-html-attributes):

```java
<button th:data-name="${student.name}" th:onclick="salute(this.getAttribute('data-name'))">using data attribute</button> 
```

**这个方法在调用 [JavaScript 事件](https://web.archive.org/web/20220813062744/https://developer.mozilla.org/en-US/docs/Web/Events)时很方便，比如`onClick`和`onLoad`。**

最后，我们可以用[双方括号语法](https://web.archive.org/web/20220813062744/https://www.thymeleaf.org/doc/tutorials/3.0/usingthymeleaf.html#expression-inlining)调用我们的`salute`函数:

```java
<button th:onclick="salute([[${student.name}]])">using double brackets</button>
```

**在百里香**中，双方括号之间的表达式被视为内嵌表达式。这就是为什么我们可以使用在`th:text` 属性中也有效的任何类型的表达式。

## 4.结论

在本教程中，我们学习了如何调用模板中的 JavaScript 函数。我们从设置依赖项开始。然后，我们构造了控制器和模板。最后，我们探索了在百里香叶中调用任何 JavaScript 函数的方法。

和往常一样，代码可以在 GitHub 的[上获得。](https://web.archive.org/web/20220813062744/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-thymeleaf)