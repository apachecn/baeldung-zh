# Java 中的对象类型转换

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-type-casting>

## 1。概述

Java 类型系统由两种类型组成:原语和引用。

我们在本文的[中讨论了原语转换，我们将在这里关注引用转换，以便更好地理解 Java 如何处理类型。](/web/20221205112022/http://www.baeldung.com/java-primitive-conversions)

## 延伸阅读:

## [Java 泛型的基础知识](/web/20221205112022/http://www.baeldung.com/java-generics)

A quick intro tot he basics of Java Generics.[Read more](/web/20221205112022/http://www.baeldung.com/java-generics) →

## [运算符的 Java 实例](/web/20221205112022/http://www.baeldung.com/java-instanceof)

Learn about the instanceof operator in Java[Read more](/web/20221205112022/http://www.baeldung.com/java-instanceof) →

## 2。原语与引用

尽管原语转换和引用变量转换看起来很相似，但它们是完全不同的概念。

在这两种情况下，我们都在将一种类型“转变”成另一种类型。但是，简单地说，原始变量包含其值，原始变量的转换意味着其值的不可逆变化:

```
double myDouble = 1.1;
int myInt = (int) myDouble;

assertNotEquals(myDouble, myInt);
```

上例转换后，`myInt`变量为`1`，我们无法从中恢复之前的值`1.1`。

**参考变量不同**；引用变量仅引用一个对象，但不包含对象本身。

转换一个引用变量并不触及它所引用的对象，而只是以另一种方式标记这个对象，扩大或缩小使用它的机会。向上转换缩小了该对象可用的方法和属性的列表，向下转换可以扩展它。

引用就像是对象的遥控器。遥控器根据它的类型有更多或更少的按钮，对象本身存储在一个堆中。当我们进行造型时，我们改变了遥控器的类型，但没有改变对象本身。

## 3。向上投射

从子类到超类的转换称为向上转换。典型地，向上转换是由编译器隐式执行的。

向上转换与继承密切相关 Java 中的另一个核心概念。使用引用变量来引用更具体的类型是很常见的。每当我们这样做的时候，隐式向上转换就会发生。

为了演示向上转换，让我们定义一个`Animal`类:

```
public class Animal {

    public void eat() {
        // ... 
    }
}
```

现在我们来延伸一下`Animal`:

```
public class Cat extends Animal {

    public void eat() {
         // ... 
    }

    public void meow() {
         // ... 
    }
}
```

现在我们可以创建一个`Cat`类的对象，并将其分配给`Cat`类型的引用变量:

```
Cat cat = new Cat();
```

我们也可以将它赋给类型为`Animal`的引用变量:

```
Animal animal = cat;
```

在上面的赋值中，发生了隐式向上转换。

我们可以明确地这样做:

```
animal = (Animal) cat;
```

但是没有必要对继承树进行显式的强制转换。编译器知道`cat`是一个`Animal`，并且不显示任何错误。

请注意，引用可以引用声明类型的任何子类型。

使用向上转换，我们限制了对`Cat`实例可用的方法数量，但是没有改变实例本身。现在我们不能做任何特定于`Cat`的事情——我们不能在`animal`变量上调用`meow()`。

虽然`Cat`对象仍然是`Cat`对象，但是调用`meow()`会导致编译器错误:

```
// animal.meow(); The method meow() is undefined for the type Animal
```

为了调用`meow()`,我们需要向下转换`animal`,稍后我们会这样做。

但是现在我们将描述是什么给了我们向上投射。由于向上转换，我们可以利用多态性。

### 3.1。多态性

让我们定义`Animal`的另一个子类，一个`Dog`类:

```
public class Dog extends Animal {

    public void eat() {
         // ... 
    }
}
```

现在我们可以定义`feed()`方法，它像对待`animals`一样对待所有的猫和狗:

```
public class AnimalFeeder {

    public void feed(List<Animal> animals) {
        animals.forEach(animal -> {
            animal.eat();
        });
    }
}
```

我们不希望`AnimalFeeder`关心哪个`animal`在列表上——是`Cat`还是`Dog`。在`feed()`方法中，它们都是`animals`。

当我们将特定类型的对象添加到`animals`列表时，就会发生隐式向上转换:

```
List<Animal> animals = new ArrayList<>();
animals.add(new Cat());
animals.add(new Dog());
new AnimalFeeder().feed(animals);
```

我们添加猫和狗，它们被隐式地向上转换为`Animal`类型。每个`Cat`是一个`Animal`，每个`Dog`是一个`Animal`。它们是多态的。

顺便说一下，所有的 Java 对象都是多态的，因为每个对象至少是一个`Object`。我们可以将`Animal`的一个实例赋给`Object`类型的引用变量，编译器不会抱怨:

```
Object object = new Animal();
```

这就是为什么我们创建的所有 Java 对象都已经有了特定于`Object`的方法，例如`toString()`。

向上转换到接口也很常见。

我们可以创建`Mew`接口并让`Cat`实现它:

```
public interface Mew {
    public void meow();
}

public class Cat extends Animal implements Mew {

    public void eat() {
         // ... 
    }

    public void meow() {
         // ... 
    }
}
```

现在任何`Cat`对象也可以向上投射到`Mew`:

```
Mew mew = new Cat();
```

`Cat`是一个`Mew`；向上转换是合法的，并且是隐式的。

因此，`Cat`是一个`Mew`、`Animal`、`Object`和`Cat`。在我们的例子中，它可以赋给所有四种类型的引用变量。

### 3.2。超越

在上面的例子中，`eat()`方法被覆盖。这意味着尽管在`Animal`类型的变量上调用了`eat()`,但是工作是通过在真实对象上调用的方法来完成的——猫和狗:

```
public void feed(List<Animal> animals) {
    animals.forEach(animal -> {
        animal.eat();
    });
}
```

如果我们在我们的类中添加一些日志记录，我们会看到`Cat`和`Dog`方法被调用:

```
web - 2018-02-15 22:48:49,354 [main] INFO com.baeldung.casting.Cat - cat is eating
web - 2018-02-15 22:48:49,363 [main] INFO com.baeldung.casting.Dog - dog is eating 
```

**总结一下:**

*   如果对象与变量类型相同或者是子类型，引用变量可以引用该对象。
*   向上转换是隐式发生的。
*   所有的 Java 对象都是多态的，由于向上转换，它们可以被视为超类型的对象。

## 4。向下投射

如果我们想使用类型为`Animal`的变量来调用只对`Cat`类可用的方法呢？失望来了。**这是从超类到子类的转换。**

让我们看一个例子:

```
Animal animal = new Cat();
```

我们知道`animal`变量指的是`Cat`的实例。我们想在`animal`上调用`Cat`的`meow()`方法。但是编译器抱怨类型`Animal`的`meow()`方法不存在。

要调用`meow()`，我们应该将`animal`降级为`Cat`:

```
((Cat) animal).meow();
```

内括号和它们包含的类型有时被称为转换运算符。请注意，编译代码还需要外部括号。

让我们用`meow()`方法重写前面的`AnimalFeeder` 例子:

```
public class AnimalFeeder {

    public void feed(List<Animal> animals) {
        animals.forEach(animal -> {
            animal.eat();
            if (animal instanceof Cat) {
                ((Cat) animal).meow();
            }
        });
    }
}
```

现在我们可以访问所有对`Cat`类可用的方法。查看日志以确保实际调用了`meow()`:

```
web - 2018-02-16 18:13:45,445 [main] INFO com.baeldung.casting.Cat - cat is eating
web - 2018-02-16 18:13:45,454 [main] INFO com.baeldung.casting.Cat - meow
web - 2018-02-16 18:13:45,455 [main] INFO com.baeldung.casting.Dog - dog is eating
```

注意，在上面的例子中，我们试图只向下转换那些实际上是`Cat`实例的对象。为此，我们使用操作符`instanceof`。

### 4.1。`instanceof`操作员

我们经常在向下转换之前使用`instanceof`运算符来检查对象是否属于特定类型:

```
if (animal instanceof Cat) {
    ((Cat) animal).meow();
}
```

### 4.2。`ClassCastException`

如果我们没有用`instanceof`操作符检查类型，编译器就不会抱怨。但是在运行时，会有一个例外。

为了演示这一点，让我们从上面的代码中删除`instanceof`操作符:

```
public void uncheckedFeed(List<Animal> animals) {
    animals.forEach(animal -> {
        animal.eat();
        ((Cat) animal).meow();
    });
}
```

这段代码编译没有问题。但是如果我们尝试运行它，我们会看到一个异常:

`java.lang.ClassCastException: com.baeldung.casting.Dog`不能被强制转换为`com.baeldung.casting.Cat`

这意味着我们正试图将一个作为`Dog`实例的对象转换成一个`Cat`实例。

如果我们向下转换到的类型与真实对象的类型不匹配，那么在运行时总是抛出。

请注意，如果我们尝试向下转换为不相关的类型，编译器不会允许这样做:

```
Animal animal;
String s = (String) animal;
```

编译器说“不能从动物强制转换为字符串。”

对于要编译的代码，这两种类型应该在同一个继承树中。

我们总结一下:

*   向下转换对于访问特定于子类的成员是必要的。
*   向下转换是使用 cast 运算符完成的。
*   为了安全地向下转换对象，我们需要`instanceof`操作符。
*   如果真实对象与我们向下转换到的类型不匹配，那么`ClassCastException`将在运行时抛出。

## 5。`cast()`法

还有另一种方法使用`Class`的方法来投射对象:

```
public void whenDowncastToCatWithCastMethod_thenMeowIsCalled() {
    Animal animal = new Cat();
    if (Cat.class.isInstance(animal)) {
        Cat cat = Cat.class.cast(animal);
        cat.meow();
    }
}
```

在上面的例子中，`cast(`和`isInstance()`方法被用来代替相应的 cast 和`instanceof`操作符。

对泛型类型使用`cast()`和`isInstance()`方法是很常见的。

让我们用`feed()`方法创建`AnimalFeederGeneric<T>`类，根据类型参数的值，只“喂养”一种动物，猫或狗:

```
public class AnimalFeederGeneric<T> {
    private Class<T> type;

    public AnimalFeederGeneric(Class<T> type) {
        this.type = type;
    }

    public List<T> feed(List<Animal> animals) {
        List<T> list = new ArrayList<T>();
        animals.forEach(animal -> {
            if (type.isInstance(animal)) {
                T objAsType = type.cast(animal);
                list.add(objAsType);
            }
        });
        return list;
    }

}
```

`feed()`方法检查每只动物，并只返回那些属于`T`实例的动物。

注意，`Class`实例也应该传递给泛型类，因为我们不能从类型参数`T`中获得它。在我们的例子中，我们在构造函数中传递它。

让我们让`T`等于`Cat`，并确保该方法只返回猫:

```
@Test
public void whenParameterCat_thenOnlyCatsFed() {
    List<Animal> animals = new ArrayList<>();
    animals.add(new Cat());
    animals.add(new Dog());
    AnimalFeederGeneric<Cat> catFeeder
      = new AnimalFeederGeneric<Cat>(Cat.class);
    List<Cat> fedAnimals = catFeeder.feed(animals);

    assertTrue(fedAnimals.size() == 1);
    assertTrue(fedAnimals.get(0) instanceof Cat);
}
```

## 6。结论

在这篇基础教程中，我们探索了向上转换、向下转换、如何使用它们以及这些概念如何帮助你利用多态性。

和往常一样，这篇文章的代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221205112022/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-oop-inheritance)