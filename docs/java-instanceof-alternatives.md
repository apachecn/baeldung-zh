# Java 中 instanceof 运算符的替代方法

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-instanceof-alternatives>

## 1.概观

**[`instanceof`](/web/20221218213112/https://www.baeldung.com/java-instanceof)** **是将对象的实例比作类型的运算符**。它也被称为类型比较运算符。

在本教程中，我们将看看传统的 `instanceof` 方法的不同替代方案。我们可能需要替代方案来改进代码设计和可读性。

## 2.示例设置

让我们开发一个简单的程序，`Dinosaur`和`Species`。这个程序将有一个父类和两个子类，即子类将扩展父类。

首先，让我们创建父类:

```java
public class Dinosaur {
}
```

接下来，让我们创建第一个子类:

```java
public class Anatotitan extends Dinosaur {
    String run() {
        return "running";
    }
}
```

最后，让我们创建第二个子类:

```java
public class Euraptor extends Dinosaur {	
    String flies() {
        return "flying";
    }
}
```

`Dinosaur `类有其他子类共有的方法，但是为了简单起见，我们跳过了它们。

接下来，让我们编写一个方法来创建对象的新实例并调用它们的移动。在返回结果之前，我们将使用`instanceof`来检查新的实例类型:

```java
public static void moveDinosaur(Dinosaur dinosaur) {
    if (dinosaur instanceof Anatotitan) {
        Anatotitan anatotitan = (Anatotitan) dinosaur;
        anatotitan.run();
    } 
    else if (dinosaur instanceof Euraptor) {
        Euraptor euraptor = (Euraptor) dinosaur;
        euraptor.flies();
    }
}
```

在接下来的部分中，我们将应用不同的替代方案。

## 3.使用`getClass()`

**[`getClass()`](/web/20221218213112/https://www.baeldung.com/java-finding-class) 方法有助于获取对象的类**。当检查一个对象是否属于一个特定的类时，我们可以使用`getClass()`作为`instanceof` 的替代。

在我们的示例设置中，让我们保持父类和子类的结构。然后，让我们为这种方法编写一个测试方法。 我们用 `getClass()` 代替 `instanceof` :

```java
public static String moveDinosaurUsingGetClass(Dinosaur dinosaur) {
    if (dinosaur.getClass().equals(Anatotitan.class)) {
        Anatotitan anatotitan = (Anatotitan) dinosaur;
        return anatotitan.run();
    } else if (dinosaur.getClass().equals(Euraptor.class)) {
        Euraptor euraptor = (Euraptor) dinosaur;
        return euraptor.flies();
    }
    return "";
}
```

让我们为这种方法编写一个单元测试:

```java
@Test
public void givenADinosaurSpecie_whenUsingGetClass_thenGetMovementOfEuraptor() {
    assertEquals("flying", moveDinosaurUsingGetClass(new Euraptor()));
}
```

这种替代方法维护了我们最初的域对象。变化的是`getClass()`的使用。

## 4.使用多态性

多态性的概念使子类覆盖了父类的方法。**我们可以使用这个概念来改变我们的示例设置，并改进我们的代码设计和可读性。**

因为我们知道所有恐龙移动，我们可以通过在我们的父类中引入一个`move()`方法来改变我们的设计:

```java
public class Dinosaur {	
    String move() {
        return "walking";
    } 
}
```

接下来，让我们通过覆盖`move()`方法来修改我们的子类:

```java
public class Anatotitan extends Dinosaur {
    @Override
    String move() {
        return "running";
    }
}

public class Euraptor extends Dinosaur {
    @Override
    String move() {
        return "flying";
    }
}
```

现在我们可以引用子类而不用使用`instanceof`方法。让我们写一个接受父类作为参数的方法。我们将根据恐龙种类返回我们的恐龙运动:

```java
public static String moveDinosaurUsingPolymorphism(Dinosaur dinosaur) { 
    return dinosaur.move(); 
}
```

让我们为这种方法编写一个单元测试:

```java
@Test 
public void givenADinosaurSpecie_whenUsingPolymorphism_thenGetMovementOfAnatotitan() { 
    assertEquals("running", moveDinosaurUsingPolymorphism(new Anatotitan()));
}
```

如果可能的话，建议使用这种方法来改变我们的设计本身。使用`instanceof`通常表明我们的设计违反了[利斯科夫替代原理(LSP)](/web/20221218213112/https://www.baeldung.com/java-liskov-substitution-principle) 。

## 5.使用枚举

在 [`enum`](/web/20221218213112/https://www.baeldung.com/a-guide-to-java-enums) 类型中，**变量可以定义为预定义常量**的集合。我们可以使用这种方法来改进我们的简单程序。

首先，让我们用包含方法的常量创建一个`enum`。常量的方法覆盖了`enum`中的`abstract`方法:

```java
public enum DinosaurEnum {
    Anatotitan {
        @Override
        public String move() {
            return "running";
        }
    },
    Euraptor {
        @Override
        public String move() {
            return "flying";
        }
    };
    abstract String move();
}
```

`enum`常量的作用类似于其他替代方法中使用的子类。

接下来，让我们修改我们的`moveDinosaur()`方法，使用`e`类型:

```java
public static String moveDinosaurUsingEnum(DinosaurEnum dinosaurEnum) {
    return dinosaurEnum.move();
}
```

最后，让我们为这种方法编写一个单元测试:

```java
@Test
public void givenADinosaurSpecie_whenUsingEnum_thenGetMovementOfEuraptor() {
    assertEquals("flying", moveDinosaurUsingEnum(DinosaurEnum.Euraptor));
}
```

这种设计使我们消除了父类和子类。在父类的行为比我们的示例设置更多的复杂场景中，不建议使用这种方法。

## 6.使用访问者模式

[访问者模式](/web/20221218213112/https://www.baeldung.com/java-visitor-pattern) 有助于对相似/相关对象进行操作。**将逻辑从对象类移动到另一个类**。

让我们将这种方法应用到我们的示例设置中。首先，让我们用一个方法创建一个接口，并传递一个`Visitor`作为参数。这将有助于检索我们的对象的类型:

```java
public interface Dinosaur {
    String move(Visitor visitor);
}
```

接下来，让我们用两个方法创建一个`Visitor`接口。这些方法接受我们的子类作为参数:

```java
public interface Visitor {
    String visit(Anatotitan anatotitan);
    String visit(Euraptor euraptor);
}
```

接下来，让我们让我们的子类实现`Dinosaur`接口并覆盖它的方法。该方法使用`Visitor`作为参数来检索我们的对象类型。这种方法取代了`instanceof`的使用:

```java
public class Anatotitan implements Dinosaur {
    public String run() {
        return "running";
    }
    @Override
    public String move(Visitor dinoMove) {
        return dinoMove.visit(this);
    }
}
```

接下来，让我们创建一个类来实现我们的`Visitor`接口并覆盖这些方法:

```java
public class DinoVisitorImpl implements Visitor {
    @Override
    public String visit(Anatotitan anatotitan) {
        return anatotitan.run();
    }
    @Override
    public String visit(Euraptor euraptor) {
        return euraptor.flies();
    }
}
```

最后，让我们为这种方法编写一个测试方法:

```java
public static String moveDinosaurUsingVisitorPattern(Dinosaur dinosaur) {
    Visitor visitor = new DinoVisitorImpl();
    return dinosaur.move(visitor);
}
```

让我们为这种方法编写一个单元测试:

```java
@Test
public void givenADinosaurSpecie_whenUsingVisitorPattern_thenGetMovementOfAnatotitan() {
    assertEquals("running", moveDinosaurUsingVisitorPattern(new Anatotitan()));
}
```

这种方法利用了一个接口。`Visitor` 包含了我们的程序逻辑。

## 7.结论

在本文中，我们研究了不同的 `instanceof`选择。**`instanceof`方法潜在地违反了里斯科夫替代原理**。采用替代方案给了我们更好、更可靠的设计。推荐使用多态方法，因为它增加了更多的价值。

和往常一样，完整的源代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20221218213112/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-operators-2)