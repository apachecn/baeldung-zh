# Java 中的静态和动态绑定

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-static-dynamic-binding>

## 1.介绍

[多态性](/web/20220628101410/https://www.baeldung.com/java-polymorphism)允许一个对象采取多种形式——当一个方法表现出多态性时，编译器必须将该方法的名称映射到最终的实现。

如果它是在编译时映射的，那么它就是一个静态或早期绑定。

如果它是在运行时解决的，它被称为动态或后期绑定。

## 2.通过代码理解

当子类扩展超类时，它可以重新实现它所定义的方法。这被称为方法重写。

例如，让我们创建一个超类`Animal:`

```java
public class Animal {

    static Logger logger = LoggerFactory.getLogger(Animal.class);

    public void makeNoise() {
        logger.info("generic animal noise");
    }

    public void makeNoise(Integer repetitions) {
        while(repetitions != 0) {
            logger.info("generic animal noise countdown " + repetitions);
            repetitions -= 1;
        }
    }
}
```

和子类`Dog`:

```java
public class Dog extends Animal {

    static Logger logger = LoggerFactory.getLogger(Dog.class);

    @Override
    public void makeNoise() {
        logger.info("woof woof!");
    }

}
```

在重载一个方法时，比如`Animal`类的`makeNoise()`，编译器将在编译时解析该方法及其代码。**这是静态绑定的一个例子。**

然而，如果我们将类型为`Dog`的对象赋给类型为`Animal`的引用，编译器将在运行时解析函数代码映射。这就是动态绑定。

为了理解这是如何工作的，让我们编写一小段代码来调用这些类及其方法:

```java
Animal animal = new Animal();

// calling methods of animal object
animal.makeNoise();
animal.makeNoise(3);

// assigning a dog object to reference of type Animal
Animal dogAnimal = new Dog();

dogAnimal.makeNoise();

The output of the above code will be:
```

```java
com.baeldung.binding.Animal - generic animal noise 
com.baeldung.binding.Animal - generic animal noise countdown 3
com.baeldung.binding.Animal - generic animal noise countdown 2
com.baeldung.binding.Animal - generic animal noise countdown 1
com.baeldung.binding.Dog - woof woof!
```

现在，让我们创建一个类:

```java
class AnimalActivity {

    public static void eat(Animal animal) {
        System.out.println("Animal is eating");
    }

    public static void eat(Dog dog) {
        System.out.println("Dog is eating");
    }
}
```

让我们将这一行添加到主类中:

```java
AnimalActivity.eat(dogAnimal);
```

输出将是:

```java
com.baeldung.binding.AnimalActivity - Animal is eating
```

**这个例子显示了静态函数经历了静态绑定**。

原因是子类不能覆盖静态方法。如果子类实现了相同的方法，它将隐藏超类的方法。类似地，如果一个方法是最终的或者私有的，JVM 将会做一个静态绑定。

静态绑定方法不与特定对象相关联，而是在`Type`(Java 中的类)上调用。这种方法的执行稍微快一些。

默认情况下，任何其他方法都是 Java 中的虚方法。JVM 在运行时解析这些方法，这就是动态绑定。

确切的实现依赖于 JVM，但是它将采用类似 C++的方法，其中 JVM 查找虚拟表来决定在哪个对象上调用该方法。

## 3.结论

绑定是实现多态的语言不可或缺的一部分，理解静态和动态绑定的含义以确保我们的应用程序如我们所愿地运行是很重要的。

然而，有了这种理解，我们就能够有效地使用类继承和方法重载。

和往常一样，代码可以在 GitHub 的[上获得。](https://web.archive.org/web/20220628101410/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-oop-others)