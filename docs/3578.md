# OpenAPI 文件中的日期

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/openapi-dates>

## 1.介绍

在本教程中，我们将看到如何在 OpenAPI 文件中声明日期，在本例中，是用 [Swagger](/web/20221129000635/https://www.baeldung.com/spring-rest-openapi-documentation) 实现的。这将允许我们在调用外部 API 时以标准化的方式管理输入和输出日期。

## 2.Swagger vs. OAS

Swagger 是一组实现 [**OpenAPI 规范**](https://web.archive.org/web/20221129000635/http://spec.openapis.org/oas/v3.0.3) (OAS)的工具，这是一个与语言无关的接口，用于记录 RESTful APIs。这允许我们在不访问源代码的情况下理解任何服务的能力。

为了实现这一点，我们的项目中会有一个文件，通常是描述使用 OAS 的 API 的 YAML 或 JSON `,`。然后，我们将使用 Swagger 工具来:

*   通过浏览器(Swagger 编辑器)编辑我们的规范
*   自动生成 API 客户端库(Swagger Codegen)
*   显示自动生成的文档(Swagger UI)

[OpenAPI 文件示例](https://web.archive.org/web/20221129000635/https://swagger.io/docs/specification/basic-structure/)包含不同的部分，但是我们将关注模型定义。

## 3.定义日期

让我们使用 OAS 定义一个`User`实体:

```
components:
  User:
    type: "object"
    properties:
      id:
        type: integer
        format: int64
      createdAt:
        type: string
        format: date
        description: Creation date
        example: "2021-01-30"
      username:
        type: string
```

为了定义一个日期，我们使用一个对象:

*   `type`字段等于`string`
*   指定日期形成方式的`format`字段

在这种情况下，我们使用`date format`来描述`createdAt`日期。此格式使用[T2](https://web.archive.org/web/20221129000635/https://xml2rfc.tools.ietf.org/public/rfc/html/rfc3339.html#anchor14)ISO 8601 格式描述日期。

## 4.定义日期时间

此外，如果我们还想指定时间，我们将使用`date-time` 作为格式`.`让我们看一个例子:

```
createdAt:
  type: string
  format: date-time
  description: Creation date and time
  example: "2021-01-30T08:30:00Z"
```

在本例中，我们使用[`full-`*时间*](https://web.archive.org/web/20221129000635/https://xml2rfc.tools.ietf.org/public/rfc/html/rfc3339.html#anchor14) 格式来描述日期时间。

## 5.`pattern`字段

使用 OAS，我们也可以用其他格式描述日期。为此，让我们使用`pattern`字段:

```
customDate: 
  type: string 
  pattern: '^\d{4}(0[1-9]|1[012])(0[1-9]|[12][0-9]|3[01])

显然，这是可读性较差的方法，但却是最有效的方法。事实上，我们可以在这个领域使用任何正则表达式。

## 6.结论

在本文中，我们看到了如何使用 OpenAPI 声明日期。我们可以使用 OpenAPI 提供的标准格式以及定制模式来满足我们的需求。和往常一样，我们使用的例子的源代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20221129000635/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-rest-http)
  description: Custom date 
  example: "20210130"
```

显然，这是可读性较差的方法，但却是最有效的方法。事实上，我们可以在这个领域使用任何正则表达式。

## 6.结论

在本文中，我们看到了如何使用 OpenAPI 声明日期。我们可以使用 OpenAPI 提供的标准格式以及定制模式来满足我们的需求。和往常一样，我们使用的例子的源代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20221129000635/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-rest-http)