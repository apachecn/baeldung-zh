# 断言对象来自特定类型

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-assert-object-of-type>

## 1.概观

在本文中，我们将探讨如何验证一个对象是否属于特定类型。我们将会看到不同的测试库，以及它们提供了什么方法来断言对象类型。

我们可能需要这样做的场景可能会有所不同。一个常见的例子是当我们使用一个接口作为方法的返回类型，但是根据返回的特定对象，我们想要执行不同的操作。单元测试可以帮助我们确定返回的对象是否有我们期望的类。

## 2.示例场景

让我们想象一下，我们正在根据树木是否会在冬天落叶进行分类。我们有两个类，`Evergreen `和`Deciduous,` 都实现了一个`Tree`接口`. `我们有一个简单的排序器，它根据树的名称返回正确的类型:

```java
Tree sortTree(String name) {

    List<String> deciduous = List.of("Beech", "Birch", "Ash", "Whitebeam", "Hornbeam", "Hazel & Willow");
    List<String> evergreen = List.of("Cedar", "Holly", "Laurel", "Olive", "Pine");

    if (deciduous.contains(name)) {
        return new Deciduous(name);
    } else if (evergreen.contains(name)) {
        return new Evergreen(name);
    } else {
        throw new RuntimeException("Tree could not be classified");
    }
}
```

让我们探索一下如何测试实际返回的是什么类型的`Tree`。

### 2.1.用 JUnit5 测试

如果我们想使用 [JUnit5](/web/20220524035421/https://www.baeldung.com/junit-5) ，**，我们可以通过使用`assertEquals`方法**来检查我们对象的类是否等于我们正在测试的类:

```java
@Test
public void sortTreeShouldReturnEvergreen_WhenPineIsPassed() {
    Tree tree = tested.sortTree("Pine");
    assertEquals(tree.getClass(), Evergreen.class);
} 
```

### 2.2.用 Hamcrest 测试

在使用 [Hamcrest](/web/20220524035421/https://www.baeldung.com/java-junit-hamcrest-guide) 库的时候，**我们可以使用 assertThat 和`instanceOf `方法**:

```java
@Test
public void sortTreeShouldReturnEvergreen_WhenPineIsPassed() {
Tree tree = tested.sortTree("Pine");
assertThat(tree, instanceOf(Evergreen.class));
}
```

当我们用`org.hamcrest.Matchers.isA` `:`导入时，有一个快捷版本可供我们使用

```java
assertThat(tree, isA(Evergreen.class));
```

### 2.3.用 AssertJ 测试

我们也可以使用 [AssertJ 核心](/web/20220524035421/https://www.baeldung.com/introduction-to-assertj)库的`isExactlyInstanceOf`方法:

```java
@Test
public void sortTreeShouldReturnEvergreen_WhenPineIsPassed() {
    Tree tree = tested.sortTree("Pine");
    assertThat(tree).isExactlyInstanceOf(Evergreen.class);
}
```

完成**相同测试的另一种方法是使用`hasSameClassAs` 方法**:

```java
@Test
public void sortTreeShouldReturnDecidious_WhenBirchIsPassed() {
    Tree tree = tested.sortTree("Birch");
    assertThat(tree).hasSameClassAs(new Deciduous("Birch"));
}
```

## 3.结论

在本教程中，我们已经看到了一些在单元测试中验证对象类型的不同例子。我们已经展示了一个简单的`Junit5`例子以及使用`Hamcrest`和`AssertJ`库的方法。`Hamcrest`和`AssertJ`都在它们的错误消息中提供了额外的有用信息。

和往常一样，这个例子的代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220524035421/https://github.com/eugenp/tutorials/tree/master/testing-modules/testing-assertions)