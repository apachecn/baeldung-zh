# 用车把做模板

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/handlebars>

## 1.概观

在本教程中，我们将研究一下[Handlebars.java](https://web.archive.org/web/20220902002933/https://jknack.github.io/handlebars.java/)库，以便于模板管理。

## 2.Maven 依赖性

让我们从添加`[handlebars](https://web.archive.org/web/20220902002933/https://search.maven.org/search?q=g:com.github.jknack%2Ba:handlebars)`依赖项开始:

```java
<dependency>
    <groupId>com.github.jknack</groupId>
    <artifactId>handlebars</artifactId>
    <version>4.1.2</version>
</dependency>
```

## 3.一个简单的模板

车把模板可以是任何类型的文本文件。**由** `**{{name}} and {{#each people}}.**`等标签组成

然后我们通过传递一个上下文对象来填充这些标签，比如一个`Map`或者其他的`Object.`

### 3.1.使用`this`

**为了向我们的模板传递一个单独的`String`值，我们可以使用任何`Object` 作为上下文。**我们还必须在模板中使用{{ `this}} t` ag。

然后 Handlebars 调用上下文对象上的`toString` 方法，并用结果替换标签:

```java
@Test
public void whenThereIsNoTemplateFile_ThenCompilesInline() throws IOException {
    Handlebars handlebars = new Handlebars();
    Template template = handlebars.compileInline("Hi {{this}}!");

    String templateString = template.apply("Baeldung");

    assertThat(templateString).isEqualTo("Hi Baeldung!");
}
```

在上面的例子中，我们首先创建了一个 API 入口点`Handlebars,` 的实例。

然后，我们给这个实例一个模板。这里，**我们只是内联传递模板，**但是我们马上会看到一些更强大的方法。

最后，我们给出了编译后的模板的上下文。`{{this}}` 最终会调用`toString,` ，这就是我们看到`“Hi Baeldung!”`的原因。

### 3.2.传递一个`Map`作为上下文对象

我们刚刚看到了如何为我们的上下文发送一个`String`，现在让我们尝试一个`Map`:

```java
@Test
public void whenParameterMapIsSupplied_thenDisplays() throws IOException {
    Handlebars handlebars = new Handlebars();
    Template template = handlebars.compileInline("Hi {{name}}!");
    Map<String, String> parameterMap = new HashMap<>();
    parameterMap.put("name", "Baeldung");

    String templateString = template.apply(parameterMap);

    assertThat(templateString).isEqualTo("Hi Baeldung!");
}
```

与前面的例子类似，我们正在编译模板，然后传递上下文对象，但这次是作为`Map`。

另外，请注意我们使用的是`{{name}}` 而不是`{{this}}`。**这意味着我们的地图一定包含了`name`** `.`

### 3.3.将自定义对象作为上下文对象传递

**我们还可以向模板传递一个自定义对象:**

```java
public class Person {
    private String name;
    private boolean busy;
    private Address address = new Address();
    private List<Person> friends = new ArrayList<>();

    public static class Address {
        private String street;       
    }
}
```

使用`Person` 类，我们将获得与上一个例子相同的结果:

```java
@Test
public void whenParameterObjectIsSupplied_ThenDisplays() throws IOException {
    Handlebars handlebars = new Handlebars();
    Template template = handlebars.compileInline("Hi {{name}}!");
    Person person = new Person();
    person.setName("Baeldung");

    String templateString = template.apply(person);

    assertThat(templateString).isEqualTo("Hi Baeldung!");
}
```

模板中的`{{name}}` 将钻入我们的`Person`对象，并获取`name` 字段`.` 的值

## 4.模板加载器

到目前为止，我们已经使用了代码中定义的模板。然而，这不是唯一的选择。我们也可以从文本文件中读取模板。

Handlebars.java 为从类路径、文件系统或 servlet 上下文中读取模板提供了特殊支持。**默认情况下，Handlebars 扫描类路径来加载给定的模板:**

```java
@Test
public void whenNoLoaderIsGiven_ThenSearchesClasspath() throws IOException {
    Handlebars handlebars = new Handlebars();
    Template template = handlebars.compile("greeting");
    Person person = getPerson("Baeldung");

    String templateString = template.apply(person);

    assertThat(templateString).isEqualTo("Hi Baeldung!");
}
```

**所以，因为我们调用了`compile`而不是`compileInline,`** 这是对 Handlebars 在类路径上寻找`/greeting.hbs` 的一个提示。

然而，我们也可以用`ClassPathTemplateLoader`来配置这些属性:

```java
@Test
public void whenClasspathTemplateLoaderIsGiven_ThenSearchesClasspathWithPrefixSuffix() throws IOException {
    TemplateLoader loader = new ClassPathTemplateLoader("/handlebars", ".html");
    Handlebars handlebars = new Handlebars(loader);
    Template template = handlebars.compile("greeting");
    // ... same as before
}
```

在这种情况下，我们告诉 **Handlebars 在类路径**中寻找`/handlebars/greeting.html`。

最后，我们可以链接多个`TemplateLoader`实例:

```java
@Test
public void whenMultipleLoadersAreGiven_ThenSearchesSequentially() throws IOException {
    TemplateLoader firstLoader = new ClassPathTemplateLoader("/handlebars", ".html");
    TemplateLoader secondLoader = new ClassPathTemplateLoader("/templates", ".html");
    Handlebars handlebars = new Handlebars().with(firstLoader, secondLoader);
    // ... same as before
}
```

所以，在这里，我们有两个加载器，这意味着 Handlebars 将在两个目录中搜索`greeting`模板。

## 5.内置助手

在编写模板时，内置的助手为我们提供了额外的功能。

### 5.1.`with`助手

**`with` 助手改变当前上下文**:

```java
{{#with address}}
<h4>I live in {{street}}</h4>
{{/with}}
```

在我们的样本模板中，`{{#with address}}` 标签开始这个部分，`{{/with}}` 标签结束这个部分`.`

**本质上，我们正在钻取当前的上下文对象——假设 p`erson`——并将`address` 设置为`with`部分** `.` 的本地上下文。此后，该部分中的每个字段引用都将由`person.address`前置。

因此，`{{street}}` 标签将保存`person.address.street`的值:

```java
@Test
public void whenUsedWith_ThenContextChanges() throws IOException {
    Handlebars handlebars = new Handlebars(templateLoader);
    Template template = handlebars.compile("with");
    Person person = getPerson("Baeldung");
    person.getAddress().setStreet("World");

    String templateString = template.apply(person);

    assertThat(templateString).contains("<h4>I live in World</h4>");
}
```

我们正在编译模板，并将一个`Person`实例指定为上下文对象。注意，`Person`类有一个`Address`字段。这是我们提供给`with`助手的字段。

虽然我们进入了上下文对象的一个层次，但是如果上下文对象有几个嵌套层次的话，再深入一些也是很好的。

### 5.2.`each`助手

**`each`助手遍历集合**:

```java
{{#each friends}}
<span>{{name}} is my friend.</span>
{{/each}}
```

作为使用`{{#each friends}}`和`{{/each}}`标签开始和结束迭代部分的结果，Handlebars 将迭代上下文对象的`friends`字段。

```java
@Test
public void whenUsedEach_ThenIterates() throws IOException {
    Handlebars handlebars = new Handlebars(templateLoader);
    Template template = handlebars.compile("each");
    Person person = getPerson("Baeldung");
    Person friend1 = getPerson("Java");
    Person friend2 = getPerson("Spring");
    person.getFriends().add(friend1);
    person.getFriends().add(friend2);

    String templateString = template.apply(person);

    assertThat(templateString)
      .contains("<span>Java is my friend.</span>", "<span>Spring is my friend.</span>");
}
```

在这个例子中，我们将两个`Person`实例分配给上下文对象的`friends`字段。因此，Handlebars 在最终输出中重复 HTML 部分两次。

### 5.3.`if`助手

最后，**助手`if`提供了条件渲染**。

```java
{{#if busy}}
<h4>{{name}} is busy.</h4>
{{else}}
<h4>{{name}} is not busy.</h4>
{{/if}}
```

在我们的模板中，我们根据`busy`字段提供不同的消息。

```java
@Test
public void whenUsedIf_ThenPutsCondition() throws IOException {
    Handlebars handlebars = new Handlebars(templateLoader);
    Template template = handlebars.compile("if");
    Person person = getPerson("Baeldung");
    person.setBusy(true);

    String templateString = template.apply(person);

    assertThat(templateString).contains("<h4>Baeldung is busy.</h4>");
}
```

编译完模板后，我们将设置上下文对象。**由于`busy`字段为`true`，最终输出变为`<h4>Baeldung is busy.</h4>`** 。

## 6.自定义模板助手

我们也可以创建自己的自定义助手。

### 6.1.`Helper`

**`Helper`接口使我们能够创建一个模板助手。**

作为第一步，我们必须提供一个`Helper`的实现:

```java
new Helper<Person>() {
    @Override
    public Object apply(Person context, Options options) throws IOException {
        String busyString = context.isBusy() ? "busy" : "available";
        return context.getName() + " - " + busyString;
    }
}
```

正如我们所见，`Helper`接口只有一个接受`context`和`options` 对象的方法。出于我们的目的，我们将输出`Person`的`name`和`busy`字段。

**创建助手后，我们还必须注册带有把手的自定义助手**:

```java
@Test
public void whenHelperIsCreated_ThenCanRegister() throws IOException {
    Handlebars handlebars = new Handlebars(templateLoader);
    handlebars.registerHelper("isBusy", new Helper<Person>() {
        @Override
        public Object apply(Person context, Options options) throws IOException {
            String busyString = context.isBusy() ? "busy" : "available";
            return context.getName() + " - " + busyString;
        }
    });

    // implementation details
}
```

在我们的例子中，我们使用`Handlebars.registerHelper()` 方法在名称`isBusy` 下注册我们的助手。

**最后一步，我们必须在模板中使用助手的名字**定义一个标签:

```java
{{#isBusy this}}{{/isBusy}}
```

请注意，每个辅助对象都有一个开始和结束标记。

### 6.2.助手方法

**当我们使用`Helper`接口时，我们只能创建一个助手**。**相反，助手源类使我们能够定义多个模板助手。**

而且，我们不需要实现任何特定的接口。我们只需在一个类中编写我们的助手方法，然后 HandleBars 使用反射提取助手定义:

```java
public class HelperSource {

    public String isBusy(Person context) {
        String busyString = context.isBusy() ? "busy" : "available";
        return context.getName() + " - " + busyString;
    }

    // Other helper methods
}
```

因为一个助手源可以包含多个助手实现，所以注册不同于单个助手注册:

```java
@Test
public void whenHelperSourceIsCreated_ThenCanRegister() throws IOException {
    Handlebars handlebars = new Handlebars(templateLoader);
    handlebars.registerHelpers(new HelperSource());

    // Implementation details
}
```

我们使用`Handlebars.registerHelpers()` 方法注册我们的助手。此外，**助手方法的名字变成了助手标签的名字**。

## 7.模板重用

Handlebars 库提供了几种重用现有模板的方法。

### 7.1.模板包含

模板包含是重用模板的方法之一。它支持模板的组合。

```java
<h4>Hi {{name}}!</h4>
```

这是`header`模板—`header.html.`的内容

为了在另一个模板中使用它，我们必须参考`header`模板。

```java
{{>header}}
<p>This is the page {{name}}</p>
```

我们有`page` 模板 `– page.html –` ，它包括使用`{{>header}}.`的`header`模板

当 Handlebars.java 处理模板时，最终输出也将包含`header`的内容:

```java
@Test
public void whenOtherTemplateIsReferenced_ThenCanReuse() throws IOException {
    Handlebars handlebars = new Handlebars(templateLoader);
    Template template = handlebars.compile("page");
    Person person = new Person();
    person.setName("Baeldung");

    String templateString = template.apply(person);

    assertThat(templateString)
      .contains("<h4>Hi Baeldung!</h4>", "<p>This is the page Baeldung</p>");
}
```

### 7.2.模板继承

作为组合的替代， **Handlebars 提供了模板继承**。

我们可以使用`{{#block}}`和`{{#partial}}`标签来实现继承关系:

```java
<html>
<body>
{{#block "intro"}}
  This is the intro
{{/block}}
{{#block "message"}}
{{/block}}
</body>
</html>
```

通过这样做，`messagebase` 模板有两个模块——`intro`和`message`。

为了应用继承，我们需要使用`{{#partial}}`在其他模板中覆盖这些`blocks`:

```java
{{#partial "message" }}
  Hi there!
{{/partial}}
{{> messagebase}}
```

这是`simplemessage`模板。注意，我们包含了`messagebase`模板，也覆盖了`message`块。

## 8.摘要

在本教程中，我们已经了解了 Handlebars.java 创建和管理模板。

我们从基本的标签用法开始，然后查看加载车把模板的不同选项。

我们还研究了提供大量功能的模板助手。最后，我们研究了重用模板的不同方法。

最后，在 GitHub 上查看所有示例[的源代码。](https://web.archive.org/web/20220902002933/https://github.com/eugenp/tutorials/tree/master/libraries-2)