# 百里香叶中的条件句

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-thymeleaf-conditionals>

## 1.概观

在本教程中，我们将看看百里香中可用的**不同类型的条件句。**

关于百里香的快速介绍，请参考这篇[文章](/web/20221208143909/https://www.baeldung.com/thymeleaf-in-spring-mvc)。

## 2.Maven 依赖性

让我们从使用百里香叶和 Spring 所需的 Maven 依赖项开始:

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

对于其他 Spring 版本，我们应该使用匹配的`thymeleaf-springX `库，其中`X `代表 Spring 版本。另请注意，从`3.0.8.RELEASE`开始，**弹簧 5 由百里香叶支撑。**

所需依赖项的最新版本可以在[这里](https://web.archive.org/web/20221208143909/https://search.maven.org/classic/#search%7Cga%7C1%7Cthymeleaf)找到。

## 3.百里香叶条件句

我们必须区分允许我们根据条件在 HTML 元素中呈现文本的条件和控制 HTML 元素本身实例化的条件。

让我们定义我们将在本文中使用的`Teacher `模型类:

```java
public class Teacher implements Serializable {
    private String gender;
    private boolean isActive;
    private List<String> courses = new ArrayList<>();
    private String additionalSkills;
```

### 3.1.Elvis 操作员

Elvis 操作符`?: `允许我们根据变量的当前状态在 HTML 元素中呈现文本。

如果变量是`null`，我们可以使用默认表达式来提供默认文本:

```java
<td th:text="${teacher.additionalSkills} ?: 'UNKNOWN'" />
```

这里我们想要显示变量`teacher.additionalSkills`的内容(如果它被定义的话),并且我们想要文本“`UNKNOWN`”以其他方式呈现。

还可以根据布尔表达式显示任意文本:

```java
<td th:text="${teacher.active} ? 'ACTIVE' : 'RETIRED'" />
```

我们可以像前面的例子一样查询一个简单的布尔变量，但是字符串比较和范围检查也是可能的。

支持以下比较器及其文本表示:`> (gt)`、 `>= (ge)`、 `< (lt)`、 `<= (le)`、 `== (eq)`和`!= (ne)`。

### 3.2.如果–除非

`th:if` 和`th:unless`属性允许我们根据提供的条件呈现 HTML 元素:

```java
<td>
    <span th:if="${teacher.gender == 'F'}">Female</span>
    <span th:unless="${teacher.gender == 'F'}">Male</span>
</td>
```

如果`teacher.gender` 变量的内容等于一个`F`，则呈现值为`Female`的 span 元素。否则，渲染带有`Male`的元素。

这种设置类似于大多数编程语言中的`if-else `子句。

### 3.3.开关–外壳

如果一个表达式有两个以上的可能结果，我们可以使用`th:switch `和`th:case `属性对 HTML 元素进行有条件的呈现:

```java
<td th:switch="${#lists.size(teacher.courses)}">
    <span th:case="'0'">NO COURSES YET!</span>
    <span th:case="'1'" th:text="${teacher.courses[0]}"></span>
    <div th:case="*">
        <div th:each="course:${teacher.courses}" th:text="${course}"/>
    </div>
</td> 
```

根据`teacher.courses` 列表的大小，我们可以显示默认文本、单一课程或所有可用课程。我们使用星号(`*`)作为默认选项。

## 4.结论

在这篇短文中，我们研究了不同类型的百里香条件句，并给出了一些显示各种选项的简化示例。

这些例子可以在 GitHub 项目中找到。