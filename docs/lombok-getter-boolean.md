# 对布尔字段使用 Lombok 的@Getter

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/lombok-getter-boolean>

## 1。简介

Project Lombok 是一个减少 Java 样板文件的流行库。

在这个快速教程中，我们将看看 Lombok 的 [`@Getter`](https://web.archive.org/web/20221128045848/https://projectlombok.org/features/GetterSetter) 注释如何对布尔字段进行操作，以消除创建相应 getter 方法的需要。

## 2。Maven 依赖关系

让我们从将[项目龙目岛](https://web.archive.org/web/20221128045848/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.projectlombok%22)添加到我们的`pom.xml`开始:

```java
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <version>1.18.20</version>
</dependency>
```

## 3。在`boolean`字段上使用@Getter

假设我们希望 Lombok 为我们的私有布尔字段生成一个访问器方法。

我们可以用`@Getter`来注释这个字段:

```java
@Getter
private boolean running;
```

Lombok 将使用它的[注释处理器](/web/20221128045848/https://www.baeldung.com/java-annotation-processing-builder)在类中生成一个`isRunning()`方法。

现在，我们可以引用它，即使我们没有自己编写方法:

```java
@Test
public void whenBasicBooleanField_thenMethodNamePrefixedWithIsFollowedByFieldName() {
    LombokExamples lombokExamples = new LombokExamples();
    assertFalse(lombokExamples.isRunning());
}
```

### 3.1。与其访问器同名的`boolean`字段

让我们添加另一行代码，使示例变得稍微复杂一点:

```java
@Getter
private boolean isRunning = true;
```

如果 Lombok 创建了一个名为`isIsRunning`的方法，那就有点麻烦了。

相反，龙目岛像以前一样创造了`isRunning` :

```java
@Test
public void whenBooleanFieldPrefixedWithIs_thenMethodNameIsSameAsFieldName() {
    LombokExamples lombokExamples = new LombokExamples();
    assertTrue(lombokExamples.isRunning());
}
```

### 3.2。具有相同访问器名称的两个`boolean`字段

有时候，会有冲突。

假设我们需要在同一个类中包含以下行:

```java
 @Getter
    public boolean running = true;

    @Getter
    public boolean isRunning = false;
```

我们应该避免像这样令人困惑的命名约定，原因有很多。其中之一是它为龙目岛制造了冲突。

使用 Lombok 的约定，这两个字段将具有相同的访问器方法名:`isRunning`。但是在同一个类中有两个同名的方法会产生编译器错误。

Lombok 通过只创建一个访问器方法来解决这个问题，在这种情况下，根据字段声明顺序将它指向`running, `:

```java
@Test
public void whenTwoBooleanFieldsCauseNamingConflict_thenLombokMapsToFirstDeclaredField() {
    LombokExamples lombokExamples = new LombokExamples();
    assertTrue(lombokExamples.isRunning() == lombokExamples.running);
    assertFalse(lombokExamples.isRunning() == lombokExamples.isRunning);
}
```

## 4。在`Boolean`字段上使用@Getter

现在，Lombok 对`Boolean `类型的处理略有不同。

让我们最后一次尝试相同的运行示例，但是用`Boolean `代替原始类型:

```java
@Getter
private Boolean running;
```

Lombok 不会创建`isRunning`，而是会生成`getRunning`:

```java
@Test
public void whenFieldOfBooleanType_thenLombokPrefixesMethodWithGetInsteadOfIs() {
    LombokExamples lombokExamples = new LombokExamples();
    assertTrue(lombokExamples.getRunning());
}
```

## 5。结论

在本文中，我们探讨了如何将 Lombok 的@Getter 注释用于布尔原语和布尔对象。

请务必查看 Github 上的样本[。](https://web.archive.org/web/20221128045848/https://github.com/eugenp/tutorials/tree/master/lombok-modules/lombok)