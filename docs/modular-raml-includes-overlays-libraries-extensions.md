# 使用包含、库、覆盖和扩展的模块化 RAML

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/modular-raml-includes-overlays-libraries-extensions>

[This article is part of a series:](javascript:void(0);)[• Introduction to RAML – The RESTful API Modeling Language](/web/20220813052709/https://www.baeldung.com/raml-restful-api-modeling-language-tutorial)
[• Eliminate Redundancies in RAML with Resource Types and Traits](/web/20220813052709/https://www.baeldung.com/simple-raml-with-resource-types-and-traits)
• Modular RAML Using Includes, Libraries, Overlays and Extensions (current article)[• Define Custom RAML Properties Using Annotations](/web/20220813052709/https://www.baeldung.com/raml-custom-properties-with-annotations)

## 1。简介

在我们关于 RAML(RESTful API 建模语言)的[第一篇](/web/20220813052709/https://www.baeldung.com/raml-restful-api-modeling-language-tutorial)和[两篇](/web/20220813052709/https://www.baeldung.com/simple-raml-with-resource-types-and-traits)文章中，我们介绍了一些基本语法，包括数据类型和 JSON 模式的使用，并且我们展示了如何通过将通用模式提取到`resource types`和`traits`中来简化 RAML 定义。

在本文中，我们向**展示了如何利用`includes`、`libraries`、`overlays`和`extensions`将 RAML API 定义分解成模块**。

## 2。我们的 API

出于本文的目的，我们将把重点放在 API 中涉及名为`Foo`的实体类型的部分。

以下是构成我们 API 的资源:

*   获取`/api/v1/foos`
*   帖子`/api/v1/foos`
*   获取`/api/v1/foos/{fooId}`
*   放`/api/v1/foos/{fooId}`
*   删除`/api/v1/foos/{fooId}`

## 3。包括

`include`的目的是通过将属性值放在外部文件中，模块化 RAML 定义中的复杂属性值。

我们的第一篇文章简要介绍了当我们指定数据类型和其属性在整个 API 中被内联重复的例子时`includes`的使用。

### 3.1。一般用法和语法

`!include`标签接受一个参数:包含属性值的外部文件的位置。该位置可以是绝对 URL、相对于根 RAML 文件的路径或相对于包含文件的路径。

以正斜杠(/)开头的位置表示相对于根 RAML 文件位置的路径，以不带斜杠开头的位置被解释为相对于包含文件的位置。

后者的逻辑推论是被包含的文件本身可能包含其他的`!include`指令。

下面的例子展示了`!include`标签的所有三种用法:

```java
#%RAML 1.0
title: Baeldung Foo REST Services API
...
types: !include /types/allDataTypes.raml
resourceTypes: !include allResourceTypes.raml
traits: !include http://foo.com/docs/allTraits.raml
```

### 3.2。键入的片段

除了将所有的`types`、`resource types`或`traits`放在它们各自的`include`文件中，您还可以使用特殊类型的`includes`称为`typed fragments`来将这些结构分成多个`include`文件，为每个`type`、`resource type`或`trait`指定不同的文件。

你也可以用`typed fragments`来定义`user documentation items`、`named examples`、`annotations`、`libraries`、`overlays`和`extensions`。我们将在本文后面介绍`overlays`和`extensions`的用法。

虽然不是必需的，但是作为`typed fragment`的`include`文件的第一行可以是以下格式的 RAML 片段标识符:

```java
#%RAML 1.0 <fragment-type>
```

例如，`trait`的`typed fragment`文件的第一行应该是:

```java
#%RAML 1.0 Trait
```

如果使用了片段标识符，那么文件的内容必须只包含指定片段类型的有效 RAML。

让我们先来看看 API 的`traits`部分的一部分:

```java
traits:
  - hasRequestItem:
      body:
        application/json:
          type: <<typeName>>
  - hasResponseItem:
      responses:
          200:
            body:
              application/json:
                type: <<typeName>>
                example: !include examples/<<typeName>>.json
```

为了使用`typed fragments`模块化这个部分，我们首先重写`traits`部分如下:

```java
traits:
  - hasRequestItem: !include traits/hasRequestItem.raml
  - hasResponseItem: !include traits/hasResponseItem.raml
```

然后我们将编写文件`hasRequestItem.raml`:

```java
#%RAML 1.0 Trait
body:
  application/json:
    type: <<typeName>>
```

文件`hasResponseItem.raml`看起来像这样:

```java
#%RAML 1.0 Trait
responses:
    200:
      body:
        application/json:
          type: <<typeName>>
          example: !include /examples/<<typeName>>.json
```

## 4。库

RAML `libraries`可用于模块化任意数量的`data types`、`security schemes`、`resource types`、`traits`、`annotations`及其组合。

### 4.1。定义一个库

虽然通常在外部文件中定义，然后引用为一个`include`，但是`library`也可以被内联定义。包含在外部文件中的`library`也可以引用其他`libraries`。

与常规的`include`或`typed fragment`不同，包含在外部文件中的`library`必须声明被定义的顶级元素名。

让我们将我们的`traits`部分重写为一个`library`文件:

```java
#%RAML 1.0 Library
# This is the file /libraries/traits.raml
usage: This library defines some basic traits
traits:
  hasRequestItem:
    usage: Use this trait for resources whose request body is a single item
    body:
      application/json:
        type: <<typeName>>
  hasResponseItem:
    usage: Use this trait for resources whose response body is a single item
    responses:
        200:
          body:
            application/json:
              type: <<typeName>>
              example: !include /examples/<<typeName>>.json
```

### 4.2。应用库

`Libraries`通过顶层的`uses`属性应用，其值是一个或多个对象，这些对象的属性名是`library`名，其属性值构成了`libraries`的内容。

一旦我们为`security schemes`、`data types`、`resource types`和`traits`创建了`libraries`，我们就可以将`libraries`应用到根 RAML 文件:

```java
#%RAML 1.0
title: Baeldung Foo REST Services API
uses:
  mySecuritySchemes: !include libraries/security.raml
  myDataTypes: !include libraries/dataTypes.raml
  myResourceTypes: !include libraries/resourceTypes.raml
  myTraits: !include libraries/traits.raml
```

### 4.3。引用库

通过连接`library`名称、点(.)，以及被引用的元素的名称(例如，数据类型、资源类型、特征等)。

你可能还记得[我们之前的文章](/web/20220813052709/https://www.baeldung.com/simple-raml-with-resource-types-and-traits)中，我们是如何使用我们已经定义的`traits`来重构我们的`resource types`的。以下示例显示了如何将我们的" item" `resource type`重写为`library,`，如何将`traits` `library`文件(如上所示)包含在新的`library`中，以及如何通过在`trait`名称前面加上它们的`library`名称限定符("`myTraits`")来引用`traits`:

```java
#%RAML 1.0 Library
# This is the file /libraries/resourceTypes.raml
usage: This library defines the resource types for the API
uses:
  myTraits: !include traits.raml
resourceTypes:
  item:
    usage: Use this resourceType to represent any single item
    description: A single <<typeName>>
    get:
      description: Get a <<typeName>> by <<resourcePathName>>
      is: [ myTraits.hasResponseItem, myTraits.hasNotFound ]
    put:
      description: Update a <<typeName>> by <<resourcePathName>>
      is: [ myTraits.hasRequestItem, myTraits.hasResponseItem, myTraits.hasNotFound ]
    delete:
      description: Delete a <<typeName>> by <<resourcePathName>>
      is: [ myTraits.hasNotFound ]
      responses:
        204:
```

## 5。覆盖和扩展

`Overlays`和`extensions`是在用于扩展 API 的外部文件中定义的模块。`An overlay`用于扩展 API 的非行为方面，例如描述、使用说明和用户文档项目，而`extension`用于扩展或覆盖 API 的行为方面。

与被其他 RAML 文件引用的`includes`不同，所有的`overlay`和`extension`文件都必须包含对其主文件的引用(通过顶级的`masterRef`属性),主文件可以是有效的 RAML API 定义，也可以是要应用它们的另一个`overlay`或`extension`文件。

### 5.1。定义

覆盖或扩展文件的第一行必须格式化如下:

```java
RAML 1.0 Overlay 
```

覆盖文件的第一行必须采用类似的格式:

```java
RAML 1.0 Extension 
```

### 5.2。使用限制

当使用一组`overlays`和/或`extensions`时，它们都必须引用同一个主 RAML 文件。此外，RAML 处理工具通常期望根 RAML 文件和所有的`overlay`和`extension`文件具有共同的文件扩展名(例如。raml”)。

### 5.3。覆盖图的使用案例

`overlays`背后的动机是提供一种将接口与实现分离的机制，从而允许 RAML 定义中更面向人的部分更频繁地变化或增长，而 API 的核心行为方面保持稳定。

`overlays`的一个常见用例是以多种语言提供用户文档和其他描述性元素。让我们重写 API 的标题，并添加一些用户文档:

```java
#%RAML 1.0
title: API for REST Services used in the RAML tutorials on Baeldung.com
documentation:
  - title: Overview
  - content: |
      This document defines the interface for the REST services
      used in the popular RAML Tutorial series at Baeldung.com.
  - title: Copyright
  - content: Copyright 2016 by Baeldung.com. All rights reserved.
```

以下是我们如何为该部分定义西班牙语覆盖图:

```java
#%RAML 1.0 Overlay
# File located at (archivo situado en):
# /overlays/es_ES/documentationItems.raml
masterRef: /api.raml
usage: |
  To provide user documentation and other descriptive text in Spanish
  (Para proporcionar la documentación del usuario y otro texto descriptivo
  en español)
title: |
  API para servicios REST utilizados en los tutoriales RAML
  en Baeldung.com
documentation:
  - title: Descripción general
  - content: |
      Este documento define la interfaz para los servicios REST
      utilizados en la popular serie de RAML Tutorial en Baeldung.com.
  - title: Derechos de autor
  - content: |
      Derechos de autor 2016 por Baeldung.com.
      Todos los derechos reservados.
```

`overlays`的另一个常见用例是具体化`annotation`元数据，这本质上是一种向 API 添加非标准结构的方式，以便为 RAML 处理器(如测试和监控工具)提供挂钩。

### 5.4。扩展的用例

正如你可以从名字中推断出的，`extensions`用于通过添加新的行为和/或修改 API 的现有行为来扩展 API。与面向对象编程世界类似，子类扩展了超类，子类可以添加新方法和/或覆盖现有方法。扩展也可以扩展 API 的非功能方面。

例如，可以使用一个`extension`来定义额外的资源，这些资源只对一组选定的用户公开，比如管理员或被分配了特定角色的用户。一个`extension`也可以用来为一个新版本的 API 添加特性。

下面是一个`extension`,它覆盖了我们的 API 版本，并添加了以前版本中不可用的资源:

```java
#%RAML 1.0 Extension
# File located at:
# /extensions/en_US/additionalResources.raml
masterRef: /api.raml
usage: This extension defines additional resources for version 2 of the API.
version: v2
/foos:
  /bar/{barId}:
    get:
      description: |
        Get the foo that is related to the bar having barId = {barId}
      typeName: Foo
      queryParameters:
        barId?: integer
        typeName: Foo
        is: [ hasResponseItem ]
```

这里有一个西班牙语的`overlay`来表示这个`extension`:

```java
#%RAML 1.0 Overlay
# Archivo situado en:
# /overlays/es_ES/additionalResources.raml
masterRef: /api.raml
usage: |
  Se trata de un español demasiado que describe los recursos adicionales
  para la versión 2 del API.
version: v2
/foos:
  /bar/{barId}:
    get:
      description: |
        Obtener el foo que se relaciona con el bar tomando barId = {barId}
```

这里值得注意的是，虽然我们在这个例子中使用了一个`overlay`用于西班牙语重写，因为它没有修改 API 的任何行为，但是我们也可以很容易地将这个模块定义为一个`extension`。它可能更适合被定义为一个`extension`，因为它的目的是覆盖在它上面的英语`extension`中发现的属性。

## 6。结论

在本教程中，我们介绍了几种技术，通过将常见的构造分离到外部文件中，使 RAML API 定义更加模块化。

首先，我们展示了如何使用 RAML 中的`include`特性将单个复杂的属性值重构为可重用的外部文件模块，称为`typed fragments`。接下来，我们演示了一种使用`include`特性将某些元素集合具体化为可重用的`libraries`的方法。最后，我们通过使用`overlays`和`extensions`扩展了 API 的一些行为和非行为方面。

要了解更多关于 RAML 模块化技术的知识，请访问 RAML 1.0 规范。

你可以在[github 项目](https://web.archive.org/web/20220813052709/https://github.com/eugenp/tutorials/tree/master/raml/modularization)中查看本教程使用的 API 定义的**完整实现**。

Next **»**[Define Custom RAML Properties Using Annotations](/web/20220813052709/https://www.baeldung.com/raml-custom-properties-with-annotations)**«** Previous[Eliminate Redundancies in RAML with Resource Types and Traits](/web/20220813052709/https://www.baeldung.com/simple-raml-with-resource-types-and-traits)