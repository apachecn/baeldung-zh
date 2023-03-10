# JSTL 图书馆指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jstl>

## 1。概述

JavaServer Pages 标记库(JSTL)是一组标记，可用于实现一些常见操作，如循环、条件格式等。

在本教程中，我们将讨论如何设置 JSTL 和如何使用它的众多标签。

## 2。设置

为了启用 JSTL 特性，我们必须将库添加到我们的项目中。对于 Maven 项目，我们在`pom.xml`文件中添加依赖关系:

```java
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>jstl</artifactId>
    <version>1.2</version>
</dependency>
```

将库添加到我们的项目后，最后的设置将是使用 taglib 指令将核心 JSTL 标记和任何其他标记的名称空间文件添加到我们的 JSP 中，如下所示:

```java
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
```

接下来，我们将看看这些标签，它们大致分为五类。

## 3。核心标签

JSTL 核心标签库包含用于执行基本操作的标签，如打印值、变量声明、异常处理、执行迭代、声明条件语句等。

让我们来看看核心标签。

### 3.1。`<c:out>`标签

**`<c:out>`用于显示变量中包含的值或隐式表达式的结果。**

它有三个属性:`value, default, and escapeXML.``escapeXML`属性输出包含在`value`属性或其附件中的原始 XML 标签。

`<c:out>`标签的一个例子是:

```java
<c:out value="${pageTitle}"/>
```

### 3.2。`<c:set>`标签

`<c:set>`标签用于声明 JSP 中的作用域变量。我们还可以分别在`var`和`value`属性中声明变量的名称和值。

一个示例的形式如下:

```java
<c:set value="JSTL Core Tags Example" var="pageTitle"/>
```

### 3.3。`<c:remove>`标签

`<c:remove>`标签删除了限定了作用域的变量，这相当于将`null`赋给了一个变量。它采用了`var`和`scope`属性，其中`scope`具有所有范围的默认值。

下面，我们展示了一个使用`<c:remove>` 标签的例子:

```java
<c:remove var="pageTitle"/>
```

### 3.4。`<c:catch>`标签

`<c:catch>`标签捕捉其外壳内抛出的任何异常。如果抛出异常，它的值存储在这个标签的`var`属性中。

典型用法如下:

```java
<c:catch var ="exceptionThrown">
    <% int x = Integer.valueOf("a");%>
</c:catch>
```

为了检查是否抛出了异常，我们使用了如下所示的`<c:if>`标记:

```java
<c:if test = "${exceptionThrown != null}">
    <p>The exception is : ${exceptionThrown} <br />
      There is an exception: ${exceptionThrown.message}
    </p>
</c:if>
```

### 3.5。`<c:if>`标签

`<c:if>`是一个条件标记，只有当它的`test`属性评估为真时，它才显示或执行它所包含的 scriptlets。评估的结果可以存储在它的`var`属性中。

### 3.6。`<c:choose>`、`<c:when>`和`<c:otherwise>`标签

`<c:choose>`是一个父标签，用于执行类似开关或 if-else 表达式。它有两个子标签；`<c:when>` 和 `<c:otherwise>`分别代表 if/else-if 和 else。

`<c:when>`接受一个`test`属性，该属性保存要计算的表达式。下面，我们展示了这些标签的用法示例:

```java
<c:set value="<%= Calendar.getInstance().get(Calendar.SECOND)%>" var="seconds"/>
<c:choose>
    <c:when test="${seconds le 30 }">
        <c:out value="${seconds} is less than 30"/>
    </c:when>
    <c:when test="${seconds eq 30 }">
        <c:out value="${seconds} is equal to 30"/>
    </c:when>
    <c:otherwise>
        <c:out value="${seconds} is greater than 30"/>
    </c:otherwise>
</c:choose>
```

### 3.7。`<c:import>`标签

`<c:import>`标签处理从绝对或相对 URL 获取和公开内容。

我们可以使用`url`和`var`属性分别保存 URL 和从 URL 获取的内容。例如，我们可以通过以下方式从 URL 导入内容:

```java
<c:import var = "data" url = "http://www.example.com"/>
```

### 3.8。`<c:forEach>`标签

`<c:forEach>`标签类似于 Java 的 for、while 或 do-while 语法。`items`属性保存要迭代的条目列表，而`begin`和`end`属性分别保存起始和结束索引(零索引)。

`<c:forEach>`标签还有一个`step`属性，控制每次迭代后索引增量的大小。下面，我们展示一个使用示例:

```java
<c:forEach var = "i" items="1,4,5,6,7,8,9">
    Item <c:out value = "No. ${i}"/><p>
</c:forEach>
```

### 3.9。`<c:forTokens>`标签

`<c:forTokens>`标签用于将一个`String`拆分成令牌并遍历它们。

类似于`<c:forEach>`标签，它有一个`items` 属性和一个附加的`delim`属性，该属性是`String`的分隔符，如下所示:

```java
<c:forTokens 
  items = "Patrick:Wilson:Ibrahima:Chris" 
  delims = ":" var = "name">
    <c:out value = "Name: ${name}"/><p>
</c:forTokens>
```

### 3.10。`<c:url>`和`<c:param>`标签

`<c:url>`标签对于用正确的请求编码格式化 URL 很有用。格式化后的 URL 存储在`var`属性中。

`<c:url>`标签还有一个`<c:param>`子标签，用于指定 URL 参数。我们在下面展示一个例子:

```java
<c:url value = "/core_tags" var = "myURL">
    <c:param name = "parameter_1" value = "1234"/>
    <c:param name = "parameter_2" value = "abcd"/>
</c:url>
```

### 3.11。`<c:redirect>`标签

`<c:redirect>`标签执行 URL 重写并将用户重定向到其`url`属性中指定的页面。典型的用例如下所示:

```java
<c:redirect url="/core_tags"/>
```

## 4。格式化标签

**JSTL 格式化标签库提供了一种方便的方式来格式化文本、数字、日期、时间和其他变量，以便更好地显示。**

JSTL 格式标签也可以用来增强网站的国际化。

在使用这些格式化标签之前，我们必须将标签库添加到 JSP 中:

```java
<%@ taglib prefix="fmt" uri="http://java.sun.com/jsp/jstl/fmt" %>
```

让我们来识别各种格式标签以及如何使用它们。

### 4.1。`<fmt:formatDate>`标签

标签在格式化日期或时间时很有用。`value` 属性保存要格式化的日期，`type`属性取三个值之一；`date, time or both`。

`<fmt:formatDate>`还有一个`pattern`属性，我们可以在其中指定想要的格式模式。下面是其中一种模式的示例:

```java
<c:set var="now" value="<%= new java.util.Date()%>"/>
<fmt:formatDate type="time" value="${now}"/>
```

### 4.2。`<fmt:parseDate>`标签

`<fmt:parseDate>`标签类似于`<fmt:formatDate>` 标签。

**不同之处在于，使用`<fmt:parseDate>`标签，我们可以指定底层日期解析器应该期望的日期值的格式模式。**

我们可以解析日期:

```java
<c:set var="today" value="28-03-2018"/>
<fmt:parseDate value="${today}" var="parsedDate" pattern="dd-MM-yyyy"/>
```

### 4.3。`<fmt:formatNumber>`标签

`<fmt:formatNumber>`标签以特定的模式或精度处理数字的呈现，该模式或精度可以是在其`type`属性中指定的`number, currency or percentage`之一。`<fmt:formatNumber>`的用法示例如下:

```java
<c:set var="fee" value="35050.1067"/>
<fmt:formatNumber value="${fee}" type="currency"/>
```

### 4.4。`<fmt:parseNumber>`标签

`<fmt:parseNumber>`标签类似于`<fmt:formatNumber>`标签。不同之处在于，使用`<fmt:parseNumber>`标签，我们可以指定底层数字解析器应该期望数字采用的格式模式。

我们可以这样使用它:

```java
<fmt:parseNumber var="i" type="number" value="${fee}"/>
```

### 4.5。`<fmt:bundle>`标签

`<fmt:bundle>` 标签是`<fmt:message>`标签的父标签。`<fmt:bundle>` 将在其`basename`属性中指定的包添加到括起来的`<fmt:message>`标签中。

标签对于启用国际化很有用，因为我们可以指定特定于地区的对象。典型的用法是:

```java
<fmt:bundle basename="com.baeldung.jstl.bundles.CustomMessage" prefix="verb.">
    <fmt:message key="go"/><br/>
    <fmt:message key="come"/><br/>
    <fmt:message key="sit"/><br/>
    <fmt:message key="stand"/><br/>
</fmt:bundle>
```

### 4.6。`<fmt:setBundle>`标签

`<fmt:setBundle>`标签用于在 JSP 中加载资源包，并使其在整个页面中可用。加载的资源包存储在`<fmt:setBundle>`标签的`var`属性中。我们可以通过以下方式设置捆绑包:

```java
<fmt:setBundle basename="com.baeldung.jstl.bundles.CustomMessage" var="lang"/>
```

### 4.7。`<fmt:setLocale>`标签

`<fmt:setLocale>`标签用于设置 JSP 中放在声明之后的部分的区域设置。通常，我们会通过以下方式进行设置:

```java
<fmt:setLocale value="fr_FR"/>
```

fr_FR 表示地区，在本例中是法语。

### 4.8。`<fmt:timeZone>`标签

`<fmt:timeZone>`标签是一个父标签，它指定其附件中标签的任何时间格式化或解析操作所使用的时区。

该时区参数由其`value`属性提供。下面显示了一个用法示例:

```java
<fmt:timeZone value="${zone}">
    <fmt:formatDate value="${now}" timeZone="${zn}" 
      type="both"/>
</fmt:timeZone>
```

### 4.9。`<fmt:setTimeZone>`标签

`<fmt:setTimeZone>`标签可用于将在其`value`属性中指定的时区复制到在其`var`属性中指定的作用域变量中。我们对此的定义是:

```java
<fmt:setTimeZone value="GMT+9"/>
```

### 4.10。`<fmt:message>`标签

`<fmt:message` **`>`** 标签用于显示国际化消息。要检索的消息的唯一标识符应该传递给它的`key`属性。

用于查找消息的特定包，也可以通过`bundle`属性指定。

这可能看起来像这样:

```java
<fmt:setBundle basename = "com.baeldung.jstl.bundles.CustomMessage" var = "lang"/>
<fmt:message key="verb.go" bundle="${lang}"/>
```

### 4.11。`<fmt:requestEncoding>`标签

标签`<fmt:requestEncoding>`在为动作类型为`post`的表单指定编码类型时非常有用。

要使用的字符编码的名称通过`<fmt:requestEncoding>`标签的`key`属性提供。

让我们看下面的例子:

```java
<fmt:requestEncoding value = "UTF-8" />
```

## 5。XML 标签

JSTL XML 标记库提供了在 JSP 中与 XML 数据交互的便捷方式。

为了能够访问这些 XML 标记，我们将通过以下方式将标记库添加到 JSP 中:

```java
<%@ taglib prefix="x" uri="http://java.sun.com/jsp/jstl/xml" %>
```

让我们看看 JSTL XML 标签库中的不同标签。

### 5.1。`<x:out>`标签

`<x:out>`标签类似于 JSP 中的`<%= %>` scriptlet 标签，但是`<x:out>`专门用于 XPath 表达式。

`<x:out>`标签有一个`select`和`escapeXML`属性，分别用于指定 XPath 表达式来计算一个`String`和启用特殊 XML 字符的转义。

一个简单的例子是:

```java
<x:out select="$output/items/item[1]/name"/>
```

上面的`$output`指的是一个预加载的 XSL 文件。

### 5.2。`<x:parse>`标签

`<x:parse>`标签用于解析其 *xml* 或`doc` 属性或附件中指定的 XML 数据。一个典型的例子是:

```java
<x:parse xml="${xmltext}" var="output"/>
```

### 5.3。`<x:set>`标签

`<x:set>`标签将在其`var`属性中指定的变量设置为传递给其`select`属性的 XPath 表达式。一个典型的例子是:

```java
<x:set var="fragment" select="$output//item"/>
```

### 5.4。`<x:if>`标签

如果提供给标签`select`属性的 XPath 表达式的值为 true，标签`<x:if>`就会处理它的主体。

评估的结果可以存储在它的`var`属性中。

一个简单的用例如下所示:

```java
<x:if select="$output//item">
    Document has at least one <item> element.
</x:if>
```

### 5.5。`<x:forEach>`标签

标签用于循环 XML 文档中的节点。XML 文档通过`<x:forEach>`标签的`select`属性提供。

就像`<c:forEach>`核心标签一样，`<x:forEach>`标签也有`begin, end` 和`step`属性。

因此，我们会有:

```java
<ul class="items">
    <x:forEach select="$output/items/item/name" var="item">
        <li>Item Name: <x:out select="$item"/></li>
    </x:forEach>
</ul>
```

### 5.6。`<x:choose>`、`<x:when>`和`<x:otherwise>`标签

`<x:choose>`标签是一个父标签，用于执行类似开关或 if/else-if/else 表达式，没有属性，但包含了`<x:when>` 和`<x:otherwise>`标签。

`<x:when>` 标签类似于 if/else-if，并带有一个保存待计算表达式的`select`属性。

`<x:otherwise>`标签类似于 else/default 子句，没有属性。

下面，我们展示一个示例用例:

```java
<x:choose>
    <x:when select="$output//item/category = 'Sneakers'">
        Item category is Sneakers
    </x:when>
    <x:when select="$output//item/category = 'Heels'">
        Item category is Heels
    </x:when>
    <x:otherwise>
       Unknown category.
    </x:otherwise>
</x:choose>
```

### 5.7。`<x:transform>`和`<x:param>`标签

`<x:transform>`标签通过对 XML 文档应用可扩展样式表语言(XSL)来转换 JSP 中的 XML 文档。

要转换的 XML 文档或`String`被提供给`doc`属性，而要应用的 XSL 被传递给`<x:transform>` 标签的`xslt`属性。

`<x:param>`标签是`<x:transform>`标签的子标签，用于设置转换样式表中的参数。

一个简单的用例将采用以下形式:

```java
<c:import url="/items_xml" var="xslt"/>
<x:transform xml="${xmltext}" xslt="${xslt}">
    <x:param name="bgColor" value="blue"/>
</x:transform>
```

## 6。SQL 标签

**JSTL SQL 标签库为执行关系数据库操作** s 提供标签

为了启用 JSTL SQL 标记，我们将标记库添加到 JSP 中:

```java
<%@ taglib prefix="sql" uri="http://java.sun.com/jsp/jstl/sql" %>
```

JSTL SQL 标签支持不同的数据库，包括 MySQL，Oracle 和微软 SQL Server。

接下来，我们将看看不同的可用 SQL 标记。

### 6.1。`<sql:setDataSource>`标签

`<sql:setDataSource>`标签用于定义 JDBC 配置变量。

这些配置变量保存在`<sql:setDataSource>`标签的`driver, url, user, password` 和 `dataSource` 属性中，如下所示:

```java
<sql:setDataSource var="dataSource" driver="com.mysql.cj.jdbc.Driver"
  url="jdbc:mysql://localhost/test" user="root" password=""/>
```

在上面的示例中，`var`属性保存了一个标识相关数据库的值。

### 6.2。`<sql:query>`标签

`<sql:query>`标签用于执行 SQL SELECT 语句，其结果存储在`var`属性中定义的作用域变量中。通常，我们将其定义为:

```java
<sql:query dataSource="${dataSource}" var="result">
    SELECT * from USERS;
</sql:query>
```

`<sql:query>`标签的`sql`属性保存要执行的 SQL 命令。其他属性包括`maxRows`、`startRow`和`dataSource.`

### 6.3。`<sql:update>`标签

`<sql:update>`标签类似于`<sql:query>`标签，但是只执行不需要返回值的 SQL 插入、更新或删除操作。

一个使用示例是:

```java
<sql:update dataSource="${dataSource}" var="count">
    INSERT INTO USERS(first_name, last_name, email) VALUES
      ('Grace', 'Adams', '[[email protected]](/web/20220627172721/https://www.baeldung.com/cdn-cgi/l/email-protection)');
</sql:update>
```

`<sql:update>`标签的`var`属性包含受其`sql`属性中指定的 SQL 语句影响的行数。

### 6.4。`<sql:param>`标签

`<sql:param>`标签是一个子标签，可以在`<sql:query>`或`<sql:update>`标签中使用，为 sql 语句中的值占位符提供值，如下所示:

```java
<sql:update dataSource = "${dataSource}" var = "count">
    DELETE FROM USERS WHERE email = ?
    <sql:param value = "[[email protected]](/web/20220627172721/https://www.baeldung.com/cdn-cgi/l/email-protection)" />
</sql:update>
```

`<sql:param>`标签有单一属性；`value`保存要提供的值。

### 6.5。`<sql:dateParam>`标签

在`<sql:query>`或`<sql:update>`标签中使用了`<sql:dateParam>`标签，为 sql 语句中的值占位符提供日期和时间值。

我们可以在 JSP 中这样定义它:

```java
<sql:update dataSource = "${dataSource}" var = "count">
    UPDATE Users SET registered = ? WHERE email = ?
    <sql:dateParam value = "<%=registered%>" type = "DATE" />
    <sql:param value = "<%=email%>" />
</sql:update>
```

像`<sql:param>`标签一样，`<sql:dateParam>`标签也有一个`value` 属性和一个附加的`type`属性，该属性的值可以是`date, time` 或`timestamp` (日期和时间)中的一个。

### 6.6。`<sql:transaction>`标签

`<sql:transaction>`标签用于通过将`<sql:query>`和`<sql:update>`标签组合在一起来创建类似 JDBC 事务的操作，如下所示:

```java
<sql:transaction dataSource = "${dataSource}">
    <sql:update var = "count">
        UPDATE Users SET first_name = 'Patrick-Ellis' WHERE
          email='[[email protected]](/web/20220627172721/https://www.baeldung.com/cdn-cgi/l/email-protection)'
    </sql:update>
    <sql:update var = "count">
        UPDATE Users SET last_name = 'Nelson' WHERE 
          email ='[[email protected]](/web/20220627172721/https://www.baeldung.com/cdn-cgi/l/email-protection)'
    </sql:update>
    <sql:update var = "count">
        INSERT INTO Users(first_name, last_name, email) 
          VALUES ('Grace', 'Adams', '[[email protected]](/web/20220627172721/https://www.baeldung.com/cdn-cgi/l/email-protection)');
    </sql:update>
</sql:transaction>
```

标签确保所有数据库操作被成功处理(提交),或者在任何操作中出现错误时，所有操作都正常失败(回滚)。

## 7 .**。JSTL 函数**

JSTL 方法是 JSP 中数据操作的实用工具。虽然有些函数采用不同的数据类型，但大多数都是专用于`String`操作的。

要在 JSP 中启用 JSTL 方法，我们需要将 taglib 添加到页面中:

```java
<%@ taglib prefix = "fn"
  uri = "http://java.sun.com/jsp/jstl/functions" %>
```

让我们看看这些函数以及如何使用它们。

### 7.1。`fn:contains()`和`fn:containsIgnoreCase()`

`fn:contains()`方法评估一个`String`来检查它是否包含一个给定的子串，如下所示:

```java
<c:set var = "string1" value = "This is first string"/>
<c:if test = "${fn:contains(string1, 'first')}">
    <p>Found 'first' in string<p>
</c:if>
```

`fn:contains()` 函数接受两个`String`参数；第一个是源`String`，第二个参数是子串。它根据评估结果返回一个布尔值。

`fn:containsIgnoreCase()`函数是`fn:contains()`方法的不区分大小写的变体，可以这样使用:

```java
<c:if test = "${fn:containsIgnoreCase(string1, 'first')}">
    <p>Found 'first' string<p>
</c:if>
<c:if test = "${fn:containsIgnoreCase(string1, 'FIRST')}">
    <p>Found 'FIRST' string<p>
</c:if>
```

### 7.3。`fn:endsWith()`功能

`fn:endsWith()`函数评估一个`String`来检查它的后缀是否匹配另一个子串。它需要两个参数:第一个参数是要测试其后缀的`String`，而第二个参数是被测试的后缀。

我们可以这样定义:

```java
<c:if test = "${fn:endsWith(string1, 'string')}">
    <p>String ends with 'string'<p>
</c:if>
```

### 7.4。`fn:escapeXml()`功能

`fn:escapeXML()`函数用于转义输入`String`中的 XML 标记，如下所示:

```java
<p>${fn:escapeXml(string1)}</p>
```

### 7.5。`fn:indexOf()`功能

`fn:indexOf()`函数遍历`String`并返回给定子字符串第一次出现的索引。

它需要两个参数:第一个参数是 source `String`，第二个参数是匹配并返回第一个匹配项的子字符串。

`fn:indexOf()`函数返回一个整数，其用法如下:

```java
<p>Index: ${fn:indexOf(string1, "first")}</p>
```

### 7.6。`fn:join()`功能

`fn:join()`函数将一个数组的所有元素连接成一个单独的`String`，可以这样使用:

```java
<c:set var = "string3" value = "${fn:split(string1, ' ')}" />
<c:set var = "string4" value = "${fn:join(string3, '-')}" />
```

### 7.7。`fn:length()` 功能

`fn:length()`函数返回给定集合中元素的数量或给定`String.`中字符的数量

`fn:length()` 函数接受一个单独的`Object`,它可以是一个集合或者是一个`String`,并返回一个整数，如下所示:

```java
<p>Length: ${fn:length(string1)}</p>
```

### 7.8。`fn:replace()` 功能

`fn:replace()`函数用另一个`String.`替换字符串中所有出现的子字符串

它需要三个参数:source `String,`要在 source 中查找的子字符串，而`String`要替换子字符串的所有匹配项，如下所示:

```java
<c:set var = "string3" value = "${fn:replace(string1, 'first', 'third')}" />
```

### 7.9。`fn:split()`功能

`fn:split()`函数使用指定的分隔符对`String`执行拆分操作。下面是一个用法示例:

```java
<c:set var = "string3" value = "${fn:split(string1, ' ')}" />
```

### 7.10。`fn:startsWith()`功能

`fn:startsWith()`函数检查`String`的前缀，如果它匹配给定的子字符串，则返回 true，如下所示:

```java
<c:if test = "${fn:startsWith(string1, 'This')}">
    <p>String starts with 'This'</p>
</c:if>
```

### 7.11。`fn:substring()`功能

`fn:substring()`函数在指定的起始和结束索引处从源`String`创建一个子串。我们会这样使用它:

```java
<c:set var = "string3" value = "${fn:substring(string1, 5, 15)}" />
```

### 7.12。`fn:substringAfter()`功能

`fn:substringAfter()`函数检查源`String`中给定的子串，并在指定子串第一次出现后立即返回`String`。

我们会这样使用它:

```java
<c:set var = "string3" value = "${fn:substringAfter(string1, 'is')}" />
```

### 7.13。`fn:substringBefore()`功能

`fn:substringBefore()`函数检查源`String`中给定的子串，并返回指定子串第一次出现之前的`String`。

在我们的 JSP 页面中，它看起来像这样:

```java
<c:set var = "string3" value = "${fn:substringBefore(string1, 'is')}" />
```

### 7.14。`fn:toLowerCase()` 功能

`fn:to LowerCase()`函数将`String`中的所有字符转换成小写，可以这样使用:

```java
<c:set var = "string3" value = "${fn:toLowerCase(string1)}" />
```

### 7.15。`fn:toUpperCase()`功能

`fn:toUpperCase()`函数将`String`中的所有字符转换成大写:

```java
<c:set var = "string3" value = "${fn:toUpperCase(string1)}" />
```

### 7.16。`fn:trim()` 功能

`fn:trim()`函数删除`String:`中前后的空格

```java
<c:set var = "string1" value = "This is first String    "/>
```

## 9。结论

在这篇内容丰富的文章中，我们研究了各种 JSTL 标签以及如何使用它们。

像往常一样，代码片段可以在 GitHub 上找到[。](https://web.archive.org/web/20220627172721/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-mvc-forms-jsp)