# Groovy 中的元编程

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/groovy-metaprogramming>

## 1.概观

Groovy 是一种动态且强大的 JVM 语言，它有许多特性，比如[闭包](/web/20220826103300/https://www.baeldung.com/groovy-closures)和[特征](/web/20220826103300/https://www.baeldung.com/groovy-traits)。

在本教程中，我们将探索 Groovy 中元编程的概念。

## 2.什么是元编程？

元编程是一种使用元数据编写程序来修改自身或另一个程序的编程技术。

在 Groovy 中，可以在运行时和编译时执行元编程。接下来，我们将探索这两种技术的一些显著特征。

## 3.运行时元编程

运行时元编程使我们能够改变类的现有属性和方法。同样，我们可以附加新的属性和方法；都是在运行时。

Groovy 提供了一些方法和属性，有助于在运行时改变类的行为。

### 3.1.`propertyMissing`

当我们试图访问一个 Groovy 类的未定义属性时，它会抛出一个`MissingPropertyException.`来避免异常，Groovy 提供了`propertyMissing`方法。

首先，让我们用一些属性编写一个`Employee`类:

```
class Employee {
    String firstName
    String lastName  
    int age
}
```

其次，我们将创建一个`Employee`对象，并尝试显示一个未定义的属性`address.`，因此，它将抛出`MissingPropertyException` :

```
Employee emp = new Employee(firstName: "Norman", lastName: "Lewis")
println emp.address 
```

```
groovy.lang.MissingPropertyException: No such property: 
address for class: com.baeldung.metaprogramming.Employee
```

**Groovy 提供了`propertyMissing`方法来捕捉丢失的属性请求。**因此，我们可以在运行时避免一个`MissingPropertyException`。

为了捕捉一个丢失的属性的 getter 方法调用，我们将用属性名的一个参数来定义它:

```
def propertyMissing(String propertyName) {
    "property '$propertyName' is not available"
}
```

```
assert emp.address == "property 'address' is not available"
```

此外，同一个方法可以将第二个参数作为属性值，以捕捉缺失属性的 setter 方法调用:

```
def propertyMissing(String propertyName, propertyValue) { 
    println "cannot set $propertyValue - property '$propertyName' is not available" 
}
```

### 3.2.`methodMissing`

`methodMissing` 方法类似于`propertyMissing`。然而，`methodMissing`拦截对任何缺失方法的调用，从而避免`MissingMethodException`。

让我们试着在一个`Employee`对象上调用`getFullName`方法。由于`getFullName`丢失，执行将在运行时抛出`MissingMethodException`:

```
try {
    emp.getFullName()
} catch (MissingMethodException e) {
    println "method is not defined"
}
```

因此，我们可以定义`methodMissing`，而不是将方法调用包装在`try-catch`中:

```
def methodMissing(String methodName, def methodArgs) {
    "method '$methodName' is not defined"
}
```

```
assert emp.getFullName() == "method 'getFullName' is not defined"
```

### 3.3.`ExpandoMetaClass`

**Groovy 在其所有的类中都提供了一个`metaClass`属性。**`metaClass`属性是指`ExpandoMetaClass`的一个实例。

[`ExpandoMetaClass`](https://web.archive.org/web/20220826103300/http://docs.groovy-lang.org/latest/html/api/groovy/lang/ExpandoMetaClass.html) 类提供了许多在运行时转换现有类的方法。例如，我们可以添加属性、方法或构造函数。

首先，让我们使用`metaClass`属性将缺少的`address`属性添加到`Employee`类中:

```
Employee.metaClass.address = ""
```

```
Employee emp = new Employee(firstName: "Norman", lastName: "Lewis", address: "US")
assert emp.address == "US"
```

接下来，让我们在运行时将缺少的`getFullName`方法添加到`Employee`类对象中:

```
emp.metaClass.getFullName = {
    "$lastName, $firstName"
}
```

```
assert emp.getFullName() == "Lewis, Norman"
```

类似地，我们可以在运行时向`Employee`类添加一个构造函数:

```
Employee.metaClass.constructor = { String firstName -> 
    new Employee(firstName: firstName) 
}
```

```
Employee norman = new Employee("Norman")
assert norman.firstName == "Norman"
assert norman.lastName == null
```

同样，我们可以使用`metaClass.static.`添加`static`方法

**`metaClass`属性不仅可以方便地修改用户定义的类，还可以在运行时修改现有的 Java 类。**

例如，让我们给`String`类添加一个`capitalize`方法:

```
String.metaClass.capitalize = { String str ->
    str.substring(0, 1).toUpperCase() + str.substring(1)
}
```

```
assert "norman".capitalize() == "Norman"
```

### 3.4.扩展ˌ扩张

一个扩展可以在运行时给一个类添加一个方法，并使它可以被全局访问。

扩展中定义的方法应该总是静态的，以`self`类对象作为第一个参数。

例如，让我们编写一个`BasicExtension`类来为`Employee`类添加一个`getYearOfBirth`方法:

```
class BasicExtensions {
    static int getYearOfBirth(Employee self) {
        return Year.now().value - self.age
    }
}
```

为了启用`BasicExtension` s，我们需要在项目的`META-INF/services`目录中添加配置文件。

因此，让我们添加具有以下配置的`org.codehaus.groovy.runtime.ExtensionModule`文件:

```
moduleName=core-groovy-2 
moduleVersion=1.0-SNAPSHOT 
extensionClasses=com.baeldung.metaprogramming.extension.BasicExtensions
```

让我们验证一下在`Employee`类中添加的`getYearOfBirth`方法:

```
def age = 28
def expectedYearOfBirth = Year.now() - age
Employee emp = new Employee(age: age)
assert emp.getYearOfBirth() == expectedYearOfBirth.value
```

类似地，要在一个类中添加`static`方法，我们需要定义一个单独的扩展类。

例如，让我们通过定义`StaticEmployeeExtension`类将`static`方法`getDefaultObj`添加到我们的`Employee`类中:

```
class StaticEmployeeExtension {
    static Employee getDefaultObj(Employee self) {
        return new Employee(firstName: "firstName", lastName: "lastName", age: 20)
    }
}
```

然后，我们通过向`ExtensionModule`文件添加以下配置来启用`StaticEmployeeExtension`:

```
staticExtensionClasses=com.baeldung.metaprogramming.extension.StaticEmployeeExtension
```

现在，我们所需要的就是在`Employee`类上测试我们的`static` `getDefaultObj`方法:

```
assert Employee.getDefaultObj().firstName == "firstName"
assert Employee.getDefaultObj().lastName == "lastName"
assert Employee.getDefaultObj().age == 20
```

类似地，**使用扩展，我们可以给预编译的 Java 类**添加一个方法，比如`Integer` 和`Long`:

```
public static void printCounter(Integer self) {
    while (self > 0) {
        println self
        self--
    }
    return self
}
assert 5.printCounter() == 0 
```

```
public static Long square(Long self) {
    return self*self
}
assert 40l.square() == 1600l 
```

## 4.编译时元编程

使用特定的注释，我们可以毫不费力地在编译时改变类结构。换句话说，**我们可以在编译**时使用注释来修改类的抽象语法树。

让我们讨论一些注释，它们在 Groovy 中可以很方便地减少样板代码。其中许多都可以在`groovy.transform`封装中获得。

如果我们仔细分析，我们会意识到一些注释提供了类似于 Java 的[项目 Lombok](/web/20220826103300/https://www.baeldung.com/intro-to-project-lombok) 的特性。

### 4.1.`@ToString`

`@ToString`注释在编译时将`toString`方法的默认实现添加到一个类中。我们所需要的就是给类添加注释。

例如，让我们将`@ToString`注释添加到我们的`Employee`类中:

```
@ToString
class Employee {
    long id
    String firstName
    String lastName
    int age
}
```

现在，我们将创建一个`Employee`类的对象，并验证由`toString`方法返回的字符串:

```
Employee employee = new Employee()
employee.id = 1
employee.firstName = "norman"
employee.lastName = "lewis"
employee.age = 28

assert employee.toString() == "com.baeldung.metaprogramming.Employee(1, norman, lewis, 28)"
```

我们也可以用`@ToString`来声明`excludes`、`includes`、`includePackage`、`ignoreNulls`等参数来修改输出字符串。

例如，让我们从 Employee 对象的字符串中排除`id`和`package`:

```
@ToString(includePackage=false, excludes=['id'])
```

```
assert employee.toString() == "Employee(norman, lewis, 28)"
```

### 4.2.`@TupleConstructor`

在 Groovy 中使用`@TupleConstructor` 在类中添加一个参数化的构造函数。该注释为每个属性创建一个带参数的构造函数。

例如，让我们将`@TupleConstructor` 添加到`Employee`类中:

```
@TupleConstructor 
class Employee { 
    long id 
    String firstName 
    String lastName 
    int age 
}
```

现在，我们可以创建`Employee`对象，按照类中定义的属性顺序传递参数。

```
Employee norman = new Employee(1, "norman", "lewis", 28)
assert norman.toString() == "Employee(norman, lewis, 28)" 
```

如果我们在创建对象时没有为属性提供值，Groovy 将考虑默认值:

```
Employee snape = new Employee(2, "snape")
assert snape.toString() == "Employee(snape, null, 0)"
```

与`@ToString`类似，我们可以用`@TupleConstructor`声明`excludes`、`includes`和`includeSuperProperties`等参数，根据需要改变相关构造函数的行为。

### 4.3.`@EqualsAndHashCode`

我们可以使用`@EqualsAndHashCode`在编译时生成`equals`和`hashCode`方法的默认实现。

让我们通过将`@EqualsAndHashCode`添加到`Employee`类来验证它的行为:

```
Employee normanCopy = new Employee(1, "norman", "lewis", 28)

assert norman == normanCopy
assert norman.hashCode() == normanCopy.hashCode()
```

### 4.4.`@Canonical`

**`@Canonical`是`@ToString`、`@TupleConstructor`、`@EqualsAndHashCode` 注解**的组合。

只需添加它，我们就可以轻松地将这三者包含到一个 Groovy 类中。此外，我们可以用所有三个注释的任何特定参数来声明`@Canonical`。

### 4.5.`@AutoClone`

实现`Cloneable`接口的一个快速可靠的方法是添加`@AutoClone`注释。

让我们在将`@AutoClone`添加到`Employee`类后验证`clone`方法:

```
try {
    Employee norman = new Employee(1, "norman", "lewis", 28)
    def normanCopy = norman.clone()
    assert norman == normanCopy
} catch (CloneNotSupportedException e) {
    e.printStackTrace()
}
```

### 4.6.使用`@Log, @Commons, @Log4j, @Log4j2,` 和`@Slf4j`的测井支持

要为任何 Groovy 类添加日志支持，我们只需要在`groovy.util.logging`包中添加可用的注释。

让我们通过向`Employee`类添加`@Log`注释来启用 JDK 提供的日志记录。之后，我们将添加`logEmp`方法:

```
def logEmp() {
    log.info "Employee: $lastName, $firstName is of $age years age"
}
```

在`Employee`对象上调用`logEmp`方法将在控制台上显示日志:

```
Employee employee = new Employee(1, "Norman", "Lewis", 28)
employee.logEmp()
```

```
INFO: Employee: Lewis, Norman is of 28 years age
```

类似地，`@Commons` 注释可用于添加 Apache Commons 日志支持。`@Log4j`可用于 Apache Log4j 1.x 日志支持，`@Log4j2`可用于 [Apache Log4j 2.x](/web/20220826103300/https://www.baeldung.com/java-logging-intro) 。最后，使用`@Slf4j`为 Java 支持添加[简单日志门面。](/web/20220826103300/https://www.baeldung.com/slf4j-with-log4j2-logback)

## 5.结论

在本教程中，我们探索了 Groovy 中元编程的概念。

一路走来，我们已经看到了运行时和编译时的一些值得注意的元编程特性。

与此同时，我们探索了 Groovy 中额外可用的方便注释，以获得更整洁、更动态的代码。

像往常一样，本文的代码实现可以在 GitHub 上的[处获得。](https://web.archive.org/web/20220826103300/https://github.com/eugenp/tutorials/tree/master/core-groovy-modules/core-groovy-2)