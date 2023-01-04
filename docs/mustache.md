# 小胡子简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/mustache>

## 1.概观

在本文中，我们将关注于 [Mustache](https://web.archive.org/web/20220909002549/https://mustache.github.io/) 模板，并使用它的一个[Java API](https://web.archive.org/web/20220909002549/https://github.com/spullara/mustache.java)来生成动态 HTML 内容。

Mustache 是一个**无逻辑模板引擎，用于创建动态内容**，如 HTML、配置文件等。

## 2。简介

简单地说，该引擎被归类为`logicless`，因为它没有支持 if-else 语句和 for 循环的构造。

Mustache 模板由用 **{ { } }** (类似于 mustaches——因此得名)包围的标记名组成，并由包含模板数据的模型对象支持。

## 3。Maven 依赖关系

客户端和服务器端的多种语言都支持模板的编译和执行。

为了能够处理来自 Java 的模板，我们利用了它的 Java 库，该库可以作为 Maven 依赖项添加。

Java 8+:

```
<dependency>
    <groupId>com.github.spullara.mustache.java</groupId>
    <artifactId>compiler</artifactId>
    <version>0.9.4</version>
</dependency>
```

Java 6/7:

```
<dependency>
    <groupId>com.github.spullara.mustache.java</groupId>
    <artifactId>compiler</artifactId>
    <version>0.8.18</version>
</dependency>
```

我们可以在[中央 Maven 资源库](https://web.archive.org/web/20220909002549/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22com.github.spullara.mustache.java%22%20AND%20a%3A%22compiler%22)中查看该库的最新版本。

## 4。用途

让我们看一个简单的场景，展示如何:

1.  写一个简单的模板
2.  使用 Java API 编译模板
3.  通过提供必要的数据来执行它

### 4.1。一个简单的小胡子模板

我们将创建一个简单的模板来显示 todo 任务的详细信息:

```
<h2>{{title}}</h2>
<small>Created on {{createdOn}}</small>
<p>{{text}}</p>
```

在上面的模板中，花括号({{}})内的字段可以是:

*   Java 类的方法和属性
*   一个`Map`对象的键

### 4.2。编辑小胡子模板

我们可以编译如下所示的模板:

```
MustacheFactory mf = new DefaultMustacheFactory();
Mustache m = mf.compile("todo.mustache"); 
```

`MustacheFactory`在类路径中搜索给定的模板。在我们的例子中，我们将`todo.mustache`放在`src/main/resources`下面。

### 4.3。执行小胡子模板

提供给模板的数据将是`Todo`类的一个实例，其定义是:

```
public class Todo {
    private String title;
    private String text;
    private boolean done;
    private Date createdOn;
    private Date completedOn;

    // constructors, getters and setters
}
```

可以执行编译后的模板来获得 HTML，如下所示:

```
Todo todo = new Todo("Todo 1", "Description");
StringWriter writer = new StringWriter();
m.execute(writer, todo).flush();
String html = writer.toString();
```

## 5。胡子部分和迭代

现在让我们看看如何列出待办事项。为了迭代列表数据，我们使用了 Mustache 部分。

一个段是一个代码块，它根据当前上下文中的键值重复一次或多次。

它看起来像这样:

```
{{#todo}}
<!-- Other code -->
{{/todo}}
```

一个节以井号(#)开始，以斜杠(/)结束，其中每个符号后面跟一个键，该键的值用作呈现该节的基础。

根据该项的值，可能会出现以下情况:

### 5.1。具有非空列表或非假值的部分

让我们创建一个使用部分的模板`todo-section.mustache`:

```
{{#todo}}
<h2>{{title}}</h2>
<small>Created on {{createdOn}}</small>
<p>{{text}}</p>
{{/todo}}
```

让我们来看看这个模板的运行情况:

```
@Test
public void givenTodoObject_whenGetHtml_thenSuccess() 
  throws IOException {

    Todo todo = new Todo("Todo 1", "Todo description");
    Mustache m = MustacheUtil.getMustacheFactory()
      .compile("todo.mustache");
    Map<String, Object> context = new HashMap<>();
    context.put("todo", todo);

    String expected = "<h2>Todo 1</h2>";
    assertThat(executeTemplate(m, todo)).contains(expected);
}
```

让我们创建另一个模板`todos.mustache`来列出待办事项:

```
{{#todos}}
<h2>{{title}}</h2>
{{/todos}}
```

并使用它创建待办事项列表:

```
@Test
public void givenTodoList_whenGetHtml_thenSuccess() 
  throws IOException {

    Mustache m = MustacheUtil.getMustacheFactory()
      .compile("todos.mustache");

    List<Todo> todos = Arrays.asList(
      new Todo("Todo 1", "Todo description"),
      new Todo("Todo 2", "Todo description another"),
      new Todo("Todo 3", "Todo description another")
    );
    Map<String, Object> context = new HashMap<>();
    context.put("todos", todos);

    assertThat(executeTemplate(m, context))
      .contains("<h2>Todo 1</h2>")
      .contains("<h2>Todo 2</h2>")
      .contains("<h2>Todo 3</h2>");
}
```

### 5.2。具有空`List`或`False`或`Null`值的部分

让我们用一个`null`值来测试`todo-section.mustache`:

```
@Test
public void givenNullTodoObject_whenGetHtml_thenEmptyHtml() 
  throws IOException {
    Mustache m = MustacheUtil.getMustacheFactory()
      .compile("todo-section.mustache");
    Map<String, Object> context = new HashMap<>();
    assertThat(executeTemplate(m, context)).isEmpty();
}
```

同样，用空列表测试`todos.mustache`:

```
@Test
public void givenEmptyList_whenGetHtml_thenEmptyHtml() 
  throws IOException {
    Mustache m = MustacheUtil.getMustacheFactory()
      .compile("todos.mustache");

    Map<String, Object> context = new HashMap<>();
    assertThat(executeTemplate(m, context)).isEmpty();;
}
```

## 6。倒置部分

**反转部分是那些基于键的**或`false`或`null`值或空列表的不存在而只呈现一次的部分。换句话说，当一个部分没有被呈现时，这些被呈现。

这些以插入符号(^)开始，以斜杠(/)结束，如下所示:

```
{{#todos}}
<h2>{{title}}</h2>
{{/todos}}
{{^todos}}
<p>No todos!</p>
{{/todos}}
```

上面的模板提供了一个空列表:

```
@Test
public void givenEmptyList_whenGetHtmlUsingInvertedSection_thenHtml() 
  throws IOException {

    Mustache m = MustacheUtil.getMustacheFactory()
      .compile("todos-inverted-section.mustache");

    Map<String, Object> context = new HashMap<>();
    assertThat(executeTemplate(m, context).trim())
      .isEqualTo("<p>No todos!</p>");
}
```

## 7。Lambdas

mustache 部分的键的**值可以是函数或 lambda 表达式**。在这种情况下，完整的 lambda 表达式是通过将节中的文本作为参数传递给 lambda 表达式来调用的。

我们来看一个模板`todos-lambda.mustache`:

```
{{#todos}}
<h2>{{title}}{{#handleDone}}{{doneSince}}{{/handleDone}}</h2>
{{/todos}}
```

`handleDone`键解析为 Java 8 lambda 表达式，如下所示:

```
public Function<Object, Object> handleDone() {
    return (obj) -> done ? 
      String.format("<small>Done %s minutes ago<small>", obj) : "";
}
```

执行上述模板生成的 HTML 是:

```
<h2>Todo 1</h2>
<h2>Todo 2</h2>
<h2>Todo 3<small>Done 5 minutes ago<small></h2>
```

## 8。结论

在这篇介绍性文章中，我们研究了如何创建带有截面、倒置截面和 lambdas 的 mustache 模板。我们使用 Java API 通过提供相关数据来编译和执行模板。

小胡子还有一些更高级的功能值得探索，比如:

*   提供一个可调用的值，这将导致并发计算
*   使用`DecoratedCollection`获取集合元素的第一个、最后一个和索引
*   `invert` API，它给出给定文本和模板的数据

和往常一样，Github 上的[提供了完整的源代码。](https://web.archive.org/web/20220909002549/https://github.com/eugenp/tutorials/tree/master/mustache)