# Java 中的标记接口

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-marker-interfaces>

## 1.介绍

在这个快速教程中，我们将学习 Java 中的标记接口。

## 2.标记接口

标记接口是一个[接口](/web/20220628235356/https://www.baeldung.com/java-interfaces)，其中**没有方法或常量**。它提供了关于对象的**运行时类型信息，因此编译器和 JVM 拥有关于对象**的**附加信息。**

标记界面也称为标记界面。

尽管标记接口仍在使用，但它们很可能带有代码味道，应该小心使用。主要原因是它们模糊了界面所代表的界限，因为标记没有定义任何行为。更新的开发倾向于用注释来解决一些相同的问题。

## 3.JDK 标记接口

Java 有很多内置的标记接口，比如`Serializable`、`Cloneable`、`Remote.`

让我们以 [`Cloneable`界面](/web/20220628235356/https://www.baeldung.com/java-deep-copy)为例。如果我们试图克隆一个没有实现这个接口的对象，JVM 会抛出一个`CloneNotSupportedException`。因此，`Cloneable` **标记接口是 JVM** 的一个指示器，我们可以调用`Object.clone()`方法。

同样，当调用`ObjectOutputStream.writeObject()`方法时，**JVM 检查对象是否实现了`Serializable `标记接口**。如果不是这样，就抛出一个`NotSerializableException`。因此，对象不会被序列化到输出流中。

## 4.自定义标记界面

让我们创建自己的标记接口。

例如，我们可以创建一个标记来指示某个对象是否可以从数据库中删除:

```java
public interface Deletable {
}
```

为了从数据库中删除一个实体，表示这个实体的对象必须实现我们的`Deletable `标记接口:

```java
public class Entity implements Deletable {
    // implementation details
}
```

假设我们有一个 DAO 对象，它有一个从数据库中删除实体的方法。我们可以编写我们的`delete()`方法，这样**只有实现我们的标记接口**的对象才能被删除:

```java
public class ShapeDao {

    // other dao methods

    public boolean delete(Object object) {
        if (!(object instanceof Deletable)) {
            return false;
        }

        // delete implementation details

        return true;
    }
}
```

正如我们所看到的，**我们正在给 JVM 一个指示，关于我们的对象的运行时行为。**如果对象实现了我们的标记接口，就可以从数据库中删除。

## 5.标记界面与注释

通过引入注释，Java 为我们提供了一种替代方法，可以获得与标记接口相同的结果。此外，像标记接口一样，我们可以将注释应用于任何类，并且可以使用它们作为指示器来执行某些操作。

那么关键的区别是什么呢？

与注释不同，接口允许我们利用多态性。因此，我们可以**给标记接口添加额外的限制。**

例如，让我们添加一个限制，即只能从数据库中删除一个`Shape `类型:

```java
public interface Shape {
    double getArea();
    double getCircumference();
}
```

在这种情况下，我们的标记接口，姑且称之为`DeletableShape,`将如下所示:

```java
public interface DeletableShape extends Shape {
}
```

然后我们的类将实现标记接口:

```java
public class Rectangle implements DeletableShape {
    // implementation details
}
```

因此，**所有的`DeletableShape` 实现也是`Shape `实现**。显然，**我们不能使用注释**来做到这一点。

然而，每个设计决策都有权衡，并且**多态性可以用作反对标记接口的反驳**。**在我们的例子中，每个扩展了`Rectangle`的类都会自动实现`DeletableShape.`**

## 6.标记接口与典型接口

在前面的例子中，我们可以通过修改我们的 DAO 的`delete()`方法来测试我们的对象是否是一个`Shape `或者不是一个`,`，而不是测试它是否是一个`Deletable:`，从而得到相同的结果

```java
public class ShapeDao { 

    // other dao methods 

    public boolean delete(Object object) {
        if (!(object instanceof Shape)) {
            return false;
        }

        // delete implementation details

        return true;
    }
}
```

那么，当我们可以使用一个典型的界面达到同样的效果时，为什么还要创建一个标记界面呢？

让我们想象一下，除了`Shape`类型，我们还想从数据库中删除`Person`类型。在这种情况下，有两个选项可以实现:

第一个选项是**向我们之前的`delete()`方法**添加一个额外的检查，以验证要删除的对象是否是`Person `的实例。

```java
public boolean delete(Object object) {
    if (!(object instanceof Shape || object instanceof Person)) {
        return false;
    }

    // delete implementation details

    return true;
}
```

但是如果我们还想从数据库中删除更多的类型呢？显然，这不是一个好的选择，因为我们不得不为每一个新类型改变我们的方法。

第二个选项是**让****`Person`类型实现`Shape `接口**，它充当一个标记接口。但是一个`Person`物体真的是一个`Shape`吗？答案显然是否定的，这使得第二个选择比第一个更糟糕。

因此，尽管我们可以通过使用一个典型的接口作为标记来达到同样的结果，但我们最终会得到一个糟糕的设计。

## 7.结论

在本文中，我们讨论了什么是标记接口以及如何使用它们。然后我们看了一些这类接口的内置 Java 例子，以及 JDK 是如何使用它们的。

接下来，我们创建了自己的标记接口，并与使用注释进行了权衡。最后，我们将看到为什么在某些场景中使用标记接口而不是传统接口是一个好的实践。

和往常一样，代码可以在 [GitHub](https://web.archive.org/web/20220628235356/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-oop-types) 上找到。