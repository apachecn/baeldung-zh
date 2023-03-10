# Java 中的原型模式

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-pattern-prototype>

## 1.介绍

在本教程中，我们将学习一种[创造性设计模式](/web/20221205164615/https://www.baeldung.com/creational-design-patterns)——原型模式。首先，我们将解释这个模式，然后继续用 Java 实现它。

我们还将讨论它的一些优点和缺点。

## 2.原型模式

当我们有一个类的实例(原型)并且我们想要通过复制原型来创建新的对象时，通常使用原型模式**。**

让我们用一个类比来更好地理解这个模式。

在一些游戏中，我们希望背景中有树木或建筑物。我们可能会意识到，我们不必在角色每次移动时都创建新的树或建筑并在屏幕上渲染它们。

因此，我们首先创建树的一个实例。然后我们可以从这个实例(原型)创建尽可能多的树，并更新它们的位置。我们也可以选择改变树的颜色来提升游戏等级。

原型模式非常相似。不需要创建新的对象，我们只需要克隆原型实例。

## 3.UML 图

[![Prototype Pattern](img/e532e63c80362a2b6986eda9c4b97254.png)](/web/20221205164615/https://www.baeldung.com/wp-content/uploads/2019/10/Prototype-Pattern.png)

在图中，我们看到客户端告诉原型克隆自己并创建一个对象。`Prototype`是一个接口，声明了一个克隆自身的方法。`ConcretePrototype1` 和`ConcretePrototype2`执行克隆自身的操作。

## 4.履行

我们在 Java 中实现这种模式的方法之一是使用`clone()`方法。为此，我们将实现`Cloneable`接口。

当我们试图克隆时，**我们应该决定是进行浅层复制还是深层复制。最终，归结为需求。**

例如，如果类只包含[原语](/web/20221205164615/https://www.baeldung.com/java-primitives)和[不可变字段](/web/20221205164615/https://www.baeldung.com/java-immutable-object)，我们可以使用浅层拷贝。

如果它包含对可变字段的引用，我们应该使用[深度拷贝](/web/20221205164615/https://www.baeldung.com/java-deep-copy)。我们可以用 **[复制构造函数](/web/20221205164615/https://www.baeldung.com/java-copy-constructor)或[序列化和反序列化](/web/20221205164615/https://www.baeldung.com/java-serialization)来实现。**

让我们以前面提到的例子为例，继续看看如何在不使用`Cloneable`接口的情况下应用原型模式。为了做到这一点，让我们用`abstract`方法`‘copy'`创建一个名为`Tree` 的`abstract`类。

```java
public abstract class Tree {

    // ...
    public abstract Tree copy();

}
```

现在假设我们有两个不同的`Tree`实现，分别叫做`PlasticTree`和`PineTree`:

```java
public class PlasticTree extends Tree {

    // ...

    @Override
    public Tree copy() {
        PlasticTree plasticTreeClone = new PlasticTree(this.getMass(), this.getHeight());
        plasticTreeClone.setPosition(this.getPosition());
        return plasticTreeClone;
    }

}
```

```java
public class PineTree extends Tree {
    // ...

    @Override
    public Tree copy() {
        PineTree pineTreeClone = new PineTree(this.getMass(), this.getHeight());
        pineTreeClone.setPosition(this.getPosition());
        return pineTreeClone;
    }
}
```

所以这里我们看到扩展了`Tree`并实现了`copy`方法的类可以作为原型来创建它们自己的副本。

原型模式也让我们创建对象的副本，而不依赖于具体的类。假设我们有一个树的列表，我们想要创建它们的副本。由于多态性，我们可以很容易地创建多个副本，而不需要知道树的类型。

## 5.测试

现在我们来测试一下:

```java
public class TreePrototypesUnitTest {

    @Test
    public void givenAPlasticTreePrototypeWhenClonedThenCreateA_Clone() {
        // ...

        PlasticTree plasticTree = new PlasticTree(mass, height);
        plasticTree.setPosition(position);
        PlasticTree anotherPlasticTree = (PlasticTree) plasticTree.copy();
        anotherPlasticTree.setPosition(otherPosition);

        assertEquals(position, plasticTree.getPosition());
        assertEquals(otherPosition, anotherPlasticTree.getPosition());
    }
}
```

我们看到树已经从原型中克隆出来，我们有两个不同的`PlasticTree`实例。我们只是更新了克隆中的位置，并保留了其他值。

现在让我们克隆一个树的列表:

```java
@Test
public void givenA_ListOfTreesWhenClonedThenCreateListOfClones() {

    // create instances of PlasticTree and PineTree

    List<Tree> trees = Arrays.asList(plasticTree, pineTree);
    List<Tree> treeClones = trees.stream().map(Tree::copy).collect(toList());

    // ...

    assertEquals(height, plasticTreeClone.getHeight());
    assertEquals(position, plasticTreeClone.getPosition());
}
```

请注意，我们能够在这里对列表进行深层复制，而不依赖于`Tree.`的具体实现

## 6.优点和缺点

当我们的新对象与我们现有的对象仅略有不同时，这种模式非常方便。在某些情况下，实例在一个类中可能只有几种状态组合。因此，**我们可以预先创建具有适当状态的实例，然后在需要的时候克隆它们，而不是创建新的实例。**

有时，我们可能会遇到仅在状态上不同的子类。我们可以通过创建具有初始状态的原型，然后克隆它们来消除这些子类。

原型模式，就像其他设计模式一样，应该只在合适的时候使用。由于我们是在克隆对象，当有许多类时，这个过程会变得复杂，从而导致混乱。此外，很难克隆具有循环引用的类。

## 7.结论

在本教程中，我们学习了原型模式的关键概念，并了解了如何用 Java 实现它。我们也讨论了它的一些利弊。

和往常一样，这篇文章的源代码可以在 Github 的[上找到。](https://web.archive.org/web/20221205164615/https://github.com/eugenp/tutorials/tree/master/patterns-modules/design-patterns-creational)