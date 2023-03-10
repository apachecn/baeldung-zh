# Java 中的继承和组合(Is-a vs Has-a 关系)

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-inheritance-composition>

## 1。概述

继承和组合——以及抽象、封装和多态——是面向对象编程(OOP)的基石[。](https://web.archive.org/web/20221205124259/https://en.wikipedia.org/wiki/Object-oriented_programming)

在本教程中，我们将涵盖继承和组合的基础知识，我们将重点关注发现两种类型的关系之间的差异。

## 2。继承的基础知识

继承是一种强大但被过度使用和误用的机制。

简而言之，通过继承，基类(也称为基本类型)定义给定类型的公共状态和行为，并让子类(也称为子类型)提供该状态和行为的专用版本。

为了清楚地了解如何使用继承，让我们创建一个简单的例子:一个基类`Person`为一个人定义了公共字段和方法，而子类`Waitress`和`Actress`提供了额外的细粒度方法实现。

下面是`Person`类:

```java
public class Person {
    private final String name;

    // other fields, standard constructors, getters
}
```

这些是子类:

```java
public class Waitress extends Person {

    public String serveStarter(String starter) {
        return "Serving a " + starter;
    }

    // additional methods/constructors
} 
```

```java
public class Actress extends Person {

    public String readScript(String movie) {
        return "Reading the script of " + movie;
    } 

    // additional methods/constructors
}
```

此外，让我们创建一个单元测试来验证`Waitress`和`Actress`类的实例也是`Person`的实例，从而表明在类型级别满足“is-a”条件:

```java
@Test
public void givenWaitressInstance_whenCheckedType_thenIsInstanceOfPerson() {
    assertThat(new Waitress("Mary", "[[email protected]](/web/20221205124259/https://www.baeldung.com/cdn-cgi/l/email-protection)", 22))
      .isInstanceOf(Person.class);
}

@Test
public void givenActressInstance_whenCheckedType_thenIsInstanceOfPerson() {
    assertThat(new Actress("Susan", "[[email protected]](/web/20221205124259/https://www.baeldung.com/cdn-cgi/l/email-protection)", 30))
      .isInstanceOf(Person.class);
}
```

**这里强调继承的语义方面很重要**。除了重用`Person class`、**的实现，我们还在基础类型`Person`和子类型`Waitress`和`Actress`之间创建了一个定义良好的“是-a”关系**。女服务员和女演员实际上都是人。

这可能会让我们问:**在哪些用例中继承是正确的方法？**

**如果子类型满足“is-a”条件，并且主要提供类层次结构中更下一层的附加功能，** **那么继承就是一条路。**

当然，只要被覆盖的方法保留了由 [Liskov 替换原则](https://web.archive.org/web/20221205124259/https://en.wikipedia.org/wiki/Liskov_substitution_principle)所提倡的基本类型/子类型可替换性，方法覆盖就是允许的。

此外，我们应该记住**子类型继承了基础类型的 API** ，这在某些情况下可能是多余的或者仅仅是不可取的。

不然就用作文代替。

## 3。设计模式中的继承

虽然共识是我们应该尽可能地支持组合而不是继承，但是有几个典型的用例中继承有它的位置。

### 3.1。层超类型图案

在这种情况下，我们**使用继承将公共代码移动到一个基类(超类型)，在每层的基础上**。

下面是这种模式在领域层的基本实现:

```java
public class Entity {

    protected long id;

    // setters
} 
```

```java
public class User extends Entity {

    // additional fields and methods   
} 
```

我们可以将相同的方法应用于系统中的其他层，例如服务层和持久层。

### 3.2。模板方法模式

在模板方法模式中，我们可以**使用一个基类来定义算法的不变部分，然后在子类中实现变化部分**:

```java
public abstract class ComputerBuilder {

    public final Computer buildComputer() {
        addProcessor();
        addMemory();
    }

    public abstract void addProcessor();

    public abstract void addMemory();
} 
```

```java
public class StandardComputerBuilder extends ComputerBuilder {

    @Override
    public void addProcessor() {
        // method implementation
    }

    @Override
    public void addMemory() {
        // method implementation
    }
}
```

## 4。构图基础

组合是 OOP 为重用实现提供的另一种机制。

简而言之， **composition 允许我们对由其他对象**组成的对象进行建模，从而定义它们之间的“has-a”关系。

此外，**组合是[关联](https://web.archive.org/web/20221205124259/https://en.wikipedia.org/wiki/Association_(object-oriented_programming))** 的最强形式，这意味着**当一个对象被破坏**时，组成该对象或被该对象包含的对象也被破坏。

为了更好地理解合成是如何工作的，让我们假设我们需要使用代表计算机的对象`.`

一台计算机由不同的部分组成，包括微处理器、内存、声卡等等，所以我们可以把计算机和它的每个部分都建模为单独的类。

下面是一个简单的`Computer`类的实现:

```java
public class Computer {

    private Processor processor;
    private Memory memory;
    private SoundCard soundCard;

    // standard getters/setters/constructors

    public Optional<SoundCard> getSoundCard() {
        return Optional.ofNullable(soundCard);
    }
}
```

下面的类模拟了微处理器、内存和声卡(为了简洁起见，省略了接口):

```java
public class StandardProcessor implements Processor {

    private String model;

    // standard getters/setters
}
```

```java
public class StandardMemory implements Memory {

    private String brand;
    private String size;

    // standard constructors, getters, toString
} 
```

```java
public class StandardSoundCard implements SoundCard {

    private String brand;

    // standard constructors, getters, toString
} 
```

很容易理解推动组合胜过继承背后的动机。在给定类和其他类之间可能建立语义正确的“有-有”关系的每一个场景中，组合都是正确的选择。

在上面的例子中，`Computer`满足“has-a”条件，它的类为它的部件建模。

同样值得注意的是，在这种情况下，**包含的`Computer`对象拥有被包含对象`if and only if`的所有权，这些对象不能在另一个`Computer`对象中重用。**如果他们可以，我们会使用聚合，而不是组合，在组合中所有权是不隐含的。

## 5。没有抽象的组合

或者，我们可以通过硬编码`Computer`类的依赖关系来定义组合关系，而不是在构造函数中声明它们:

```java
public class Computer {

    private StandardProcessor processor
      = new StandardProcessor("Intel I3");
    private StandardMemory memory
      = new StandardMemory("Kingston", "1TB");

    // additional fields / methods
}
```

**当然，这将是一个严格、紧密耦合的设计，因为我们将使`Computer`强烈依赖于`Processor`和`Memory`的具体实现。**

我们不会利用接口和[依赖注入](https://web.archive.org/web/20221205124259/https://en.wikipedia.org/wiki/Dependency_injection)提供的抽象层次。

通过基于接口的初始设计，我们得到了一个松散耦合的设计，这也更容易测试。

## 6。结论

在本文中，我们学习了 Java 中继承和组合的基础知识，并深入探讨了这两种类型的关系(“is-a”与“has-a”)之间的差异。

和往常一样，本教程中展示的所有代码示例都可以从 GitHub 上的[处获得。](https://web.archive.org/web/20221205124259/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-oop-patterns)