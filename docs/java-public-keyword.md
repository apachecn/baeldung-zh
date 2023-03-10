# Java“公共”访问修饰符

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-public-keyword>

## 1.概观

在这篇简短的文章中，我们将深入讨论`public`修饰符，并且我们将讨论何时以及如何对类和成员使用它。

此外，我们将说明使用公共数据字段的缺点。

对于访问修饰符的一般概述，一定要看看我们关于 Java 中的[访问修饰符的文章。](/web/20221129021829/https://www.baeldung.com/java-access-modifiers)

## 2.何时使用公共访问修饰符

公共类和接口以及公共成员定义了 API。它是我们代码中的一部分，其他人可以看到并使用它来控制我们对象的行为。

然而，过度使用 public 修饰符违反了面向对象编程(OOP)的封装原则，并且有一些缺点:

*   它增加了 API 的大小，使客户端更难使用
*   更改我们的代码变得越来越难，因为客户依赖它——任何未来的更改都可能破坏他们的代码

## 3.公共接口和类

### 3.1.公共接口

公共接口定义了一个可以有一个或多个实现的规范。这些实现可以由我们提供，也可以由他人编写。

例如，Java API 公开了`Connection`接口来定义数据库连接操作，将实际实现留给每个供应商。在运行时，我们根据项目设置获得所需的连接:

```java
Connection connection = DriverManager.getConnection(url);
```

`getConnection`方法返回特定技术实现的实例。

### 3.2.公共课程

我们定义公共类，以便客户端可以通过实例化和静态引用来使用它们的成员:

```java
assertEquals(0, new BigDecimal(0).intValue()); // instance member
assertEquals(2147483647, Integer.MAX_VALUE); // static member 
```

此外，我们可以通过使用可选的 [`abstract`](/web/20221129021829/https://www.baeldung.com/java-abstract-class) 修饰符来设计用于继承的公共类。**当我们使用`abstract`修饰符时，类就像一个骨架，除了每个子类需要实现的抽象方法之外，它还有任何具体实现都可以使用的字段和预实现方法**。

例如，Java collections 框架提供了`AbstractList`类作为创建定制列表的基础:

```java
public class ListOfThree<E> extends AbstractList<E> {

    @Override
    public E get(int index) {
        //custom implementation
    }

    @Override
    public int size() {
        //custom implementation
    }

}
```

所以，我们只需要实现`get()`和`size()`方法。其他方法如`indexOf()` 和`containsAll()`已经为我们实现了。

### 3.3.嵌套的公共类和接口

类似于公共顶级类和接口，嵌套的公共类和接口定义了 API 数据类型。但是，它们在两个方面特别有用:

*   它们向 API 最终用户表明封闭的顶级类型和它的封闭类型具有逻辑关系，并且一起使用
*   它们通过减少源代码文件的数量使我们的代码库更加紧凑，如果我们将它们声明为顶级类和接口的话，我们将会使用它们

一个例子是来自核心 Java API 的`Map` `.` `Entry`接口:

```java
for (Map.Entry<String, String> entry : mapObject.entrySet()) { }
```

制作`Map` `.` `Entry a`嵌套接口将它与`java.util.Map`接口紧密地联系在一起，避免了我们在`java.util`包中创建另一个文件。

更多细节请阅读[嵌套类](/web/20221129021829/https://www.baeldung.com/java-nested-classes)一文。

## 4.公共方法

公共方法使用户能够执行现成的操作。一个例子是`String` API 中的公共`toLowerCase`方法:

```java
assertEquals("alex", "ALEX".toLowerCase());
```

如果公共方法不使用任何实例字段，我们可以安全地将其设为静态。来自`Integer`类的`parseInt`方法是一个公共静态方法的例子:

```java
assertEquals(1, Integer.parseInt("1"));
```

构造函数通常是公共的，这样我们可以实例化和初始化对象，尽管有时它们可能是私有的，比如在 [singletons](/web/20221129021829/https://www.baeldung.com/java-singleton) 中。

## 5.公共字段

公共字段允许直接改变对象的状态。经验法则是我们不应该使用公共字段。这有几个原因，我们将会看到。

### 5.1.线程安全

对非 final 字段或 final 可变字段使用公共可见性不是线程安全的。我们无法控制在不同的线程中改变它们的引用或状态。

请查看我们关于[线程安全](/web/20221129021829/https://www.baeldung.com/java-thread-safety)的文章，了解更多关于编写线程安全代码的信息。

### 5.2.对修改采取措施

我们无法控制非 final 公共字段，因为它的引用或状态可以直接设置。

相反，最好使用私有修饰符隐藏这些字段，并使用公共 setter:

```java
public class Student {

    private int age;

    public void setAge(int age) {
        if (age < 0 || age > 150) {
            throw new IllegalArgumentException();
        }

        this.age = age;
    }
}
```

### 5.3.更改数据类型

可变或不可变的公共字段是客户端契约的一部分。在未来的版本中，更难改变这些字段的数据表示，因为客户可能需要重构它们的实现。

通过赋予字段私有范围和使用访问器，我们可以灵活地更改内部表示，同时保持旧的数据类型:

```java
 public class Student {

    private StudentGrade grade; //new data representation

    public void setGrade(int grade) {        
        this.grade = new StudentGrade(grade);
    }

    public int getGrade() {
        return this.grade.getGrade().intValue();
    }
}
```

使用公共字段的唯一例外是使用静态最终不可变字段来表示常量:

```java
public static final String SLASH = "/";
```

## 6.结论

在本教程中，我们看到 public 修饰符被用来定义一个 API。

此外，我们还描述了过度使用这个修饰符会如何限制对我们的实现进行改进的能力。

最后，我们讨论了为什么对字段使用公共修饰符是不好的做法。

和往常一样，这篇文章的代码示例可以在 GitHub 上的[处获得。](https://web.archive.org/web/20221129021829/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-oop-modifiers)