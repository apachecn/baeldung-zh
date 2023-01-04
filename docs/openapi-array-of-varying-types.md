# 在 OpenAPI 中定义不同类型的数组

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/openapi-array-of-varying-types>

## 1.概观

OpenAPI 规范，以前被称为 Swagger 规范，有助于以一种标准化的、机器可读的方式描述 API。

在本教程中，我们将学习 **如何使用 OpenAPI 规范定义不同类型的数组。**在整篇文章中，我们将使用 OpenAPI v3 的特性。

## 2.定义不同类型的数组

首先，让我们定义我们将在整篇文章中使用的例子。我们假设我们想要定义一个包含以下两个对象的数组，分别代表一只狗和一只狮子:

```java
#dog
type: object
properties:
  barks:
    type: boolean
  likesSticks:
    type: boolean
#lion
type: object
properties:
  roars:
    type: boolean
  likesMeat:
    type: boolean
```

**有三种方法可以定义包含这两个对象的数组:关键字`oneOf`、`anyOf,`和任意类型模式。**

### 2.1.`oneOf`关键字

**`oneOf` 关键字指定数组只能包含一组预定义类型中的一种类型**:

```java
type: array
items:
  oneOf:
    - $ref: '#/components/schemas/Dog'
    - $ref: '#/components/schemas/Lion'
```

以下数组是上述定义的有效示例:

```java
{
  "dogs": [
    {
      "barks": true,
      "likesSticks": true
    },
    {
      "barks": false,
      "likesSticks": true
    }
  ]
}
```

另一方面，狗和狮子的混合是不允许的:

```java
{
  "dogsAndLions": [
    {
      "barks": true,
      "likesSticks": true
    },
    {
      "barks": false,
      "likesSticks": true
    },
    {
      "roars": true,
      "likesMeat": true
    }
  ]
}
```

### 2.2.`anyOf`关键字

**`anyOf`关键字指定数组可以包含预定义类型的任意组合。**这意味着只有狗，或者只有狮子，或者狗和狮子都可以组成一个有效的数组:

```java
type: array
items:
  anyOf:
    - $ref: '#/components/schemas/Dog'
    - $ref: '#/components/schemas/Lion'
```

下面的示例显示了三个都有效的数组:

```java
{
  "onlyDogs": [
    {
      "barks": true,
      "likesSticks": true
    },
    {
      "barks": false,
      "likesSticks": true
    }
  ],
  "onlyLions": [
    {
      "roars": true,
      "likesMeat": true
    },
    {
      "roars": true,
      "likesMeat": true
    }
  ],
  "dogsAndLions": [
    {
      "barks": true,
      "likesSticks": true
    },
    {
      "barks": false,
      "likesSticks": true
    },
    {
      "roars": true,
      "likesMeat": true
    }
  ]
} 
```

### 2.3.任意类型模式

**使用任意类型模式允许定义一个数组，该数组包含 OpenAPI 规范**支持的所有类型的混合。它还附带了一个由大括号'`{}`'组成的简便的速记语法:

```java
type: array
items: {}
```

让我们看看上面定义的显式语法:

```java
type: array
items:
  anyOf:
    - type: string
    - type: number
    - type: integer
    - type: boolean
    - type: array
      items: {}
    - type: object
```

现在让我们来看一个包含一个字符串、一个数字、一个整数、一个布尔值、一个数组和一个随机对象的示例数组:

```java
{
  "arbitraryStuff": [
    "Hello world",
    42.1,
    42,
    true,
    [{ "name": "Randy Random" }],
    { "name": "Robbi Random" }
  ]
}
```

## 3.结论

在本文中，我们学习了如何使用 OpenAPI 规范定义不同类型的数组。

首先，我们看到了如何对包含一组预定义类型中的一种类型的数组使用关键字`oneOf`。然后，我们讨论了如何用关键字`anyOf`定义一个包含多个预定义类型的数组。

最后，我们了解到可以使用任意类型模式来定义包含任意类型的数组。