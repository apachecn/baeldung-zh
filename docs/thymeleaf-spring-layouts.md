# 百里香叶:自定义布局方言

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/thymeleaf-spring-layouts>

## 1。简介

[百里叶](https://web.archive.org/web/20220120194602/http://www.thymeleaf.org/)是一个 Java 模板引擎，用于处理和创建 HTML、XML、JavaScript、CSS 和纯文本。关于百里香和春天的介绍，请看[这篇文章](/web/20220120194602/https://www.baeldung.com/thymeleaf-in-spring-mvc)。

在这篇文章中，我们将把重点放在模板上——大多数相当复杂的网站都需要这样或那样做的事情。简而言之，页面需要共享公共的页面组件，比如页眉、页脚、菜单等等。

百里香通过自定义方言解决了这个问题——您可以使用`Thymeleaf Standard Layout System`或`Layout Dialect`构建布局——它们使用装饰模式来处理布局文件。

在这篇文章中，我们将讨论`Thymeleaf Layout Dialect`的一些特性——可以在[这里](https://web.archive.org/web/20220120194602/https://github.com/ultraq/thymeleaf-layout-dialect)找到。更具体地说，我们将讨论它的功能，如创建布局，自定义标题或头部元素合并。

## 2。Maven 依赖关系

首先，让我们看看将百里香与 Spring 集成所需的配置。在我们的依赖关系中需要`thymeleaf-spring`库:

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

注意，对于 Spring 4 项目，必须使用`thymeleaf-spring4`库来代替`thymeleaf-spring5`。依赖关系的最新版本可以在[这里](https://web.archive.org/web/20220120194602/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.thymeleaf%22%20AND%20a%3A%22thymeleaf-spring5%22)找到。

我们还需要自定义布局方言的依赖关系:

```java
<dependency>
    <groupId>nz.net.ultraq.thymeleaf</groupId>
    <artifactId>thymeleaf-layout-dialect</artifactId>
    <version>2.4.1</version>
</dependency>
```

最新版本可以在 [Maven 中央存储库](https://web.archive.org/web/20220120194602/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22com.github.zhanhb%22%20AND%20a%3A%22thymeleaf-layout-dialect%22)找到。

## 3。百里香布局方言配置

在这一节中，我们将讨论如何配置百里香来使用`Layout Dialect`。如果你想退一步看看如何在你的 web app 项目中配置百里香 3.0，请查看这个[教程](/web/20220120194602/https://www.baeldung.com/spring-thymeleaf-3)。

一旦我们将 Maven dependency 添加为项目的一部分，我们就需要将`Layout Dialect`添加到我们现有的百里香模板引擎中。我们可以通过 Java 配置做到这一点:

```java
SpringTemplateEngine engine = new SpringTemplateEngine();
engine.addDialect(new LayoutDialect());
```

或者使用 XML 文件配置:

```java
<bean id="templateEngine" class="org.thymeleaf.spring5.SpringTemplateEngine">
    <property name="additionalDialects">
        <set>
            <bean class="nz.net.ultraq.thymeleaf.LayoutDialect"/>
        </set>
    </property>
</bean> 
```

当修饰内容和布局模板的部分时，默认行为是将所有内容元素放在布局元素之后。

有时我们需要更智能的元素合并，允许将相似的元素组合在一起(js 脚本，样式表等等)。).

为了实现这一点，我们需要将排序策略添加到我们的配置中，在我们的例子中，它将是分组策略:

```java
engine.addDialect(new LayoutDialect(new GroupingStrategy()));
```

或者在 XML 中:

```java
<bean class="nz.net.ultraq.thymeleaf.LayoutDialect">
    <constructor-arg ref="groupingStrategy"/>
</bean>
```

默认策略是追加元素。有了上面提到的，我们将有一切排序和分组。

如果这两种策略都不适合我们的需求，我们可以实现自己的`SortingStrategy`并像上面那样将其传递给方言。

## 4。名称空间和属性处理器的特性

一旦完成配置，我们就可以开始使用`layout`名称空间和五个新的属性处理器:`decorate`、`title-pattern`、`insert`、`replace`和`fragment.`

为了创建用于 HTML 文件的布局模板，我们创建了以下文件，名为`template.html`:

```java
<!DOCTYPE html>
<html xmlns:layout="http://www.ultraq.net.nz/thymeleaf/layout">
...
</html>
```

如我们所见，我们将名称空间从标准的`xmlns:th=”http://www.thymeleaf.org”`更改为`xmlns:layout=”http://www.ultraq.net.nz/thymeleaf/layout”`。

现在我们可以开始在 HTML 文件中使用属性处理器了。

### 4.1。`layout:fragment`

为了在我们的布局中创建定制的部分或者可重用的模板，这些模板可以被共享相同名称的部分所替换，我们在我们的`template.html`主体中使用了`fragment`属性:

```java
<body>
    <header>
        <h1>New dialect example</h1>
    </header>
    <section layout:fragment="custom-content">
        <p>Your page content goes here</p>
    </section>
    <footer>
        <p>My custom footer</p>
        <p layout:fragment="custom-footer">Your custom footer here</p>
    </footer>
</body>
```

请注意，有两个部分将被我们的自定义 HTML 替换——内容和页脚。

编写缺省的 HTML 也很重要，如果找不到任何片段，将会使用这个 HTML。

### 4.2。`layout:decorate`

我们需要做的下一步是创建一个 HTML 文件，它将被我们的布局修饰:

```java
<!DOCTYPE html>
<html xmlns:layout="http://www.ultraq.net.nz/thymeleaf/layout"
  layout:decorate="~{template.html}">
<head>
<title>Layout Dialect Example</title>
</head>
<body>
    <section layout:fragment="custom-content">
        <p>This is a custom content that you can provide</p>
    </section>
    <footer>
        <p layout:fragment="custom-footer">This is some footer content
          that you can change</p>
    </footer>
</body>
</html>
```

如第三行所示，我们使用`layout:decorate` 并指定装饰器源。布局文件中与内容文件中的片段匹配的所有片段都将被其自定义实现替换。

### 4.3。`layout:title-pattern`

鉴于`Layout dialect`会自动用内容模板中的标题覆盖布局的标题，您可以保留布局中的部分标题。

例如，我们可以创建面包屑或在页面标题中保留网站的名称。在这种情况下，`layout:title-pattern`处理器会提供帮助。您只需在布局文件中指定:

```java
<title layout:title-pattern="$LAYOUT_TITLE - $CONTENT_TITLE">Baeldung</title>
```

因此，上一段中呈现的布局和内容文件的最终结果将如下所示:

```java
<title>Baeldung - Layout Dialect Example</title>
```

### 4.4。`layout:insert/layout:replace`

第一个处理器`layout:insert`，类似于百里香的原始处理器`th:insert`，但是允许将整个 HTML 片段传递给插入的模板。如果您有一些想要重用的 HTML，但是它的内容太复杂，无法单独用上下文变量来确定或构造，那么它就非常有用。

第二个是`layout:replace`，类似于第一个，但是具有`th:replace`的行为，它实际上用定义的片段的代码代替了主机标签。

## 5。结论

在本文中，我们描述了在百里香中实现布局的几种方法。

请注意，这些示例使用的是百里香 3.0 版；如果您想了解如何迁移您的项目，请遵循这个[过程](https://web.archive.org/web/20220120194602/https://ultraq.github.io/thymeleaf-layout-dialect/MigrationGuide.html)。

本教程的完整实现可以在 GitHub 项目中找到。

**怎么考？**我们的建议是先使用浏览器，然后检查现有的 JUnit 测试。

请记住，您可以使用上述`Layout Dialect`来构建布局，也可以轻松创建自己的解决方案。希望这篇文章能让你对这个话题有更多的了解，你会根据你的需要找到你喜欢的方法。