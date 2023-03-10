# 使用注释定义自定义 RAML 属性

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/raml-custom-properties-with-annotations>

[This article is part of a series:](javascript:void(0);)[• Introduction to RAML – The RESTful API Modeling Language](/web/20220122043809/https://www.baeldung.com/raml-restful-api-modeling-language-tutorial)
[• Eliminate Redundancies in RAML with Resource Types and Traits](/web/20220122043809/https://www.baeldung.com/simple-raml-with-resource-types-and-traits)
[• Modular RAML Using Includes, Libraries, Overlays and Extensions](/web/20220122043809/https://www.baeldung.com/modular-raml-includes-overlays-libraries-extensions)
• Define Custom RAML Properties Using Annotations (current article)

## 1。简介

在这篇关于 RAML(RESTful API 建模语言)系列的第四篇文章中，我们展示了**如何使用`annotations`为 RAML API 规范定义自定义属性**。这个过程也被称为扩展规范的元数据。

`Annotations`可用于为 RAML 处理工具提供挂钩，这些工具需要官方语言范围之外的附加规范。

## 2。声明注释类型

可以使用顶级的`annotationTypes`属性声明一个或多个 `annotation types`。

在最简单的情况下，`annotation type name`是指定它所需要的，在这种情况下，`annotation type value`被隐式定义为一个字符串:

```java
annotationTypes:
  simpleImplicitStringValueType:
```

这相当于这里显示的更明确的`annotation type`定义:

```java
annotationTypes:
  simpleExplicitStringValueType:
    type: string
```

在其他情况下，`annotation type`规范将包含一个被认为是`annotation type declaration`的值对象。

在这些情况下，`annotation type`使用与`data type`相同的语法定义，并增加了两个可选属性:`allowedTargets`，其值是一个字符串或一个字符串数组，限制了`annotation`可以应用到的目标位置的类型，以及`allowMultiple`，其布尔值表明`annotation`是否可以在单个目标中应用多次(默认为`false`)。

下面是一个简单的例子，它声明了一个包含附加属性的`annotation type`:

```java
annotationTypes:
  complexValueType:
    allowMultiple: true
    properties:
      prop1: integer
      prop2: string
      prop3: boolean
```

### 2.1。支持使用注释的目标位置

`Annotations`可以应用于(用在)几个根级目标位置，包括 API 本身的根级、`resource types`、`traits`、`data types`、`documentation items`、`security schemes`、`libraries`、`overlays`、`extensions`和其他`annotation types`。

`Annotations`也可用于`security scheme settings`、`resources`、`methods`、`response`、`request bodies`、`response bodies`、`named examples`的申报。

### 2.2。限制注释类型的目标

要将一个`annotation type`限制为一个或多个特定的目标位置类型，您可以定义它的`allowedTargets`属性。

当将`annotation type`限制为单一目标位置类型时，您可以为`allowedTargets`属性分配一个表示该目标位置类型的字符串值:

```java
annotationTypes:
  supportsOnlyOneTargetLocationType:
    allowedTargets: TypeDeclaration
```

为了允许一个`annotation type`有多个目标位置类型，您可以为`allowedTargets`属性分配一个表示这些目标位置类型的字符串值数组:

```java
annotationTypes:
  supportsMultipleTargetLocationTypes:
    allowedTargets: [ Library, Overlay, Extension ]
```

如果在`annotation type`上没有定义`allowedTargets`属性，那么默认情况下，该`annotation type`可以应用于任何支持的目标位置类型。

## 3。应用注释类型

一旦在 RAML API 规范的根级别定义了`annotation types`,就可以将它们应用到预期的目标位置，在每个实例中提供它们的属性值。目标位置内的`annotation type`的应用简称为该目标位置上的`annotation`。

### 3.1。语法

为了应用`annotation type`，添加圆括号()中的`annotation type name`作为目标位置的属性，并提供`annotation type`将用于该特定目标的`annotation type value`属性。如果`annotation type`在一个 RAML `library`中，那么你可以连接`library`引用，后跟一个点(。)后面是`annotation type name.`

### 3.2。示例

这里有一个例子，展示了我们如何将上面代码片段中列出的一些`annotation types`应用到我们 API 的各种`resources`和`methods`:

```java
/foos:
  type: myResourceTypes.collection
  (simpleImplicitStringValueType): alpha
  ...
  get:
    (simpleExplicitStringValueType): beta
  ...
  /{fooId}:
    type: myResourceTypes.item
    (complexValueType):
      prop1: 4
      prop2: testing
      prop3: true
```

## 4。用例

`annotations`的一个潜在用例是为 API 定义和配置测试用例。

假设我们想要开发一个 RAML 处理工具，它可以基于`annotations`针对我们的 API 生成一系列测试。我们可以定义下面的`annotation type`:

```java
annotationTypes:
  testCase:
    allowedTargets: [ Method ]
    allowMultiple: true
    usage: |
      Use this annotation to declare a test case.
      You may apply this annotation multiple times per location.
    properties:
      scenario: string
      setupScript?: string[]
      testScript: string[]
      expectedOutput?: string
      cleanupScript?: string[]
```

然后，我们可以通过应用`annotations`为我们的 `/foos`资源配置一系列测试用例，如下所示:

```java
/foos:
  type: myResourceTypes.collection
  get:
    (testCase):
      scenario: No Foos
      setupScript: deleteAllFoosIfAny
      testScript: getAllFoos
      expectedOutput: ""
    (testCase):
      scenario: One Foo
      setupScript: [ deleteAllFoosIfAny, addInputFoos ]
      testScript: getAllFoos
      expectedOutput: '[ { "id": 999, "name": Joe } ]'
      cleanupScript: deleteInputFoos
    (testCase):
      scenario: Multiple Foos
      setupScript: [ deleteAllFoosIfAny, addInputFoos ]
      testScript: getAllFoos
      expectedOutput: '[ { "id": 998, "name": "Bob" }, { "id": 999, "name": "Joe" } ]'
      cleanupScript: deleteInputFoos 
```

## 5。结论

在本教程中，我们展示了如何通过使用名为`annotations`的自定义属性来扩展 RAML API 规范的元数据。

首先，我们展示了如何使用顶级的`annotationTypes`属性来声明`annotation types`,并枚举了允许应用它们的目标位置的类型。

接下来，我们演示了如何在我们的 API 中应用`annotations`，并说明了如何限制给定的`annotation`可以应用到的目标位置的类型。

最后，我们通过定义测试生成工具可能支持的`annotation types`并展示如何将这些`annotations`应用于 API，引入了一个潜在的用例。

关于在 RAML 中使用`annotations`的更多信息，请访问 [RAML 1.0 规范](https://web.archive.org/web/20220122043809/https://github.com/raml-org/raml-spec/blob/master/versions/raml-10/raml-10.md/#annotations)。

你可以在 github 项目中查看本教程使用的 API 定义的完整实现。

**«** Previous[Modular RAML Using Includes, Libraries, Overlays and Extensions](/web/20220122043809/https://www.baeldung.com/modular-raml-includes-overlays-libraries-extensions)