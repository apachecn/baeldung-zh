# Java“私有”访问修饰符

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-private-keyword>

## 1.概观

在 Java 编程语言中，字段、构造函数、方法和类可以用[访问修饰符](/web/20221128041817/https://www.baeldung.com/java-access-modifiers)来标记。在本教程中，我们将讨论 Java 中的`private`访问修饰符。

## 2.关键词

访问修饰符很重要，因为它允许封装和信息隐藏，这是面向对象编程的核心原则。封装负责捆绑方法和数据，而信息隐藏是封装的结果—它隐藏了对象的内部表示。

首先要记住的是，声明为`private`的**元素只能被声明为**的类访问。

## 3.菲尔茨

现在，我们将看到一些简单的代码示例来更好地理解这个主题。

首先，让我们创建一个包含几个`private`实例变量的`Employee` 类:

```java
public class Employee {
    private String privateId;
    private boolean manager;
    //...
}
```

在这个例子中，我们将`privateId`变量标记为`private`,因为我们想要为 id 生成添加一些逻辑。正如我们所见，我们对`manager`属性做了同样的事情，因为我们不想允许直接修改这个字段。

## 4.构造器

现在让我们创建一个`private`构造函数:

```java
private Employee(String id, String name, boolean managerAttribute) {
    this.name = name;
    this.privateId = id + "_ID-MANAGER";
}
```

通过将我们的构造函数标记为`private`，我们只能在类内部使用它。

**让我们添加一个`static`方法，这将是我们从`Employee`类:**外部使用这个`private`构造函数的唯一方式

```java
public static Employee buildManager(String id, String name) {
    return new Employee(id, name, true);
}
```

现在，我们可以通过简单地编写以下代码来获得我们的`Employee` 类的管理器实例:

```java
Employee manager = Employee.buildManager("123MAN","Bob");
```

当然，在幕后，`buildManager`方法调用我们的`private`构造函数。

## 5.方法

现在让我们给我们的类添加一个`private`方法:

```java
private void setManager(boolean manager) {
    this.manager = manager;
}
```

让我们假设，出于某种原因，我们公司有一条武断的规定，只有名为“卡尔”的员工才能被提升为经理，尽管其他班级并不知道这一点。我们将创建一个带有一些逻辑的`public`方法来处理这个调用我们的`private`方法的规则:

```java
public void elevateToManager() {
    if ("Carl".equals(this.name)) {
        setManager(true);
    }
}
```

## 6.`private`在行动

让我们看一个如何从外部使用我们的`Employee`类的例子:

```java
public class ExampleClass {

    public static void main(String[] args) {
        Employee employee = new Employee("Bob","ABC123");
        employee.setPrivateId("BCD234");
        System.out.println(employee.getPrivateId());
    }
}
```

在执行`ExampleClass`之后，我们将在控制台上看到它的输出:

```java
BCD234_ID
```

在这个例子中，我们使用了`public`构造函数和`public`方法`changeId(customId)`，因为我们不能直接访问`private`变量`privateId`。

让我们看看**如果我们试图从我们的`Employee`类外部访问一个`private`方法、构造函数或变量**会发生什么:

```java
public class ExampleClass {

    public static void main(String[] args) {
        Employee employee = new Employee("Bob","ABC123",true);
        employee.setManager(true);
        employee.privateId = "ABC234";
    }
}
```

**我们将得到每个非法语句的编译错误**:

```java
The constructor Employee(String, String, boolean) is not visible
The method setManager(boolean) from the type Employee is not visible
The field Employee.privateId is not visible
```

## 7.班级

有一个**特例，我们可以创建一个`private`类**——作为某个其他类的内部类。否则，如果我们将一个外部类声明为`private`，我们将禁止其他类访问它，使它变得无用:

```java
public class PublicOuterClass {

    public PrivateInnerClass getInnerClassInstance() {
        PrivateInnerClass myPrivateClassInstance = this.new PrivateInnerClass();
        myPrivateClassInstance.id = "ID1";
        myPrivateClassInstance.name = "Bob";
        return myPrivateClassInstance;
    }

    private class PrivateInnerClass {
        public String name;
        public String id;
    }
}
```

在这个例子中，我们通过指定*私有*访问修饰符在`PublicOuterClass `中创建了一个`private`内部类。

因为我们使用了`private`关键字，如果出于某种原因，**试图从`PublicOuterClass`** 之外实例化我们的`PrivateInnerClass`，代码不会编译，我们会看到错误:

```java
PrivateInnerClass cannot be resolved to a type
```

## 8.结论

在这个快速教程中，我们讨论了 Java 中的`private`访问修饰符。这是一种很好的实现封装的方法，这导致了信息隐藏。因此，我们可以确保只向其他类公开我们想要的数据和行为。

与往常一样，GitHub 上的[提供了代码示例。](https://web.archive.org/web/20221128041817/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-oop-modifiers)