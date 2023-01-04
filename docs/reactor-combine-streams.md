# 在 Project Reactor 中组合发布者

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/reactor-combine-streams>

## 1。概述

在本文中，我们将看看在[项目反应器](https://web.archive.org/web/20220630142440/https://projectreactor.io/)中组合`Publishers`的各种方式。

## 2。Maven 依赖关系

让我们用[项目反应器](https://web.archive.org/web/20220630142440/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22io.projectreactor%22%20AND%20a%3A%22reactor-core%22)的依赖关系来设置我们的例子:

```java
<dependency>
    <groupId>io.projectreactor</groupId>
    <artifactId>reactor-core</artifactId>
    <version>3.1.4.RELEASE</version>
</dependency>
<dependency>
    <groupId>io.projectreactor</groupId>
    <artifactId>reactor-test</artifactId>
    <version>3.1.4.RELEASE</version>
    <scope>test</scope>
</dependency>
```

## 3。合并出版商

给定一个必须使用`Flux<T>`或`Mono<T>`的场景，有不同的方式来组合流。

让我们创建几个例子来说明静态方法在`Flux<T>`类中的用法，比如`concat, concatWith, merge, zip` 和`combineLatest.`

我们的例子将使用两个类型为`Flux<Integer>`的发布者，即`evenNumbers`，它是`Integer`的一个`Flux`，拥有一个从 1 ( `min`变量)开始并受限于 5 ( `max`变量)的偶数序列。

我们将创建`oddNumbers`，也是奇数的`Integer`类型的`Flux`:

```java
Flux<Integer> evenNumbers = Flux
  .range(min, max)
  .filter(x -> x % 2 == 0); // i.e. 2, 4

Flux<Integer> oddNumbers = Flux
  .range(min, max)
  .filter(x -> x % 2 > 0);  // ie. 1, 3, 5
```

### 3.1。`oncat()`c

`concat`方法执行输入的连接，转发下游源发出的元素。

通过顺序订阅第一个源，然后在订阅下一个源之前等待它完成，以此类推，直到最后一个源完成，可以实现串联。任何错误都会立即中断序列，并被转发到下游。

这里有一个简单的例子:

```java
@Test
public void givenFluxes_whenConcatIsInvoked_thenConcat() {
    Flux<Integer> fluxOfIntegers = Flux.concat(
      evenNumbers, 
      oddNumbers);

    StepVerifier.create(fluxOfIntegers)
      .expectNext(2)
      .expectNext(4)
      .expectNext(1)
      .expectNext(3)
      .expectNext(5)
      .expectComplete()
      .verify();
}
```

### 3.2。 `**concatWith()**`

使用静态方法`concatWith`，我们将产生两个类型为`Flux<T>`的源的串联结果:

```java
@Test
public void givenFluxes_whenConcatWithIsInvoked_thenConcatWith() {
    Flux<Integer> fluxOfIntegers = evenNumbers.concatWith(oddNumbers);

    // same stepVerifier as in the concat example above
}
```

### 3.3。`ombineLatest()`c

Flux 静态方法`combineLatest`将生成由来自每个发布者源的最近发布的值的组合提供的数据。

以下是使用两个`Publisher`源和一个`BiFunction`作为参数的方法的示例:

```java
@Test
public void givenFluxes_whenCombineLatestIsInvoked_thenCombineLatest() {
    Flux<Integer> fluxOfIntegers = Flux.combineLatest(
      evenNumbers, 
      oddNumbers, 
      (a, b) -> a + b);

    StepVerifier.create(fluxOfIntegers)
      .expectNext(5) // 4 + 1
      .expectNext(7) // 4 + 3
      .expectNext(9) // 4 + 5
      .expectComplete()
      .verify();
}
```

我们在这里可以看到，函数`combineLatest`使用了`evenNumbers` ( `4`)的最新元素和`oddNumbers (1,3,5)`的元素应用了函数“a + b”，从而生成了序列`5,7,9`。

### 3.4。`merge()`

`merge` 函数将包含在数组中的`Publisher`序列的数据合并到一个交错的合并序列中:

```java
@Test
public void givenFluxes_whenMergeIsInvoked_thenMerge() {
    Flux<Integer> fluxOfIntegers = Flux.merge(
      evenNumbers, 
      oddNumbers);

    StepVerifier.create(fluxOfIntegers)
      .expectNext(2)
      .expectNext(4)
      .expectNext(1)
      .expectNext(3)
      .expectNext(5)
      .expectComplete()
      .verify();
}
```

值得注意的一件有趣的事情是，与`concat (`懒惰的订阅`)`相反，源代码被热切地订阅。

这里，如果我们在发布者的元素之间插入一个延迟，我们可以看到`merge`函数的不同结果:

```java
@Test
public void givenFluxes_whenMergeWithDelayedElementsIsInvoked_thenMergeWithDelayedElements() {
    Flux<Integer> fluxOfIntegers = Flux.merge(
      evenNumbers.delayElements(Duration.ofMillis(500L)), 
      oddNumbers.delayElements(Duration.ofMillis(300L)));

    StepVerifier.create(fluxOfIntegers)
      .expectNext(1)
      .expectNext(2)
      .expectNext(3)
      .expectNext(5)
      .expectNext(4)
      .expectComplete()
      .verify();
} 
```

### 3.5.**T2`mergeSequential()`**

`mergeSequential`方法将来自数组中提供的`Publisher`序列的数据合并到一个有序的合并序列中。

与`concat`不同的是，sources 订阅踊跃。

此外，与`merge`不同，它们发出的值按照订阅顺序合并到最终序列中:

```java
@Test
public void testMergeSequential() {
    Flux<Integer> fluxOfIntegers = Flux.mergeSequential(
      evenNumbers, 
      oddNumbers);

    StepVerifier.create(fluxOfIntegers)
      .expectNext(2)
      .expectNext(4)
      .expectNext(1)
      .expectNext(3)
      .expectNext(5)
      .expectComplete()
      .verify();
}
```

### 3.6.**T2`mergeDelayError()`**

`mergeDelayError`将数组中包含的发布者序列的数据合并到一个交错的合并序列中。

与`concat`不同的是，sources 订阅踊跃。

静态`merge`方法的这种变体将延迟任何错误，直到剩余的合并待办事项处理完毕。

这里有一个`mergeDelayError:`的例子

```java
@Test
public void givenFluxes_whenMergeWithDelayedElementsIsInvoked_thenMergeWithDelayedElements() {
    Flux<Integer> fluxOfIntegers = Flux.mergeDelayError(1, 
      evenNumbers.delayElements(Duration.ofMillis(500L)), 
      oddNumbers.delayElements(Duration.ofMillis(300L)));

    StepVerifier.create(fluxOfIntegers)
      .expectNext(1)
      .expectNext(2)
      .expectNext(3)
      .expectNext(5)
      .expectNext(4)
      .expectComplete()
      .verify();
}
```

### `**3.7\. mergeWith()**`

静态方法`mergeWith`将来自这个`Flux`和一个`Publisher`的数据合并成一个交错的合并序列。

同样，与`concat`不同的是，内部资源被急切地订阅:

```java
@Test
public void givenFluxes_whenMergeWithIsInvoked_thenMergeWith() {
    Flux<Integer> fluxOfIntegers = evenNumbers.mergeWith(oddNumbers);

    // same StepVerifier as in "3.4\. Merge"
    StepVerifier.create(fluxOfIntegers)
      .expectNext(2)
      .expectNext(4)
      .expectNext(1)
      .expectNext(3)
      .expectNext(5)
      .expectComplete()
      .verify();
}
```

### 3.8。z `ip()`

静态方法`zip`将多个源粘合在一起，即等待所有源发出一个元素，并将这些元素组合成一个输出值(由提供的 combinator 函数构造)。

操作员将继续这样做，直到任何源完成:

```java
@Test
public void givenFluxes_whenZipIsInvoked_thenZip() {
    Flux<Integer> fluxOfIntegers = Flux.zip(
      evenNumbers, 
      oddNumbers, 
      (a, b) -> a + b);

    StepVerifier.create(fluxOfIntegers)
      .expectNext(3) // 2 + 1
      .expectNext(7) // 4 + 3
      .expectComplete()
      .verify();
}
```

由于没有来自`evenNumbers`的元素需要配对，来自`oddNumbers` Publisher 的元素 5 被忽略。

### 3.9。z `ipWith()`

`zipWith` 执行与`zip`相同的方法，但是只有两个发布者:

```java
@Test
public void givenFluxes_whenZipWithIsInvoked_thenZipWith() {
    Flux<Integer> fluxOfIntegers = evenNumbers
     .zipWith(oddNumbers, (a, b) -> a * b);

    StepVerifier.create(fluxOfIntegers)
      .expectNext(2)  // 2 * 1
      .expectNext(12) // 4 * 3
      .expectComplete()
      .verify();
}
```

## 4。结论

在这个快速教程中，我们发现了组合`Publishers.`的多种方式

和往常一样，这些例子可以在 GitHub 的[中找到。](https://web.archive.org/web/20220630142440/https://github.com/eugenp/tutorials/tree/master/reactor-core)