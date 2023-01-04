# 将 Java 枚举转换成流

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-enumeration-to-stream>

## 1.概观

`Enumeration`是 Java 第一版(JDK 1.0)的接口。这个接口是通用的，**提供了对一系列元素**的惰性访问。尽管在较新版本的 Java 中有更好的选择，遗留实现仍然可能使用`Enumeration`接口返回结果。因此，为了使遗留实现现代化，开发人员可能必须将一个`Enumeration`对象转换成 [Java 流 API](/web/20220525135539/https://www.baeldung.com/java-streams) 。

在这个简短的教程中，我们将实现一个将`Enumeration`对象转换成 Java 流 API 的实用方法。因此，我们将能够使用流方法，如`filter`和`map`。

## 2.Java 的`Enumeration`接口

让我们以一个例子来说明`Enumeration`对象的使用:

```
public static <T> void print(Enumeration<T> enumeration) {
    while (enumeration.hasMoreElements()) {
        System.out.println(enumeration.nextElement());
    }
}
```

`Enumeration`有两种主要方法:`hasMoreElements`和`nextElement`。我们应该同时使用这两种方法来迭代元素集合。

## 3.创建一个`Spliterator`

作为第一步，我们将为抽象类`AbstractSpliterator`创建一个具体的类。这个类是使`Enumeration`对象适应`Spliterator`接口所必需的:

```
public class EnumerationSpliterator<T> extends AbstractSpliterator<T> {

    private final Enumeration<T> enumeration;

    public EnumerationSpliterator(long est, int additionalCharacteristics, Enumeration<T> enumeration) {
        super(est, additionalCharacteristics);
        this.enumeration = enumeration;
    }
}
```

除了创建类，我们还需要创建一个构造函数。我们应该将前两个参数传递给`super`构造函数。第一个参数是`Spliterator`的估计大小。第二个用于定义附加特征。最后，我们将使用最后一个参数来接收`Enumeration`对象。

我们还需要覆盖`tryAdvance`和`forEachRemaining`方法。它们将被`Stream` API 用来对`Enumeration`的元素执行动作:

```
@Override
public boolean tryAdvance(Consumer<? super T> action) {
    if (enumeration.hasMoreElements()) {
        action.accept(enumeration.nextElement());
        return true;
    }
    return false;
}

@Override
public void forEachRemaining(Consumer<? super T> action) {
    while (enumeration.hasMoreElements())
        action.accept(enumeration.nextElement());
}
```

## 4.将`Enumeration`转换为`Stream`

现在，使用`EnumerationSpliterator`类，我们能够使用`StreamSupport` API 来执行转换:

```
public static <T> Stream<T> convert(Enumeration<T> enumeration) {
    EnumerationSpliterator<T> spliterator 
      = new EnumerationSpliterator<T>(Long.MAX_VALUE, Spliterator.ORDERED, enumeration);
    Stream<T> stream = StreamSupport.stream(spliterator, false);

    return stream;
}
```

在这个实现中，我们需要创建一个`EnumerationSpliterator`类的实例。`Long.MAX_VALUE`是估计大小的默认值。`Spliterator.ORDERED`定义流将按照枚举提供的顺序迭代元素。

接下来，我们应该从`StreamSupport`类中调用`stream`方法。我们需要将`EnumerationSpliterator`实例作为第一个参数传递。最后一个参数是定义流是并行的还是顺序的。

## 5.测试我们的实现

通过测试我们的`convert`方法，我们可以观察到现在我们能够基于`Enumeration`创建一个有效的`Stream`对象:

```
@Test
public void givenEnumeration_whenConvertedToStream_thenNotNull() {
    Vector<Integer> input = new Vector<>(Arrays.asList(1, 2, 3, 4, 5));

    Stream<Integer> resultingStream = convert(input.elements());

    Assert.assertNotNull(resultingStream);
}
```

## 6.结论

在本教程中，我们展示了如何将一个`Enumeration`转换成一个`Stream`对象。源代码一如既往地可以在 GitHub 上找到[。](https://web.archive.org/web/20220525135539/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-streams-3)