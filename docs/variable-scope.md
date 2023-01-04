# Groovy 变量范围

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/groovy/variable-scope>

## 1.介绍

在本教程中，我们将讨论如何使用**不同的 Groovy 作用域**，并看看我们如何利用它的变量作用域

## 2.属国

贯穿全文，我们将使用 [`groovy-all`](https://web.archive.org/web/20220626071732/https://search.maven.org/artifact/org.codehaus.groovy/groovy-all) 和 [`spock-core`](https://web.archive.org/web/20220626071732/https://search.maven.org/artifact/org.spockframework/spock-core) 的依赖关系

```
dependencies {
    compile 'org.codehaus.groovy:groovy-all:2.4.13'
    testCompile 'org.spockframework:spock-core:1.1-groovy-2.4'
}
```

## 3.所有范围

Groovy 中的作用域首先遵循这样的规则:默认情况下，**所有变量都是公共的。这意味着，除非指定，否则我们将能够从代码中的任何其他范围访问我们创建的任何变量。**

我们将看到这些作用域意味着什么，为了测试这一点，我们将**运行 Groovy 脚本**。要运行脚本，我们只需运行:

```
groovy <scriptname>.groovy
```

### 3.1.全局变量

在 Groovy 脚本中创建全局变量最简单的方法是在脚本中的任何地方分配它，不需要任何特殊的关键字。我们甚至不需要定义类型:

```
x = 200
```

然后，如果我们运行下面的 groovy 脚本:

```
x = 200
logger = Logger.getLogger("Scopes.groovy")
logger.info("- Global variable")
logger.info(x.toString())
```

我们会看到，我们可以从全局范围到达我们的变量。

### 3.2.从函数范围访问全局变量

另一种访问全局变量的方法是使用函数作用域:

```
def getGlobalResult() { 
   return 1 + x
}
```

这个函数是在作用域脚本中定义的。我们给全局变量`x`加 1。

如果我们运行以下脚本:

```
x = 200
logger = Logger.getLogger("Scopes.groovy")

def getGlobalResult() {
    logger.info(x.toString())
    return 1 + x
}

logger.info("- Access global variable from inside function")
logger.info(getGlobalResult().toString())
```

结果我们会得到 201。这证明了我们可以从函数的局部范围访问全局变量。

### 3.3.从函数范围创建全局变量

我们也可以在函数范围内创建全局变量。在这个局部范围内，如果我们在创建变量时不使用任何关键字，我们将在全局范围内创建它。然后，让我们在一个新函数中创建一个全局变量`z`:

```
def defineGlobalVariable() {
    z = 234
} 
```

并尝试通过运行以下脚本来访问它:

```
logger = Logger.getLogger("Scopes.groovy")

def defineGlobalVariable() {
    z = 234
    logger = Logger.getLogger("Scopes.groovy")
    logger.info(z.toString())
}

logger.info("- function called to create variable")
defineGlobalVariable()
logger.info("- Variable created inside a function")
logger.info(z.toString())
```

我们将看到我们可以从全局范围访问`z`。所以这最终证明了我们的变量已经在全局范围内创建了。

### 3.4.非全局变量

在非全局变量的情况下，我们将快速查看一下只为局部范围创建的变量。

具体来说，我们将关注关键字`def`。这样，我们将这个变量定义为线程运行范围的一部分**。**

所以，让我们试着定义一个全局变量`y`和一个函数局部变量:

```
logger = Logger.getLogger("ScopesFail.groovy")

y = 2

def fLocal() {
    def q = 333
    println(q)
    q
}

fLocal()

logger.info("- Local variable doesn't exist outside")
logger.info(q.toString()) 
```

如果我们运行这个脚本，它将会失败。失败的原因是`q`是一个局部变量，属于函数`fLocal`的作用域。由于我们用关键字`def`、**创建了`q`，我们将无法通过全局范围访问**。

显然，我们可以使用`fLocal `函数访问 q:

```
logger = Logger.getLogger("ScopesFail.groovy")

y = 2

def fLocal() {
    def q = 333
    println(q)
    q
}

fLocal()

logger.info("- Value of the created variable")
logger.info(fLocal())
```

所以现在我们可以看到，即使我们已经创建了一个`q `变量，该变量在其他范围内不再可用。如果我们再次调用`fLocal`，我们将创建一个新的变量。

## 4.结论

在本文中，我们看到了 Groovy 作用域是如何创建的，以及如何从代码的不同区域访问它们。

与往常一样，GitHub 上的[提供了示例的完整源代码。](https://web.archive.org/web/20220626071732/https://github.com/eugenp/tutorials/tree/master/core-groovy-modules/core-groovy)