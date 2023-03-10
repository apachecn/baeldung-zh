# 百里香叶迭代

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/thymeleaf-iteration>

## 1。概述

百里香叶是一个多功能的 Java 模板引擎，用于处理 XML、XHTML 和 HTML5 文档。

在这个快速教程中，我们将看看如何使用百里香叶以及该库提供的其他一些特性来执行迭代。

关于百里香的更多信息，请看我们的介绍文章[这里](/web/20221129014209/https://www.baeldung.com/thymeleaf-in-spring-mvc)。

## 2。Maven 依赖关系

为了创建这个例子，我们将使用 Spring 框架库和百里香叶库。

在这里我们可以看到我们的依赖关系([百里香叶](https://web.archive.org/web/20221129014209/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.thymeleaf%22%20AND%20a%3A%22thymeleaf%22)和[百里香叶弹簧](https://web.archive.org/web/20221129014209/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.thymeleaf%22%20AND%20a%3A%22thymeleaf-spring4%22)):

```java
<dependency>
    <groupId>org.thymeleaf</groupId>
    <artifactId>thymeleaf</artifactId>
    <version>3.0.11.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.thymeleaf</groupId>
    <artifactId>thymeleaf-spring5</artifactId>
    <version>3.0.11.RELEASE</version>
</dependency>
```

## 3。示例设置

在我们进入视图层之前，让我们为我们的例子创建 MVC 结构。

从模型层的代码片段开始:

```java
public class Student implements Serializable {
    private Integer id;
    private String name;
    // standard contructors, getters, and setters
}
```

让我们也提供负责加载模型并将其返回到视图层的控制器方法:

```java
@GetMapping("/listStudents")
public String listStudent(Model model) {
    model.addAttribute("students", StudentUtils.buildStudents());
    return "listStudents.html";
}
```

在我们上面的例子中，`buildStudents() `方法简单地返回一个`Student`对象的列表，然后我们将这些对象添加到`model`中。

## 4。T3`th:each`属性

在百里香叶中，迭代是通过使用`th:each `属性实现的。

关于这个属性有趣的事情之一是**它将接受和迭代一些不同的数据类型** `,` ，例如:

*   实现 `java.util.Iterable`的对象
*   实现`java.util.Map`的对象
*   数组
*   任何其他对象都被视为包含一个元素的单值列表

现在让我们用上面例子中设置的数据调用`th:each attribute `:

```java
<tr th:each="student: ${students}">
    <td th:text="${student.id}" />
    <td th:text="${student.name}" />
</tr>
```

代码片段显示了`th:each`对我们的列表`Students`的迭代。**使用`${} `符号**访问模型属性，列表中的每个元素通过`student `变量传递给循环体。

## 5。状态变量

百里香还通过状态变量启用了一个有用的机制来跟踪迭代过程。

状态变量提供以下属性:

*   **索引**:当前迭代索引，从 0(零)开始
*   **count** :目前已处理的元素数量
*   **size** :列表中元素的总数
*   **偶数/奇数**:检查当前迭代索引是偶数还是奇数
*   **first** :检查当前迭代是否是第一次迭代
*   **最后一次**:检查当前迭代是否是最后一次

让我们看看状态变量在我们的示例中是如何工作的:

```java
<tr 
  th:each="student, iStat : ${students}" 
  th:style="${iStat.odd}? 'font-weight: bold;'" 
  th:alt-title="${iStat.even}? 'even' : 'odd'">
    <td th:text="${student.id}" />
    <td th:text="${student.name}" />
</tr>
```

这里，我们包含了`iStat.odd` 属性来评估条件并为当前行设置粗体样式。下一次评估也是如此，但这次我们使用`iStat.even `通过 alt/title HTML 属性打印一个值。

如果我们省略状态变量的显式创建(在我们的例子中表示为`iStat`，**，我们可以通过简单地使用** `**studentStat**, `来调用我们的状态变量，它是带有后缀`Stat.`的变量`student`的集合

## 6。结论

在本文中，我们探索了百里香叶库提供的众多特性之一。

我们通过使用属性`th:each`及其现成的属性在百里香叶中展示了迭代。

本文中显示的代码的工作版本可以在我们的 [GitHub 库](https://web.archive.org/web/20221129014209/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-thymeleaf-4)中获得。