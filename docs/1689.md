# Groovy 中的闭包

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/groovy-closures>

## 1.概观

在这篇介绍性教程中，我们将探索 [Groovy](/web/20220628153042/https://www.baeldung.com/groovy-language) 中闭包的概念，这是这种动态且强大的 JVM 语言的一个关键特性。

许多其他语言，包括 Javascript 和 Python，都支持闭包的概念。然而，闭包的特征和功能因语言而异。

我们将触及 Groovy 闭包的关键方面，展示如何使用它们的例子。

## 2.什么是终结？

一个[闭包](/web/20220628153042/https://www.baeldung.com/cs/closure)是一个匿名代码块。在 Groovy 中，它是 [`Closure`](https://web.archive.org/web/20220628153042/http://docs.groovy-lang.org/latest/html/api/groovy/lang/Closure.html) 类的一个**实例。闭包可以接受 0 个或更多参数，并且总是返回值。**

此外，闭包可能会访问其作用域之外的周围变量，并在执行过程中使用它们——连同其局部变量。

此外，我们可以将闭包赋给变量，或者将其作为参数传递给方法。因此，闭包提供了延迟执行的功能。

## 3.关闭声明

一个 Groovy 闭包包含参数、箭头- >和要执行的代码。参数是可选的，当提供时，用逗号分隔。

### 3.1.基本声明

```
def printWelcome = {
    println "Welcome to Closures!"
}
```

这里，闭包`printWelcome`在被调用时打印一条语句。现在，让我们写一个一元闭包的快速例子:

```
def print = { name ->
    println name 
}
```

这里，闭包`print`接受一个参数`name`，并在调用时打印出来。

**由于闭包的定义看起来类似于方法**，让我们来比较一下:

```
def formatToLowerCase(name) {
    return name.toLowerCase()
}
def formatToLowerCaseClosure = { name ->
    return name.toLowerCase()
} 
```

这里，方法和相应的闭包行为类似。然而，闭包和方法之间有细微的差别，我们将在后面的闭包和方法部分讨论。

### 3.2.执行

**我们可以用两种方式执行闭包——我们可以像调用其他方法一样调用它，或者我们可以使用`call`方法。**

例如，作为常规方法:

```
print("Hello! Closure")
formatToLowerCaseClosure("Hello! Closure") 
```

用`call`方法执行:

```
print.call("Hello! Closure") 
formatToLowerCaseClosure.call("Hello! Closure")
```

## 4.因素

Groovy 闭包的参数类似于常规方法的参数。

### 4.1.隐参数

我们可以定义一个没有参数的一元闭包，因为**当没有定义参数时，Groovy 会假设一个名为`it”`** 的隐式参数:

```
def greet = {
    return "Hello! ${it}"
}
assert greet("Alex") == "Hello! Alex"
```

### 4.2.多参数

这里有一个闭包，它接受两个参数并返回它们相乘的结果:

```
def multiply = { x, y -> 
    return x*y 
}
assert multiply(2, 4) == 8
```

### 4.3.参数类型

在到目前为止的例子中，我们的参数没有提供任何类型。我们还可以设置闭包参数的类型。例如，让我们重写`multiply`方法来考虑其他操作:

```
def calculate = {int x, int y, String operation ->
    def result = 0    
    switch(operation) {
        case "ADD":
            result = x+y
            break
        case "SUB":
            result = x-y
            break
        case "MUL":
            result = x*y
            break
        case "DIV":
            result = x/y
            break
    }
    return result
}
assert calculate(12, 4, "ADD") == 16
assert calculate(43, 8, "DIV") == 5.375
```

### 4.4.瓦拉格斯

我们可以在闭包中声明可变数量的参数，类似于常规方法。例如:

```
def addAll = { int... args ->
    return args.sum()
}
assert addAll(12, 10, 14) == 36
```

## 5.作为论点的终结

我们可以将一个`Closure`作为参数传递给一个常规的 Groovy 方法。这允许方法调用我们的闭包来完成它的任务，允许我们定制它的行为。

让我们讨论一个简单的用例:计算规则图形的体积。

在本例中，体积定义为面积乘以高度。但是，对于不同的形状，面积的计算会有所不同。

因此，我们将编写`volume`方法，该方法将闭包`areaCalculator`作为参数，我们将在调用期间传递面积计算的实现:

```
def volume(Closure areaCalculator, int... dimensions) {
    if(dimensions.size() == 3) {
        return areaCalculator(dimensions[0], dimensions[1]) * dimensions[2]
    } else if(dimensions.size() == 2) {
        return areaCalculator(dimensions[0]) * dimensions[1]
    } else if(dimensions.size() == 1) {
        return areaCalculator(dimensions[0]) * dimensions[0]
    }    
}
assert volume({ l, b -> return l*b }, 12, 6, 10) == 720 
```

让我们用同样的方法求一个圆锥体的体积:

```
assert volume({ radius -> return Math.PI*radius*radius/3 }, 5, 10) == Math.PI * 250
```

## 6.嵌套闭包

我们可以在闭包内声明和调用闭包。

例如，让我们给已经讨论过的`calculate`闭包添加一个日志功能:

```
def calculate = {int x, int y, String operation ->

    def log = {
        println "Performing $it"
    }

    def result = 0    
    switch(operation) {
        case "ADD":
            log("Addition")
            result = x+y
            break
        case "SUB":
            log("Subtraction")
            result = x-y
            break
        case "MUL":
            log("Multiplication")
            result = x*y
            break
        case "DIV":
            log("Division")
            result = x/y
            break
    }
    return result
}
```

## 7.字符串的惰性求值

Groovy `String`通常在创建时被评估和插入。例如:

```
def name = "Samwell"
def welcomeMsg = "Welcome! $name"

assert welcomeMsg == "Welcome! Samwell"
```

即使我们修改了变量 `name`的值，变量`welcomeMsg`也不会改变:

```
name = "Tarly"
assert welcomeMsg != "Welcome! Tarly"
```

**闭合插值允许我们提供`String` s** 的惰性评估，从它们周围的当前值重新计算。例如:

```
def fullName = "Tarly Samson"
def greetStr = "Hello! ${-> fullName}"

assert greetStr == "Hello! Tarly Samson"
```

只是这一次，更改变量也会影响插值字符串的值:

```
fullName = "Jon Smith"
assert greetStr == "Hello! Jon Smith"
```

## 8.集合中的闭包

Groovy 集合在其许多 API 中使用闭包。例如，让我们定义一个条目列表，并使用一元闭包`each`打印它们，它有一个隐式参数:

```
def list = [10, 11, 12, 13, 14, true, false, "BUNTHER"]

list.each {
    println it
}

assert [13, 14] == list.findAll{ it instanceof Integer && it >= 13 }
```

通常，基于某种标准，我们可能需要从地图中创建一个列表。例如:

```
def map = [1:10, 2:30, 4:5]

assert [10, 60, 20] == map.collect{it.key * it.value} 
```

## 9.闭包与方法

到目前为止，我们已经看到了闭包的语法、执行和参数，它们非常类似于方法。现在让我们比较一下闭包和方法。

与常规的 Groovy 方法不同:

*   我们可以将一个`Closure`作为参数传递给一个方法
*   一元闭包可以使用隐式的`it`参数
*   我们可以将一个`Closure`赋给一个变量，稍后执行它，或者作为一个方法，或者使用`call`
*   Groovy 在运行时确定闭包的返回类型
*   我们可以在闭包内部声明和调用闭包
*   闭包总是返回值

因此，闭包比常规方法有优势，是 Groovy 的一个强大特性。

## 10.结论

在本文中，我们看到了如何在 Groovy 中创建闭包，并探索了它们的使用方法。

闭包提供了一种将功能注入对象和方法以延迟执行的有效方式。

和往常一样，本文中的代码和单元测试可以在 GitHub 上的[处获得。](https://web.archive.org/web/20220628153042/https://github.com/eugenp/tutorials/tree/master/core-groovy-modules/core-groovy)