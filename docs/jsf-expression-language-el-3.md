# JSF 表达式语言 3.0 指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jsf-expression-language-el-3>

## 1。概述

在本文中，我们将了解 Expression Language 3.0 版(EL 3.0)的最新特性、改进和兼容性问题。

这是在撰写本文时的最新版本，并且附带了更新的 JavaEE 应用服务器(JBoss EAP 7 和 Glassfish 4 就是实现了对它的支持的很好的例子)。

这篇文章只关注 EL 3.0 的发展——要了解更多关于表达式语言的一般知识，请先阅读 [EL version 2.2](/web/20221221014653/http://www.baeldung.com/intro-to-jsf-expression-language) 文章。

## 2。先决条件

本文中展示的例子也已经在 Tomcat 8 上测试过了。要使用 EL3.0，您必须添加以下依赖项:

```java
<dependency>
    <groupId>javax.el</groupId>
    <artifactId>javax.el-api</artifactId>
    <version>3.0.0</version>
</dependency>
```

您总是可以通过这个[链接](https://web.archive.org/web/20221221014653/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22javax.el%22%20AND%20a%3A%22javax.el-api%22)来检查 Maven 存储库中的最新依赖项。

## 3。λ表达式

最新的 EL 迭代为 lambda 表达式提供了非常健壮的支持。Lambda 表达式是在 Java SE 8 中引入的，但是在 EL 中对它的支持来自 Java EE 7。

这里的实现是全功能的，允许在 EL 使用和评估中有很大的灵活性(和一些隐含的风险)。

### 3.1。λEL 值表达式

此功能的基本用途是允许我们将 lambda 表达式指定为 EL 值表达式中的值类型:

```java
<h:outputText id="valueOutput" 
  value="#{(x->x*x*x);(ELBean.pageCounter)}"/>
```

由此延伸，可以在 EL 中命名 lambda 函数，以便在复合语句中重用，就像在 Java SE 的 lambda 表达式中一样。复合 lambda 表达式可以用分号(`;`)分隔:

```java
<h:outputText id="valueOutput" 
  value="#{cube=(x->x*x*x);cube(ELBean.pageCounter)}"/> 
```

这个代码片段将函数分配给`cube`标识符，然后可以立即重用它。

### 3.2。将 Lambda 表达式传递给后台 Bean

让我们更进一步:通过将逻辑封装在 EL 表达式中(作为 lambda)并将其传递给 JSF 支持 bean，我们可以获得很大的灵活性:

```java
<h:outputText id="valueOutput" 
  value="#{ELBean.multiplyValue(x->x*x*x)}"/> 
```

这现在允许我们将 lambda 表达式整体作为`javax.el.LambdaExpression`的一个实例来处理:

```java
public String multiplyValue(LambdaExpression expr){
    return (String) expr.invoke( 
      FacesContext.getCurrentInstance().getELContext(), pageCounter);
} 
```

这是一项引人注目的功能，它允许:

*   一种封装逻辑的干净方式，提供了一种非常灵活的函数式编程范例。上面的后备 bean 逻辑可能基于从不同来源获取的值。
*   一种在 JDK 8 之前的代码库中引入 lambda 支持的简单方法，这些代码库可能还没有准备好升级。
*   使用新的流/集合 API 的强大工具。

## 4。集合 API 增强功能

在早期版本的 EL 中，对集合 API 的支持有所欠缺。EL 3.0 在对 Java 集合的支持中引入了主要的 API 改进，就像 lambda 表达式一样，EL 3.0 在 Java EE 7 中提供了 JDK 8 流支持。

### 4.1。动态集合定义

作为 3.0 中的新特性，我们现在可以在 EL 中动态定义特定的数据结构:

*   列表:

```java
 <h:dataTable var="listItem" value="#{['1','2','3']}">
       <h:column id="nameCol">
           <h:outputText id="name" value="#{listItem}"/>
       </h:column>
   </h:dataTable> 
```

*   集合:

```java
 <h:dataTable var="setResult" value="#{{'1','2','3'}}">
    ....
   </h:dataTable> 
```

**`Note:`** 与普通 Java 一样`Sets,`元素的顺序是不可预知的，当列出

*   地图:

```java
 <h:dataTable var="mapResult" 
     value="#{{'one':'1','two':'2','three':'3'}}"> 
```

**`Tip` :** 教科书中定义动态地图时的一个常见错误是使用双引号(")而不是单引号作为地图键——这会导致 EL 编译错误。

### 4.2。高级收集操作

EL3.0 支持高级查询语义，结合了 lambda 表达式的强大功能、新的流 API 和类似 SQL 的操作，如连接和分组。我们不会在本文中涉及这些，因为这些是高级主题。让我们看一个例子来展示它的威力:

```java
<h:dataTable var="streamResult" 
  value="#{['1','2','3'].stream().filter(x-> x>1).toList()}">
    <h:column id="nameCol">
        <h:outputText id="name" value="#{streamResult}"/>
    </h:column>
</h:dataTable> 
```

上表将使用传递的 lambda 表达式过滤后备列表

```java
 <h:outputLabel id="avgLabel" for="avg" 
   value="Average of integer list value"/>
 <h:outputText id="avg" 
   value="#{['1','2','3'].stream().average().get()}"/>
```

输出文本`avg`将计算列表中数字的平均值。通过新的 [`Optional` API](/web/20221221014653/http://www.baeldung.com/java-8-new-features) (对以前版本的另一个改进)，这两个操作都是空安全的。

请记住，对此的支持不需要 JDK 8，只需要 JavaEE 7/EL3.0。这意味着您可以在 EL 中完成大多数 JDK 8 `Stream`操作，但不能在后台 bean Java 代码中完成。

`**Tip:**` 您可以使用 JSTL `<c:set/>` 标签将您的数据结构声明为页面级变量，并在整个 JSF 页面中操作它:

```java
 <c:set var='pageLevelNumberList' value="#{[1,2,3]}"/> 
```

您现在可以在整个页面中引用`“#{pageLevelNumberList}”`,就像它是真正的 JSF 组件或 bean 一样。这允许在整个页面中进行大量的重用

```java
<h:outputText id="avg" 
  value="#{pageLevelNumberList.stream().average().get()}"/>
```

## 5。静态字段和方法

在早期版本的 EL 中，不支持静态字段、方法或枚举访问。事情已经变了。

首先，我们必须手动将包含常量的类导入到 EL 上下文中。这最好是尽早完成。这里我们在 JSF 管理的 bean 的`@PostConstruct` 初始化器中做这件事(一个`ServletContextListener` 也是一个可行的候选者):

```java
 @PostConstruct
 public void init() {
     FacesContext.getCurrentInstance()
       .getApplication().addELContextListener(new ELContextListener() {
         @Override
         public void contextCreated(ELContextEvent evt) {
             evt.getELContext().getImportHandler()
              .importClass("com.baeldung.el.controllers.ELSampleBean");
         }
     });
 } 
```

然后，我们在所需的类中定义一个`String` 常量字段(如果您愿意，也可以定义一个`Enum` ):

```java
public static final String constantField 
  = "THIS_IS_NOT_CHANGING_ANYTIME_SOON"; 
```

之后，我们现在可以访问 EL 中的变量:

```java
 <h:outputLabel id="staticLabel" 
   for="staticFieldOutput" value="Constant field access: "/>
 <h:outputText id="staticFieldOutput" 
   value="#{ELSampleBean.constantField}"/> 
```

根据 EL 3.0 规范，`java.lang.*`之外的任何类都需要手动导入，如图所示。只有在这样做之后，类中定义的常量才可以在 EL 中使用。理想情况下，导入是作为 JSF 运行时初始化的一部分完成的。

这里需要注意几点:

*   语法要求字段和方法是`public, static` (对于方法是`final`)
*   The syntax changed between the initial draft of the EL 3.0 specification and the release version. So in some textbooks, you might still find something that looks like:

    ```java
    T(YourClass).yourStaticVariableOrMethod
    ```

    这在实践中是行不通的(简化语法的设计变更是在实现周期的后期决定的)

*   最终发布的语法仍然存在一个错误——运行这些语法的最新版本很重要。

## 6。结论

我们已经研究了最新 EL 实现中的一些亮点。主要的改进是为 API 带来一些很酷的新特性，比如 lambda 和 streams 灵活性。

有了 EL 的灵活性，记住 JSF 框架的一个设计目标是很重要的:使用 MVC 模式清晰地分离关注点。

因此，值得注意的是，API 的最新改进可能会让我们接触到 JSF 的反模式，因为 EL 现在有能力处理真正的业务逻辑——比以前更有能力。因此，在现实世界的实施过程中，记住这一点很重要，以确保职责明确划分。

当然，文章[中的例子可以在 GitHub 上找到。](https://web.archive.org/web/20221221014653/https://github.com/eugenp/tutorials/tree/master/jsf)