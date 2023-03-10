# Groovy 中的特征介绍

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/groovy-traits>

## 1.概观

在本教程中，我们将探索 [Groovy](/web/20220628084525/https://www.baeldung.com/groovy-language) 中特征的概念。它们是在 Groovy 2.3 版本中引入的。

## 2.什么是特质？

特征是代表一组方法或行为的可重用组件，我们可以用它们来扩展多个类的功能。

由于这个原因，**它们被认为是接口，承载默认实现和状态。所有特征都使用关键字`trait`来定义。**

## 3.方法

在`trait`中声明方法类似于在类中声明任何常规方法。然而，我们不能在`trait`中声明受保护的或者包私有的方法。

让我们看看公共和私有方法是如何实现的。

### 3.1.公共方法

首先，我们将探索如何在`trait.` 中实现`public`方法

让我们创建一个名为`UserTrait` 的`trait`和一个`public`的`sayHello`方法:

```java
trait UserTrait {
    String sayHello() {
        return "Hello!"
    }
}
```

之后，我们将创建一个`Employee`类，它实现了`UserTrait`:

```java
class Employee implements UserTrait {}
```

现在，让我们创建一个测试来验证一个`Employee`实例可以访问`UserTrait`的`the sayHello`方法:

```java
def 'Should return msg string when using Employee.sayHello method provided by User trait' () {
    when:
        def msg = employee.sayHello()
    then:
        msg
        msg instanceof String
        assert msg == "Hello!"
}
```

### 3.2.私有方法

我们也可以在一个`trait`中创建一个`private`方法，并在另一个`public`方法中引用它。

让我们看看`UserTrait:`中的代码实现

```java
private String greetingMessage() {
    return 'Hello, from a private method!'
}

String greet() {
    def msg = greetingMessage()
    println msg
    return msg
} 
```

注意，如果我们访问实现类中的`private` 方法，它将抛出一个`MissingMethodException`:

```java
def 'Should return MissingMethodException when using Employee.greetingMessage method' () {
    when:
        def exception
        try {
            employee.greetingMessage()
        } catch(Exception e) {
            exception = e
        }

    then:
        exception
        exception instanceof groovy.lang.MissingMethodException
        assert exception.message == "No signature of method: com.baeldung.traits.Employee.greetingMessage()"
          + " is applicable for argument types: () values: []"
}
```

在一个`trait,`中，一个私有方法对于任何不应该被任何类覆盖的实现来说都是必不可少的，尽管其他公共方法也需要它。

### 3.3.抽象方法

一个`trait`也可以包含能够在另一个类中实现的`abstract`方法:

```java
trait UserTrait {
    abstract String name()

    String showName() {
       return "Hello, ${name()}!"
    }
}
```

```java
class Employee implements UserTrait {
    String name() {
        return 'Bob'
    }
} 
```

### 3.4.覆盖默认方法

通常，`trait`包含其公共方法的默认实现，但是我们可以在实现类中覆盖它们:

```java
trait SpeakingTrait {
    String speak() {
        return "Speaking!!"
    }
} 
```

```java
class Dog implements SpeakingTrait {
    String speak() {
        return "Bow Bow!!"
    }
} 
```

**特征不支持`protected`和`private`作用域。**

## 4.`this`关键字

` this`关键字的行为类似于 Java 中的行为。我们可以认为`trait`是一个`super`类。

例如，我们将创建一个在`trait`中返回`this` 的方法:

```java
trait UserTrait {
    def self() {
        return this 
    }
}
```

## 5.接口

A `trait`也可以实现接口，就像普通类一样。

让我们创建一个`interface`并在一个`trait`中实现它:

```java
interface Human {
    String lastName()
}
```

```java
trait UserTrait implements Human {
    String showLastName() {
        return "Hello, ${lastName()}!"
    }
}
```

现在，让我们在实现类中实现`interface`的`abstract`方法:

```java
class Employee implements UserTrait {
    String lastName() {
        return "Marley"
    }
}
```

## 6.性能

**我们可以给一个`trait`添加属性，就像我们在任何一个普通的类中做的一样:**

```java
trait UserTrait implements Human { 
    String email
    String address
}
```

## 7.延伸特性

与常规 Groovy `class`类似，`trait`可以使用`extends`关键字扩展另一个`trait`:

```java
trait WheelTrait {
    int noOfWheels
}

trait VehicleTrait extends WheelTrait {
    String showWheels() {
        return "Num of Wheels $noOfWheels" 
    } 
}

class Car implements VehicleTrait {}
```

**我们还可以用`implements`子句扩展多个特征:**

```java
trait AddressTrait {                                      
    String residentialAddress
}

trait EmailTrait {                                    
    String email
}

trait Person implements AddressTrait, EmailTrait {}
```

## 8.多重继承冲突

当一个类实现两个或更多的特征，这些特征的方法具有相同的签名时，我们需要知道如何解决冲突。让我们看看 Groovy 如何默认解决这种冲突，以及我们可以覆盖默认解决方案的方法。

### 8.1.默认冲突解决方案

**默认情况下，将从`implements`子句中最后声明的`trait`中选择**方法。

所以性状帮助我们实现多重继承，不会遇到[钻石问题](https://web.archive.org/web/20220628084525/http://www.lambdafaq.org/what-about-the-diamond-problem/)。

首先，让我们用一个具有相同签名的方法创建两个特征:

```java
trait WalkingTrait {
    String basicAbility() {
        return "Walking!!"
    }
}

trait SpeakingTrait {
    String basicAbility() {
        return "Speaking!!"
    }
} 
```

接下来，让我们编写一个实现这两种特征的类:

```java
class Dog implements WalkingTrait, SpeakingTrait {} 
```

因为`SpeakingTrait`是最后声明的，所以默认情况下，它的`basicAbility`方法实现将在`Dog`类中被拾取。

### 8.2.显式冲突解决方案

现在，如果我们不想简单地采用语言提供的默认冲突解决方案，我们可以通过 **显式地选择使用`trait.super`调用哪个方法来覆盖它**。`method`参考。****

例如，让我们为我们的两个特征添加另一个具有相同签名的方法:

```java
String speakAndWalk() {
    return "Walk and speak!!"
}
```

```java
String speakAndWalk() {
    return "Speak and walk!!"
}
```

现在，让我们使用`super`关键字覆盖`Dog`类中多重继承冲突的默认解决方案:

```java
class Dog implements WalkingTrait, SpeakingTrait {
    String speakAndWalk() {
        WalkingTrait.super.speakAndWalk()
    }
}
```

## 9.在运行时实现特征

为了动态实现一个`trait`，我们可以**使用`as`关键字在运行时将一个对象强制转换为`trait`**。

例如，让我们用`basicBehavior`方法创建一个`AnimalTrait`:

```java
trait AnimalTrait {
    String basicBehavior() {
        return "Animalistic!!"
    }
}
```

为了一次实现几个特征，我们可以使用`withTraits`方法来代替`as`关键字:

```java
def dog = new Dog()
def dogWithTrait = dog.withTraits SpeakingTrait, WalkingTrait, AnimalTrait
```

## 10.结论

在本文中，我们看到了如何在 Groovy 中创建特征，并探索了它们的一些有用特性。

一个`trait`是在我们的类中添加公共实现和功能的一个非常有效的方法。此外，它允许我们最小化冗余代码，并使代码维护更容易。

和往常一样，本文的代码实现和单元测试可以在 GitHub 上的[处获得。](https://web.archive.org/web/20220628084525/https://github.com/eugenp/tutorials/tree/master/core-groovy-modules/core-groovy)