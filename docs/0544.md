# FreeMarker 通用操作

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/freemarker-operations>

## 1.介绍

FreeMarker 是一个模板引擎，用 Java 编写，由 Apache 基金会维护。我们可以使用 FreeMarker 模板语言，也称为 FTL，来生成许多基于文本的格式，如网页、电子邮件或 XML 文件。

在本教程中，我们将看到我们可以用 FreeMarker 做些什么，尽管注意到它的[可配置性很强](https://web.archive.org/web/20220625223349/https://freemarker.apache.org/docs/api/freemarker/template/Configuration.html)，甚至[可以很好地与 Spring](/web/20220625223349/https://www.baeldung.com/freemarker-in-spring-mvc-tutorial) 集成。

我们开始吧！

## 2.快速概述

为了在我们的页面中注入动态内容，我们需要**使用 FreeMarker 理解的语法**:

*   模板中的`${…}`将在生成的输出中被替换为花括号中表达式的实际值——我们称之为`interpolation –` ,例如`${1 + 2`和`${variableName}`
*   FTL 标签类似于 HTML 标签(但包含`#`或`@`), FreeMarker 解释它们，例如`<#if…></#if>`
*   FreeMarker 中的注释以`<#–`开始，以`–>`结束

## 3.包含标签

FTL 指令是我们在应用中遵循 DRY 原则的一种方式。我们将在一个文件中定义重复的内容，并使用单个`include`标签在不同的 FreeMarker 模板中重用它。

一个这样的用例是当我们想要在许多页面中包含菜单部分时。首先，我们将在一个文件中定义菜单部分——我们称之为`menu.ftl`——其内容如下:

```
<a href="#dashboard">Dashboard</a>
<a href="#newEndpoint">Add new endpoint</a>
```

在我们的 HTML 页面上，让我们包含创建的`menu.ftl`:

```
<!DOCTYPE html>
<html>
<body>
<#include 'fragments/menu.ftl'>
    <h6>Dashboard page</h6>
</body>
</html>
```

我们也可以把 FTL 包括在我们的片段中，这很好。

## 4.处理值存在

FTL 会将任何`null`值视为缺失值。因此，我们需要格外小心，**在我们的模板中添加逻辑来处理`null`** 。

我们可以使用`??`操作符来检查一个属性或者嵌套属性是否存在。结果是一个布尔值:

```
${attribute??}
```

因此，我们已经测试了`null,` 的属性，但这并不总是足够的。现在让我们定义一个默认值作为这个缺失值的后备。为此，我们需要将`!`操作符放在变量名称之后:

```
${attribute!'default value'}
```

使用圆括号，我们可以包装许多嵌套的属性。

例如，为了检查属性是否存在，以及是否有一个嵌套属性与另一个嵌套属性，我们包装所有内容:

```
${(attribute.nestedProperty.nestedProperty)??}
```

最后，将所有内容放在一起，我们可以将它们嵌入静态内容中:

```
<p>Testing is student property exists: ${student???c}</p>
<p>Using default value for missing student: ${student!'John Doe'}</p>
<p>Wrapping student nested properties: ${(student.address.street)???c}</p>
```

如果 `student` 是`null`，我们会看到:

```
<p>Testing is student property exists: false</p>
<p>Using default value for missing student: John Doe</p>
<p>Wrapping student nested properties: false</p>
```

请注意**在`??`** 后使用的附加`?c`指令。我们这样做是为了将布尔值转换成人类可读的字符串。

## 5.If-Else 标签

FreeMarker 中存在控制结构，传统的 if-else 可能很熟悉:

```
<#if condition>
    <!-- block to execute if condition is true -->
<#elseif condition2>
    <!-- block to execute if condition2 is the first true condition -->
<#elseif condition3>
    <!-- block to execute if condition3 is the first true condition -->
<#else>
    <!-- block to execute if no condition is true -->
</#if>
```

虽然`elseif`和`else`分支是可选的，但是条件必须解析为布尔值。

为了帮助我们进行评估，我们可能会使用以下方法之一:

*   `x == y`查的是`x`等于`y`
*   `x != y`仅当`x`不同于`y`时返回`true`
*   `x lt y`意味着`x`必须严格小于`y` ——我们也可以用`<`代替`lt`
*   只有当`x`严格大于`y`时,`x gt y`才计算为`true`——我们可以用`>`代替`gt`
*   `x lte y`测试`x`是否小于或等于`y`—`lte`的替代选项是`<=`
*   `x gte y`测试`x`是否大于或等于`y`—`gte`的备选项为> =
*   `x??`检查`x`的存在
*   `sequence?seqContains(x)`验证序列中`x`的存在

记住 **FreeMarker 将> =和>视为 FTL 标签的结束字符是非常重要的。**解决方法是将它们的用法用括号括起来，或者**用`gte`或`gt`代替。**

将它放在一起，用于以下模板:

```
<#if status??>
    <p>${status.reason}</p>
<#else>
    <p>Missing status!</p>
</#if>
```

我们最终得到的 HTML 代码是:

```
 <!-- When status attribute exists -->
<p>404 Not Found</p>

<!-- When status attribute is missing -->
<p>Missing status!</p>
```

## 6.子变量的容器

在 FreeMarker 中，我们有三种类型的子变量容器:

*   是一系列的键-值对——键在散列中必须是唯一的，我们没有排序
*   `Sequences` 是列表，其中我们有一个与每个值相关联的索引——一个值得注意的事实是子变量可以是不同的类型
*   `Collections` 是序列的一个特例，我们不能通过索引访问大小或检索值——但是我们仍然可以用`list`标签迭代它们！

### 6.1.迭代项目

我们可以用两种基本方法迭代一个容器。第一个是迭代每个值，并为每个值设定逻辑:

```
<#list sequence as item>
    <!-- do something with ${item} -->
</#list>
```

或者，当我们想要迭代一个`Hash`，访问键和值:

```
<#list hash as key, value>
    <!-- do something with ${key} and ${value} -->
</#list>
```

第二种形式更强大，因为它还允许我们定义在迭代的各个步骤中应该发生的逻辑:

```
<#list sequence>
    <!-- one-time logic if the sequence is not empty -->
    <#items as item>
        <!-- logic repeated for every item in sequence -->
    </#items>
    <!-- one-time logic if the sequence is not empty -->
<#else>
    <!-- one-time logic if the sequence is empty -->
</#list>
```

`item`代表循环变量的名称，但是我们可以根据需要将其重命名。`else`分支是可选的。

作为一个实践示例，我们将定义一个模板，在其中列出一些状态:

```
<#list statuses>
    <ul>
    <#items as status>
        <li>${status}</li>
    </#items>
    </ul>
<#else>
    <p>No statuses available</p>
</#list>
```

当我们的容器为`[“200 OK”, “404 Not Found”, “500 Internal Server Error”]`时，这将返回以下 HTML:

```
<ul>
<li>200 OK</li>
<li>404 Not Found</li>
<li>500 Internal Server Error</li>
</ul>
```

### 6.2.物品处理

hash 允许我们使用两个简单的函数:`keys`只检索包含的键，而`values`只检索值。

一个序列更复杂；我们可以对最有用的功能进行分组:

*   `chunk`和`join`得到一个子序列或组合两个序列
*   用于修改元素顺序的`reverse`、`sort,` 和`sortBy`
*   `first`和`last`将分别检索第一个或最后一个元素
*   `size`表示序列中元素的数量
*   `seqContains`、`seqIndexOf`或`seqLastIndexOf`来查找元素

## 7.类型处理

FreeMarker 附带了大量用于处理对象的函数(内置的)。我们来看看一些常用的函数。

### 7.1.字符串处理

*   `url` 和`urlPath` 将对字符串进行 URL 转义，除了`urlPath`不会对斜杠/进行转义
*   `jString`、`jsString,` 和`jsonString` 将分别应用 Java、Javascript 和 JSON 的转义规则
*   `capFirst`、`uncapFirst`、`upperCase`、`lowerCase` 和`capitalize` 对改变字符串的大小写很有用，正如它们的名字所暗示的那样
*   `boolean`、`date`、`time`、`datetime` 、`number`是将字符串转换成其他类型的函数

现在让我们使用其中的几个函数:

```
<p>${'http://myurl.com/?search=Hello World'?urlPath}</p>
<p>${'Using " in text'?jsString}</p>
<p>${'my value?upperCase}</p>
<p>${'2019-01-12'?date('yyyy-MM-dd')}</p>
```

上面模板的输出将是:

```
<p>http%3A//myurl.com/%3Fsearch%3DHello%20World</p>
<p>MY VALUE</p>
<p>Using \" in text</p>
<p>12.01.2019</p>
```

当使用`date`函数时，我们还传递了用于解析字符串对象的模式。 **FreeMarker 使用本地格式，除非另有说明**，例如在可用于日期对象的`string`函数中。

### 7.2.数字处理

*   `round`、`floor` 和`ceiling` 可以帮助舍入数字
*   `abs` 将返回一个数字的绝对值
*   `string` 将数字转换成字符串。我们还可以传递四种预定义的数字格式:`computer`、`currency`、`number`或`percent`，或者定义自己的格式，如`[ “0.###” ]`

让我们做一连串的数学运算:

```
<p>${(7.3?round + 3.4?ceiling + 0.1234)?string('0.##')}</p>
<!-- (7 + 4 + 0.1234) with 2 decimals -->
```

不出所料，结果值是`11.12.`

### 7.3.日期处理

*   `.now`代表当前日期时间
*   `date`、`time` 和`datetime` 可以返回日期时间对象的日期和时间部分
*   会将日期时间转换为字符串——我们也可以传递所需的格式或使用预定义的格式

我们现在将获取当前时间，并将输出格式化为仅包含小时和分钟的字符串:

```
<p>${.now?time?string('HH:mm')}</p>
```

生成的 HTML 将是:

```
<p>15:39</p>
```

## 8.异常处理

我们将看到两种处理 FreeMarker 模板异常的方法。

第一种方法是使用`attempt-recover`标签来定义我们应该尝试执行什么，以及在出错时应该执行的代码块。

语法是:

```
<#attempt>
    <!-- block to try -->
<#recover>
    <!-- block to execute in case of exception -->
</#attempt>
```

`attempt`和`recover`标签都是强制性的。**如果出现错误，它将回滚尝试的块，并将只执行`recover`段**中的代码。

记住这个语法，让我们将模板定义为:

```
<p>Preparing to evaluate</p>
<#attempt>
    <p>Attribute is ${attributeWithPossibleValue??}</p>
<#recover>
    <p>Attribute is missing</p>
</#attempt>
<p>Done with the evaluation</p>
```

当`attributeWithPossibleValue`缺失时，我们会看到:

```
<p>Preparing to evaluate</p>
    <p>Attribute is missing</p>
<p>Done with the evaluation</p>
```

当`attributeWithPossibleValue`存在时的输出为:

```
<p>Preparing to evaluate</p>
    <p>Attribute is 200 OK</p>
<p>Done with the evaluation</p>
```

第二种方法是配置 FreeMarker 在异常情况下应该发生什么。

有了 Spring Boot，我们可以通过属性文件轻松地进行配置；以下是一些可用的配置:

*   `spring.freemarker.setting.template_exception_handler=rethrow`再次抛出异常
*   `spring.freemarker.setting.template_exception_handler=debug`向客户端输出堆栈跟踪信息，然后重新抛出异常。
*   `spring.freemarker.setting.template_exception_handler=html_debug`将堆栈跟踪信息输出到客户端，对其进行格式化，使其在浏览器中易于阅读，然后再次抛出异常。
*   跳过失败的指令，让模板继续执行。
*   `spring.freemarker.setting.template_exception_handler=default`

## 9.调用方法

有时我们想从 FreeMarker 模板中调用 Java 方法。我们现在来看看如何去做。

### 9.1.静态成员

要开始访问静态成员，我们可以更新我们的全局 FreeMarker 配置，或者在模型上添加一个 S `taticModels`类型属性，在属性名`statics`下:

```
model.addAttribute("statics", new DefaultObjectWrapperBuilder(new Version("2.3.28"))
    .build().getStaticModels());
```

访问静态元素非常简单。

首先，我们使用 assign 标记导入类的静态元素，然后决定名称，最后是 Java 类路径。

下面是我们如何在模板中导入`Math`类，显示静态`PI`字段的值，并使用静态`pow`方法:

```
<#assign MathUtils=statics['java.lang.Math']>
<p>PI value: ${MathUtils.PI}</p>
<p>2*10 is: ${MathUtils.pow(2, 10)}</p>
```

产生的 HTML 是:

```
<p>PI value: 3.142</p>
<p>2*10 is: 1,024</p>
```

### 9.2.Bean 成员

Bean 成员非常容易访问:**使用点号(。)**就这样！

对于我们的下一个例子，我们将添加一个`Random`对象到我们的模型:

```
model.addAttribute("random", new Random());
```

在我们的 FreeMarker 模板中，让我们生成一个随机数:

```
<p>Random value: ${random.nextInt()}</p>
```

这将导致类似于以下内容的输出:

```
<p>Random value: 1,329,970,768</p>
```

### 9.3.自定义方法

添加自定义方法的第一步是拥有一个实现 FreeMarker 的`TemplateMethodModelEx`接口并在`exec`方法中定义我们的逻辑的类:

```
public class LastCharMethod implements TemplateMethodModelEx {
    public Object exec(List arguments) throws TemplateModelException {
        if (arguments.size() != 1 || StringUtils.isEmpty(arguments.get(0)))
            throw new TemplateModelException("Wrong arguments!");
        String argument = arguments.get(0).toString();
        return argument.charAt(argument.length() - 1);
    }
}
```

我们将添加一个新类的实例作为模型的属性:

```
model.addAttribute("lastChar", new LastCharMethod());
```

下一步是在模板中使用我们的新方法:

```
<p>Last char example: ${lastChar('mystring')}</p>
```

最后，得到的输出是:

```
<p>Last char example: g</p>
```

## 10.结论

在本文中，我们看到了如何在我们的项目中使用 FreeMarker 模板引擎。我们已经关注了常见的操作，如何操作不同的对象，以及一些更高级的主题。

GitHub 上的[提供了所有这些代码片段的实现。](https://web.archive.org/web/20220625223349/https://github.com/eugenp/tutorials/tree/master/spring-freemarker)