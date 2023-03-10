# Groovy 语言简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/groovy-language>

## 1。概述

Groovy 是一种用于 JVM 的动态脚本语言。它编译成字节码，并与 Java 代码和库无缝融合。

在本文中，我们将看一下 [Groovy](https://web.archive.org/web/20221209002109/http://www.groovy-lang.org/) 的一些基本特性，包括基本语法、控制结构和集合。

然后我们将看看使它成为一门有吸引力的语言的一些主要特性，包括空安全、隐式真值、操作符和字符串。

## 2。环境

如果我们想在 Maven 项目中使用 Groovy，我们需要在`pom.xml:`中添加以下内容

```java
<build>
    <plugins>
        // ...
        <plugin>
            <groupId>org.codehaus.gmavenplus</groupId>
            <artifactId>gmavenplus-plugin</artifactId>
            <version>1.5</version>
       </plugin>
   </plugins>
</build>
<dependencies>
    // ...
    <dependency>
        <groupId>org.codehaus.groovy</groupId>
        <artifactId>groovy-all</artifactId>
        <version>2.4.10</version>
    </dependency>
</dependencies>
```

最新的 Maven 插件可以在这里找到[，最新版本的`groovy-all`](https://web.archive.org/web/20221209002109/https://mvnrepository.com/artifact/org.codehaus.gmavenplus/gmavenplus-plugin) [可以在这里](https://web.archive.org/web/20221209002109/https://mvnrepository.com/artifact/org.codehaus.groovy/groovy-all)找到。

## 3。基本特征

Groovy 中有许多有用的特性。现在，让我们看看这种语言的基本构件，以及它与 Java 的不同之处。

现在，让我们看看这种语言的基本构件，以及它与 Java 的不同之处。

### 3.1。动态打字

Groovy 最重要的特性之一是支持动态类型。

类型定义是可选的，实际类型在运行时确定。让我们来看看这两个类:

```java
class Duck {
    String getName() {
        'Duck'
    }
}
class Cat {
    String getName() {
        'Cat'
    }
} 
```

这两个类定义了同一个`getName` 方法，但是它没有在契约中明确定义。

现在，假设我们有一个包含鸭子和猫的对象列表，这些对象使用了`getName`方法。使用 Groovy，我们可以做到以下几点:

```java
Duck duck = new Duck()
Cat cat = new Cat()

def list = [duck, cat]
list.each { obj ->
    println obj.getName()
}
```

代码将会编译，上面代码的输出将会是:

```java
Duck
Cat
```

**3.2。隐式真值转换**

与 JavaScript 一样，Groovy 在需要时会将每个对象评估为布尔值，例如，当在`if`语句中使用它或者对值求反时:

```java
if("hello") {...}
if(15) {...}
if(someObject) {...}
```

关于这种转换，需要记住一些简单的规则:

*   非空的`Collections,` 数组，映射计算为`true`
*   至少有一个匹配的`Matcher`评估为`true`
*   具有更多元素的`Iterators`和`Enumerations`被强制为`true`
*   非空的`Strings`、`GStrings`和`CharSequences`，被强制为`true`
*   非零数字被评估为`true`
*   非空对象引用被强制为`true`

**如果我们想要定制隐式真值转换，我们可以定义我们的`asBoolean()`方法。**

### 3.3。进口

有些包是默认导入的，我们不需要显式导入它们:

```java
import java.lang.* 
import java.util.* 
import java.io.* 
import java.net.* 

import groovy.lang.* 
import groovy.util.* 

import java.math.BigInteger 
import java.math.BigDecimal
```

## 4。AST 转换

AST(**Abstract Syntax Tree**)transforms 允许我们挂钩到 Groovy 编译过程，并定制它以满足我们的需求。这是在编译时完成的，所以在运行应用程序时不会有性能损失。我们可以创建自己的 AST 转换，但是也可以使用内置的转换。

我们可以创建自己的转换，也可以从内置的转换中获益。

下面我们来看看一些值得了解的注解。

### 4.1。`TypeChecked`注解

该注释用于强制编译器对带注释的代码段进行严格的类型检查。类型检查机制是可扩展的，所以如果需要，我们甚至可以提供比 Java 更严格的类型检查。

让我们看看下面的例子:

```java
class Universe {
    @TypeChecked
    int answer() { "forty two" }
}
```

如果我们尝试编译这段代码，我们会观察到以下错误:

```java
[Static type checking] - Cannot return value of type java.lang.String on method returning type int
```

`@TypeChecked`注释可以应用于类和方法。

### 4.2。`CompileStatic`注解

这个注释允许编译器执行编译时检查，就像对 Java 代码一样。之后，编译器执行静态编译，从而绕过 Groovy 元对象协议。

当一个类被注释时，所有的方法、属性、文件、内部类等等。将进行类型检查。当一个方法被注释时，静态编译只应用于那些被该方法封闭的项目(闭包和匿名内部类)。

## 5。属性

在 Groovy 中，我们可以创建 POGOs(普通的 Groovy 对象),其工作方式与 Java 中的 POJOs 相同，尽管它们更紧凑，因为在编译期间会自动为公共属性生成**getter 和 setters。重要的是要记住，它们只有在尚未定义的情况下才会生成。**

这为我们提供了将属性定义为开放字段的灵活性，同时保留了在设置或获取值时覆盖行为的能力。

考虑这个对象:

```java
class Person {
    String name
    String lastName
}
```

因为类、字段和方法的默认范围是`public –` ，所以这是一个公共类，并且这两个字段是公共的。

编译器会将这些转换成私有字段，并添加`getName()`、`setName()`、`getLastName()`和`setLasfName()`方法。如果我们为一个特定的字段定义了`setter`和`getter`，编译器将不会创建一个公共方法。

### 5.1。快捷符号

Groovy 提供了获取和设置属性的快捷表示法。我们可以使用类似字段的访问符号，而不是用 Java 的方式调用 getters 和 setters:

```java
resourceGroup.getResourcePrototype().getName() == SERVER_TYPE_NAME
resourceGroup.resourcePrototype.name == SERVER_TYPE_NAME

resourcePrototype.setName("something")
resourcePrototype.name = "something"
```

## 6。操作员

现在让我们看看在普通 Java 中已知的操作符之上添加的新操作符。

### 6.1。空安全解引用

最流行的是空安全的解引用操作符 `“?”`，它允许我们在调用方法或访问`null`对象的属性时避免使用`NullPointerException` 。这在链式调用中特别有用，因为在链式调用中的某个点可能会出现一个`null`值。

例如，我们可以安全地调用:

```java
String name = person?.organization?.parent?.name
```

在上面的例子中，如果一个`person`、`person.organization`或`organization.parent` 是`null`，那么就返回`null`。

### 6.2。猫王操作员

Elvis 运算符 `“?:`"让我们可以精简三元表达式。这两个是等价的:

```java
String name = person.name ?: defaultName
```

和

```java
String name = person.name ? person.name : defaultName
```

如果名称变量是`Groovy true`(在这种情况下，不是`null`，而是长度为`non-zero`)，它们都将`person.name`的值赋给名称变量。

### 6.3。宇宙飞船操作员

宇宙飞船操作符`“<=>”`是一个关系操作符，它的表现类似于 Java 的`compareTo()`,它比较两个对象并根据两个参数的值返回-1、0 或+1。

如果左参数大于右参数，则运算符返回 1。如果左边的参数小于右边的参数，则运算符返回 1。如果参数相等，则返回 0。

使用比较操作符的最大好处是平滑地处理`nulls`，这样`x <=> y`就不会抛出`NullPointerException`:

```java
println 5 <=> null
```

上面的例子将打印 1 作为结果。

## 7。字符串

有多种方式来表达字符串文字。Java 中使用的方法(双引号字符串)是受支持的，但是如果愿意，也允许使用单引号。

也支持多行字符串，在其他语言中有时称为 heredocs，使用三重引号(单引号或双引号)。

也支持多行字符串，在其他语言中有时称为 heredocs，使用三重引号(单引号或双引号)。

用双引号定义的字符串支持使用 `${}`语法进行插值:

```java
def name = "Bill Gates"
def greeting = "Hello, ${name}"
```

事实上，任何表达式都可以放在`${}`里面:

```java
def name = "Bill Gates"
def greeting = "Hello, ${name.toUpperCase()}"
```

如果一个带双引号的字符串包含一个表达式`${}`，则称之为 GString，否则，它就是一个普通的`String` 对象。

下面的代码将运行，不会测试失败:

```java
def a = "hello" 
assert a.class.name == 'java.lang.String'

def b = 'hello'
assert b.class.name == 'java.lang.String'

def c = "${b}"
assert c.class.name == 'org.codehaus.groovy.runtime.GStringImpl'
```

## 8。收藏和地图

让我们来看看一些基本的数据结构是如何处理的。

### 8.1。`Lists`

下面是一些代码，用于向 Java 中的新实例`ArrayList` 添加一些元素:

```java
List<String> list = new ArrayList<>();
list.add("Hello");
list.add("World");
```

下面是 Groovy 中相同的操作:

```java
List list = ['Hello', 'World']
```

默认情况下，列表的类型为`java.util.ArrayList`，也可以通过调用相应的构造函数来显式声明。

对于一个`Set`没有单独的语法，但是我们可以使用类型`coercion`来实现。要么使用:

```java
Set greeting = ['Hello', 'World']
```

或者:

```java
def greeting = ['Hello', 'World'] as Set
```

### 8.2。`Map`

`Map`的语法是类似的，尽管有点冗长，因为我们需要能够指定用冒号分隔的键和值:

```java
def key = 'Key3'
def aMap = [
    'Key1': 'Value 1', 
    Key2: 'Value 2',
    (key): 'Another value'
]
```

在这个初始化之后，我们将获得一个新的`LinkedHashMap` ，其中包含条目:`Key1 -> Value1, Key2 -> Value 2, Key3 -> Another Value`。

我们可以通过多种方式访问地图中的条目:

```java
println aMap['Key1']
println aMap[key]
println aMap.Key1
```

## 9。控制结构

### 9.1。`if-else`条件句:

Groovy 支持预期的条件`if/else`语法:

```java
if (...) {
    // ...
} else if (...) {
    // ...
} else {
    // ...
} 
```

### 9.2。`switch-case`条件句:

`switch`语句向后兼容 Java 代码，这样我们就可以避免在多个匹配中共享相同代码的情况。

最重要的区别是`switch`可以针对多个不同的值类型执行匹配:

```java
def x = 1.23
def result = ""

switch ( x ) {
    case "foo":
        result = "found foo"
        break

    case "bar":
        result += "bar"
        break

    case [4, 5, 6, 'inList']:
        result = "list"
        break

    case 12..30:
        result = "range"
        break

    case Number:
        result = "number"
        break

    case ~/fo*/: 
        result = "foo regex"
        break

    case { it < 0 }: // or { x < 0 }
        result = "negative"
        break

    default:
        result = "default"
}

println(result)
```

上面的例子将打印`number.`

### 9.3。`while`循环:

Groovy 像 Java 一样支持常见的`while` 循环:

```java
def x = 0
def y = 5

while ( y-- > 0 ) {
    x++
}
```

### 9.4。`for`循环:

Groovy 接受这种简单性，并强烈鼓励`for`循环遵循这种结构:

```java
for (variable in iterable) { body }
```

`for`循环对`iterable`进行迭代。常用的可迭代对象是范围、集合、映射、数组、迭代器和枚举。事实上，任何对象都可以是可迭代的。

如果正文只包含一条语句，则正文两边的大括号是可选的。下面是迭代一个`range`、`list`、`array`、`map`和`strings`的例子:

```java
def x = 0
for ( i in 0..9 ) {
    x += i
}

x = 0
for ( i in [0, 1, 2, 3, 4] ) {
    x += i
}

def array = (0..4).toArray()
x = 0
for ( i in array ) {
    x += i
}

def map = ['abc':1, 'def':2, 'xyz':3]
x = 0
for ( e in map ) {
    x += e.value
}

x = 0
for ( v in map.values() ) {
    x += v
}

def text = "abc"
def list = []
for (c in text) {
    list.add(c)
}
```

对象迭代使得 Groovy `for-`循环成为复杂的控制结构。这是使用闭包迭代对象的有效方法，比如使用`Collection’s each`方法。

主要区别在于`for` 循环的主体不是闭包，这意味着这个主体是一个块:

```java
for (x in 0..9) { println x }
```

而这个身体是一个封闭体:

```java
(0..9).each { println it }
```

尽管它们看起来很相似，但在构造上却大不相同。

闭包是一个独立的对象，具有不同的特性。它可以在不同的地方构造，并传递给`each` 方法。然而，`for-`循环的主体在其出现点直接生成为`bytecode`。没有特殊的范围规则适用。

## 10。异常处理

最大的区别是没有强制执行检查异常处理。

为了处理一般的异常，我们可以将可能导致异常的代码放在一个`try/catch`块中:

```java
try {
    someActionThatWillThrowAnException()
} catch (e)
    // log the error message, and/or handle in some way
}
```

通过不声明我们捕获的异常的类型，任何异常都将在这里被捕获。

## 11。关闭

简单地说，闭包是一个匿名的可执行代码块，它可以传递给变量，并且可以访问定义它的上下文中的数据。

它们也类似于匿名内部类，尽管它们不实现接口或扩展基类。它们类似于 Java 中的 lambdas。

有趣的是，Groovy 可以充分利用为支持 lambdas 而引入的 JDK 附加功能，尤其是流式 API。我们总是可以在需要 lambda 表达式的地方使用闭包。

让我们考虑下面的例子:

```java
def helloWorld = {
    println "Hello World"
}
```

变量`helloWorld`现在保存了对闭包的引用，我们可以通过调用它的`call` 方法来执行它:

```java
helloWorld.call()
```

Groovy 让我们使用更自然的方法调用语法——它为我们调用了`call` 方法:

```java
helloWorld()
```

### 11.1。参数

像方法一样，闭包也可以有参数。有三种变体。

在后一个示例中，因为没有 declpersistence_startared，所以只有一个默认名称为`it`的参数。打印发送内容的修改后的闭包应该是:

```java
def printTheParam = { println it }
```

我们可以这样称呼它:

```java
printTheParam('hello')
printTheParam 'hello'
```

我们还可以在闭包中期待参数，并在调用时传递它们:

```java
def power = { int x, int y ->
    return Math.pow(x, y)
}
println power(2, 3)
```

参数的类型定义与变量相同。如果我们定义了一个类型，我们只能使用这个类型，但是我们也可以传递任何我们想要的东西:

```java
def say = { what ->
    println what
}
say "Hello World"
```

### 11.2。可选退货

闭包的最后一个语句可以隐式返回，而不需要编写 return 语句。这可以用来将样板代码减少到最少。因此，计算数字平方的闭包可以简化如下:

```java
def square = { it * it }
println square(4)
```

这个闭包使用了隐式参数`it`和可选的 return 语句。

## 12。 **结论**

本文简要介绍了 Groovy 语言及其主要特性。我们首先介绍一些简单的概念，比如基本语法、条件语句和操作符。我们还演示了一些更高级的特性，比如操作符和闭包。

如果你想找到更多关于这种语言及其语义的信息，你可以直接去[官方网站](https://web.archive.org/web/20221209002109/http://www.groovy-lang.org/semantics.html)。