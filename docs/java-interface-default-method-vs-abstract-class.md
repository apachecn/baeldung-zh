# 默认方法与抽象类的接口

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-interface-default-method-vs-abstract-class>

## 1.介绍

在 Java 接口中引入了`default`方法之后，接口和抽象类之间似乎不再有任何区别。但是，事实并非如此——它们之间有一些根本的区别。

在本教程中，我们将仔细看看接口和抽象类，看看它们有什么不同。

## 2.为什么使用默认方法？

[`default`方法](/web/20220524112200/https://www.baeldung.com/java-static-default-methods#why-default-methods-in-interfaces-are-needed)的**目的是在不破坏现有实现的情况下提供外部功能**。引入`default`方法的最初动机是用新的 lambda 函数为集合框架提供向后兼容性。

## 3.与`default`方法和抽象类的接口

让我们来看看主要的基本差异。

### 3.1.状态

抽象类可以有一个状态，它的方法可以访问实现的状态。虽然接口中允许使用`default`方法，但是它们不能访问实现的状态。

我们在`default`方法中编写的任何**逻辑都应该与接口的其他方法相关——这些方法将独立于对象的状态**。

假设我们已经创建了一个抽象类`CircleClass`，它包含一个`String`、`color`，来表示`CircleClass`对象的状态:

```java
public abstract class CircleClass {

    private String color;
    private List<String> allowedColors = Arrays.asList("RED", "GREEN", "BLUE");

    public boolean isValid() {
        if (allowedColors.contains(getColor())) {
            return true;
        } else {
            return false;
        }
    }

    //standard getters and setters
}
```

在上面的`abstract`类中，我们有一个名为`isValid()` 的非抽象方法来基于状态验证一个`CircleClass`对象。 **`isValid()`方法可以访问`CircleClass`对象**的状态，并根据允许的颜色验证`CircleClass`的实例。由于这种行为，**我们可以根据对象的状态**在抽象类方法中编写任何逻辑。

让我们创建一个简单的`CircleClass` `:`实现类

```java
public class ChildCircleClass extends CircleClass {
}
```

现在，让我们创建一个实例并验证颜色:

```java
CircleClass redCircle = new ChildCircleClass();
redCircle.setColor("RED");
assertTrue(redCircle.isValid());
```

在这里，我们可以看到，当我们在`CircleClass`对象中放置一个有效的颜色并调用`isValid()`方法时，在内部，`isValid()`方法可以访问`CircleClass`对象的状态并检查实例是否包含有效的颜色。

让我们尝试使用一个带有`default`方法的接口来做类似的事情:

```java
public interface CircleInterface {
    List<String> allowedColors = Arrays.asList("RED", "GREEN", "BLUE");

    String getColor();

    public default boolean isValid() {
        if (allowedColors.contains(getColor())) {
            return true;
        } else {
            return false;
        }
    }
}
```

我们知道，接口不能有状态，因此，`default`方法不能访问状态。

这里，我们定义了`getColor()`方法来提供状态信息。子类将覆盖`getColor()`方法来提供运行时实例的状态:

```java
public class ChidlCircleInterfaceImpl implements CircleInterface {
    private String color;

    @Override
    public String getColor() {
        return color;
    }

    public void setColor(String color) {
        this.color = color;
    }
}
```

让我们创建一个实例并验证颜色:

```java
ChidlCircleInterfaceImpl redCircleWithoutState = new ChidlCircleInterfaceImpl();
redCircleWithoutState.setColor("RED");
assertTrue(redCircleWithoutState.isValid());
```

正如我们在这里看到的，我们在子类中覆盖了`getColor()`方法，以便`default`方法在运行时验证状态。

### 3.2.构造器

抽象类可以有构造函数 **，允许我们在创建时初始化状态**。当然，接口没有构造函数。

### 3.3.句法差异

此外，语法方面也没有什么不同。一个**抽象类可以覆盖`Object`类方法**，但是一个接口不能。

一个**抽象类可以用所有可能的访问修饰符**声明实例变量，并且它们可以在子类中被访问。一个接口只能有`public,` `static`和`final` 变量，不能有任何实例变量。

此外，**抽象类可以声明实例和静态块**，而接口不能声明这两者。

最后，**抽象类不能引用 lambda 表达式**，而接口可以有一个引用 lambda 表达式的抽象方法。

## 4.结论

本文展示了抽象类和使用`default`方法的接口之间的区别。我们还根据我们的场景看到了哪一个最适合。

只要有可能，我们应该**总是选择使用`default`方法的接口，因为它允许我们扩展一个类，并且** **也实现了一个接口**。

像往常一样，本文中显示的所有代码示例都可以在 GitHub 上的[处获得。](https://web.archive.org/web/20220524112200/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-8-2)