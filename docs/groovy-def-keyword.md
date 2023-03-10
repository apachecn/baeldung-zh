# Groovy def 关键字

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/groovy-def-keyword>

## 1.概观

在这个快速教程中，我们将探索 [Groovy](/web/20220701012345/https://www.baeldung.com/groovy-language) 中的`def`关键字的概念。它为这种动态 JVM 语言提供了可选的类型特性。

## 2.`def`关键字的含义

**`def`关键字用于定义 Groovy 中的非类型化变量或函数，因为它是一种可选类型的语言。**

当我们不确定变量或字段的类型时，我们可以利用`def`让 Groovy 在运行时根据赋值决定类型:

```java
def firstName = "Samwell"  
def listOfCountries = ['USA', 'UK', 'FRANCE', 'INDIA'] 
```

在这里，`firstName`将是一个`String`，`listOfCountries`将是一个`ArrayList`。

我们还可以使用`def`关键字来定义方法的返回类型:

```java
def multiply(x, y) {
    return x*y
}
```

在这里，`multiply`可以返回任何类型的对象，这取决于我们传递给它的参数。

## 3.`def`变量

让我们了解一下`def`对于变量是如何工作的。

当我们使用`def`声明一个变量时，Groovy 将其声明为一个 [`NullObject`](https://web.archive.org/web/20220701012345/http://docs.groovy-lang.org/docs/groovy-2.3.2/html/api/org/codehaus/groovy/runtime/NullObject.html) ，并为其赋一个`null`值:

```java
def list
assert list.getClass() == org.codehaus.groovy.runtime.NullObject
assert list.is(null) 
```

当我们给`list`赋值时，Groovy 根据赋值定义它的类型:

```java
list = [1,2,4]
assert list instanceof ArrayList 
```

假设我们希望我们的变量类型是动态的，并随着赋值而变化:

```java
int rate = 20
rate = [12] // GroovyCastException
rate = "nill" // GroovyCastException
```

我们不能将`List`或`String`赋给`int`类型的变量，因为**这将抛出运行时异常**。

因此，为了克服这个问题并调用 Groovy 的动态特性，我们将使用`def`关键字:

```java
def rate
assert rate == null
assert rate.getClass() == org.codehaus.groovy.runtime.NullObject

rate = 12
assert rate instanceof Integer

rate = "Not Available"
assert rate instanceof String

rate = [1, 4]
assert rate instanceof List
```

## 4.`def`方法

`def`关键字进一步用于定义方法的动态返回类型。当一个方法可以有不同类型的返回值时，这是很方便的:

```java
def divide(int x, int y) {
    if (y == 0) {
        return "Should not divide by 0"
    } else {
        return x/y
    }
}

assert divide(12, 3) instanceof BigDecimal
assert divide(1, 0) instanceof String
```

我们还可以使用`def`来定义一个没有显式返回的方法:

```java
def greetMsg() {
    println "Hello! I am Groovy"
}
```

## 5.`def`对比类型

让我们讨论一些关于使用`def`的最佳实践。

虽然我们在声明变量时可能同时使用了`def`和 type:

```java
def int count
assert count instanceof Integer
```

**`def`关键字在那里是多余的，所以我们应该使用`def`或者一个类型。**

此外，我们应该**避免在方法中使用`def`作为非类型化参数**。

因此，代替:

```java
void multiply(def x, def y)
```

我们应该更喜欢:

```java
void multiply(x, y)
```

此外，我们应该**在定义构造函数时避免使用`def`。**

## 6.Groovy `def` vs. Java `Object`

由于我们已经通过例子看到了`def`关键字的大部分特性及其用法，我们可能想知道它是否类似于在 Java 中使用`Object`类来声明一些东西。是的， **`def`可以认为类似于** `**Object**`:

```java
def fullName = "Norman Lewis"
```

同样，我们可以在 Java 中使用`Object`:

```java
Object fullName = "Norman Lewis";
```

## 7.`def`对`@TypeChecked`

由于我们中的许多人来自严格类型语言的世界，我们可能想知道如何在 Groovy 中强制进行编译时类型检查。我们可以使用`@TypeChecked`注释轻松实现这一点。

例如，我们可以对一个类使用`@TypeChecked`来对它的所有方法和属性进行类型检查:

```java
@TypeChecked
class DefUnitTest extends GroovyTestCase {

    def multiply(x, y) {
        return x * y
    }

    int divide(int x, int y) {
        return x / y
    }
}
```

这里，`DefUnitTest`类将被进行类型检查，**编译将会失败，因为`multiply`方法未被类型化**。Groovy 编译器将显示一个错误:

```java
[Static type checking] - Cannot find matching method java.lang.Object#multiply(java.lang.Object).
Please check if the declared type is correct and if the method exists.
```

所以，**要忽略一个方法，我们可以用`TypeCheckingMode.SKIP` :**

```java
@TypeChecked(TypeCheckingMode.SKIP)
def multiply(x, y)
```

## 8.结论

在这个快速教程中，我们看到了如何使用`def`关键字来调用 Groovy 语言的动态特性，并让它在运行时确定变量和方法的类型。

这个关键字可以方便地编写动态和健壮的代码。

像往常一样，本教程的代码实现可以在 [GitHub](https://web.archive.org/web/20220701012345/https://github.com/eugenp/tutorials/tree/master/core-groovy-modules/core-groovy-2) 项目中获得。