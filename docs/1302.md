# JSONForms 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/introduction-to-jsonforms>

## 1。概述

在本系列的[第一篇文章](/web/20220812051139/https://www.baeldung.com/introduction-to-json-schema-in-java)中，我们介绍了`**JSON Schema**`的概念以及如何使用它来验证`**JSON Object**`的格式和结构。

在本文中，我们将看到如何利用`JSON`和`JSON Schema.`的功能来构建基于表单的 web 用户界面

为了实现我们的目标，我们将使用一个叫做 **[JSON Forms](https://web.archive.org/web/20220812051139/https://github.com/eclipsesource/jsonforms)** 的框架。它消除了为数据绑定手工编写 HTML 模板和 Javascript 来创建可定制表单的需要。

然后用一个 UI 框架呈现表单，目前是基于 AngularJS 的。

## 2。JSON 表单的组件

为了创建我们的表单，我们需要定义两个主要组件。

第一个组件是 **`data schema`** 定义了要在 UI 中显示的底层数据(对象类型及其属性)。

在这种情况下，我们可以使用在[上一篇文章](/web/20220812051139/https://www.baeldung.com/introduction-to-json-schema-in-java)中使用的`JSON Schema`来描述数据对象“产品”:

```
{
    "$schema": "http://json-schema.org/draft-04/schema#",
    "title": "Product",
    "description": "A product from the catalog",
    "type": "object",
    "properties": {
        "id": {
            "description": "The unique identifier for a product",
            "type": "integer"
        },
        "name": {
            "description": "Name of the product",
            "type": "string"
        },
        "price": {
            "type": "number",
            "minimum": 0,
            "exclusiveMinimum": true
        }
    },
    "required": ["id", "name", "price"]
}
```

我们可以看到，`JSON Object`显示了三个属性，分别是`id`、`name`和`price`。它们中的每一个都将是一个由名称标记的表单域。

每个属性也有一些属性。框架会将`type`属性翻译为输入字段的`type`。

price 属性特有的属性`minimum`、`exclusiveMinimum`告诉框架，在表单验证时，输入字段的值必须大于 0。

最后，`required`属性， 包含了之前定义的 的 所有属性 指定所有表单字段都需要填写。

第二个组件是 **`UI schema`** 描述了表单的布局以及`data schema`的哪些属性将被呈现为控件:

```
{
    "type": "HorizontalLayout",
    "elements": [
        {
            "type": "Control",
            "scope": { "$ref": "#/properties/id" }
        },
        {
            "type": "Control",
            "scope": { "$ref": "#/properties/name" }
        },
        {
            "type": "Control",
            "scope": { "$ref": "#/properties/price" }
        },
    ]
} 
```

**类型**属性定义为表单字段将在表单中排序。在这种情况下，我们选择了水平方式。

另外，`UI Schema`定义了数据模式的哪个属性将显示为表单字段。这是通过在**元素**数组中定义 元素**控制**得到的。

最后， `Controls`通过**作用域**属性`,`直接引用`data schema` ，这样数据属性的规范，比如它们的数据类型，就不必被复制。

## 3。在 AngularJS 中使用 JSON 表单

创建的`data schema`和`UI schema`在运行时被解释，也就是当包含表单的网页显示在浏览器、 上并被翻译成基于 AngularJS 的 UI 时，该 UI 已经具有包括数据绑定、验证等在内的全部功能。

现在，让我们看看如何在基于 AngularJS 的 web 应用程序中嵌入 JSON 表单。

## 3.1。设置项目

作为设置我们项目的先决条件，我们需要在我们的机器上安装`node.js`。如果你之前没有安装它，你可以按照[官方网站](https://web.archive.org/web/20220812051139/https://nodejs.org/)上的说明进行操作。

`node.js`还附带了`npm`，它是用来安装 JSON 表单库和其他所需依赖项的包管理器。

一旦安装了`node.js`并从 [GitHub](https://web.archive.org/web/20220812051139/https://github.com/eugenp/tutorials/tree/master/json-modules/json) 中克隆了实例后，打开一个 shell 并把 cd 放到`webapp`文件夹中。这个文件夹包含了`package.json`文件。它显示了项目的一些信息，主要是告诉`npm`它必须下载哪些依赖项。`package,json`文件如下: 

```
{
    "name": "jsonforms-intro",
    "description": "Introduction to JSONForms",
    "version": "0.0.1",
    "license": "MIT",
    "dependencies": {
         "typings": "0.6.5",
         "jsonforms": "0.0.19",
         "bootstrap": "3.3.6"
     }
}
```

T3 现在，我们可以键入`npm install` 命令了。这将开始下载所有需要的库。下载后，我们可以在`node_modules`文件夹中找到这些库。 

要了解更多细节，你可以参考国家预防机制的页面。

## 4。定义视图

现在我们已经有了所有需要的库和依赖项， 让我们定义一个显示表单的 html 页面。

在我们的页面中，我们需要导入`jsonforms.js`库，并使用专用的 AngularJS 指令`jsonforms`嵌入表单:

```
<!DOCTYPE html>
<html ng-app="jsonforms-intro">
<head>
    <title>Introduction to JSONForms</title>
    <script src="node_modules/jsonforms/dist/jsonforms.js" 
      type="text/javascript"></script>
    <script src="js/app.js" type="text/javascript"></script>
    <script src="js/schema.js" type="text/javascript"></script>
    <script src="js/ui-schema.js" type="text/javascript"></script>
</head>
<body>
    <div class="container" ng-controller="MyController">
        <div class="row" id="demo">
            <div class="col-sm-12">
                <div class="panel-primary panel-default">
                    <div class="panel-heading">
                        <h3 class="panel-title"><strong>Introduction 
                          to JSONForms</strong></h3>
                    </div>
                    <div class="panel-body jsf">
                        Bound data: {{data}}
                        <jsonforms schema="schema" 
                          ui-schema="uiSchema" data="data"/>
                    </div>
                </div>
            </div>
        </div>
    </div>
</body>
</html> 
```

作为该指令的参数，我们需要指向`data schema and`、`UI schema defined above`，以及包含要显示的`data`的`JSON object`。

## 5。角度控制器

在 AngularJS 应用程序中，指令所需的值通常由控制器提供:

```
app.controller('MyController', ['$scope', 'Schema', 'UISchema', 
  function($scope, Schema, UISchema) {

    $scope.schema = Schema;
    $scope.uiSchema = UISchema;
    $scope.data = {
        "id": 1,
        "name": "Lampshade",
        "price": 1.85
    };
}]);
```

## 6。应用程序模块

接下来，我们需要在我们的应用程序模块中注入`jsonforms`:

```
var app = angular.module('jsonforms-intro', ['jsonforms']);
```

## 7。显示表单

如果我们用浏览器打开上面定义的 html 页面，我们可以看到第一个 JSONForm:

[![JSONForms](img/951fd35e01542211e98965591beac9aa.png)](/web/20220812051139/https://www.baeldung.com/wp-content/uploads/2016/08/JSONForms-1024x159.png)

## 8。结论

在本文 中，我们已经看到了 如何使用`JSONForms`库来构建 UI 表单。耦合了一个`data schema`和一个`UI schema`，它消除了为数据绑定手工编写 HTML 模板和 Javascript 的需要。

上面的例子可以在 [GitHub 项目](https://web.archive.org/web/20220812051139/https://github.com/eugenp/tutorials/tree/master/json-modules/json)中找到。