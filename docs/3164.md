# Java 警告“未检查的强制转换”

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-warning-unchecked-cast>

## 1.概观

有时，当我们编译我们的 Java 源文件时，我们会看到 Java 编译器打印的“`unchecked cast`”警告消息。

在本教程中，我们将仔细看看警告消息。我们将讨论这个警告意味着什么，为什么我们被警告，以及如何解决这个问题。

默认情况下，一些 Java 编译器会抑制未检查的警告。

在我们研究这个“`unchecked cast`”警告之前，让我们确保我们已经[启用了编译器的选项来打印“未检查的”警告](/web/20221130142140/https://www.baeldung.com/java-unchecked-conversion#enabling-the-unchecked-warning-option)。

## 2.`“unchecked cast”`警告是什么意思？

**`unchecked cast`是一个编译时警告**。简而言之，**在没有类型检查**的情况下，将原始类型转换为参数化类型时，我们会看到这个警告。

举个例子就能直白的说明。假设我们有一个简单的方法来返回原始类型`Map`:

```
public class UncheckedCast {
    public static Map getRawMap() {
        Map rawMap = new HashMap();
        rawMap.put("date 1", LocalDate.of(2021, Month.FEBRUARY, 10));
        rawMap.put("date 2", LocalDate.of(1992, Month.AUGUST, 8));
        rawMap.put("date 3", LocalDate.of(1976, Month.NOVEMBER, 18));
        return rawMap;
    }
...
}
```

现在，让我们创建一个测试方法来调用上面的方法，并将结果转换为`Map<String, LocalDate>`:

```
@Test
public void givenRawMap_whenCastToTypedMap_shouldHaveCompilerWarning() {
    Map<String, LocalDate> castFromRawMap = (Map<String, LocalDate>) UncheckedCast.getRawMap();
    Assert.assertEquals(3, castFromRawMap.size());
    Assert.assertEquals(castFromRawMap.get("date 2"), LocalDate.of(1992, Month.AUGUST, 8));
} 
```

编译器必须允许这种类型转换，以保持与不支持泛型的旧 Java 版本的向后兼容性。

但是如果我们编译 Java 源代码，编译器会打印出警告信息。接下来，让我们使用 Maven 编译并运行我们的单元测试:

```
$ mvn clean test
...
[WARNING] .../src/test/java/com/baeldung/uncheckedcast/UncheckedCastUnitTest.java:[14,97] unchecked cast
  required: java.util.Map<java.lang.String,java.time.LocalDate>
  found:    java.util.Map
...
[INFO] -------------------------------------------------------
[INFO]  T E S T S
[INFO] -------------------------------------------------------
...
[INFO] Results:
[INFO] 
[INFO] Tests run: 16, Failures: 0, Errors: 0, Skipped: 0
[INFO] 
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
... 
```

正如 Maven 输出所示，我们成功地重现了警告。

另一方面，即使我们看到“`unchecked cast`”编译器警告，我们的测试也没有任何问题。

我们知道编译器不会无缘无故地警告我们。当我们看到这个警告时，一定有一些潜在的问题。

让我们弄清楚。

## 3.为什么 Java 编译器会警告我们？

尽管我们看到了“`unchecked cast`”警告，但是我们的测试方法在前面的部分中工作得很好。这是因为当我们将原始类型`Map`转换为`Map<String, LocalDate>`时，原始类型`Map`只包含`<String, LocalDate>`条目。也就是说，类型转换是安全的。

为了分析潜在的问题，让我们稍微改变一下`getRawMap()`方法，在原始类型`Map`中增加一个条目:

```
public static Map getRawMapWithMixedTypes() {
    Map rawMap = new HashMap();
    rawMap.put("date 1", LocalDate.of(2021, Month.FEBRUARY, 10));
    rawMap.put("date 2", LocalDate.of(1992, Month.AUGUST, 8));
    rawMap.put("date 3", LocalDate.of(1976, Month.NOVEMBER, 18));
    rawMap.put("date 4", new Date());
    return rawMap;
} 
```

这一次，我们在上面的方法中向类型为`<String, Date>` 的`Map`添加了一个新条目。

现在，让我们编写一个新的测试方法来调用`getRawMapWithMixedTypes()`方法:

```
@Test(expected = ClassCastException.class)
public void givenMixTypedRawMap_whenCastToTypedMap_shouldThrowClassCastException() {
    Map<String, LocalDate> castFromRawMap = (Map<String, LocalDate>) UncheckedCast.getRawMapWithMixedTypes();
    Assert.assertEquals(4, castFromRawMap.size());
    Assert.assertTrue(castFromRawMap.get("date 4").isAfter(castFromRawMap.get("date 3")));
}
```

如果我们编译并运行测试，将再次显示“`unchecked cast`”警告消息。此外，我们的测试将通过。

然而，由于我们的测试有`expected = ClassCastException.class`参数，这意味着测试方法抛出了一个`ClassCastException`。

如果我们仔细观察一下，**`ClassCastException`并没有被扔在铸造原料类型`Map` 到`Map<String, LocalDate>`** 的线上，尽管警告信息指向这一行。相反，当我们通过键 : `castFromRawMap.get(“date 4”). `得到错误类型的数据时，异常就会发生

如果我们将包含错误类型数据的原始类型集合转换为参数化类型集合，那么 **`ClassCastException`不会被抛出，直到我们加载错误类型**的数据。

有时候，我们可能得到异常太晚了。

例如，通过调用我们的方法，我们得到一个具有许多条目的原始类型`Map` ，然后我们将它转换成一个具有参数化类型的`Map`:

```
(Map<String, LocalDate>) UncheckedCast.getRawMapWithMixedTypes()
```

对于`Map`中的每个条目，我们需要将`LocalDate`对象发送到远程 API。在我们遇到`ClassCastException`之前，很可能已经进行了很多 API 调用。根据要求，可能会涉及一些额外的恢复或数据清理过程。

如果我们能更早地得到异常，那就好了，这样我们就能决定如何处理类型错误的条目。

当我们理解了"`unchecked cast`"警告背后的潜在问题时，让我们看看我们能做些什么来解决这个问题。

## 4.我们应该如何处理这个警告？

### 4.1.避免使用原始类型

从 Java 5 开始就引入了泛型。如果我们的 Java 环境支持泛型，我们应该避免使用原始类型。这是因为**使用原始类型会让我们失去泛型所有的安全性和表达性优势。**

此外，我们应该搜索遗留代码，并将这些原始类型用法重构为泛型。

然而，有时我们不得不与一些旧的图书馆合作。来自那些旧的外部库的方法可能返回原始类型集合。

调用这些方法并强制转换为参数化类型将产生“`unchecked cast`”编译器警告。但是我们无法控制外部库。

接下来，让我们来看看如何处理这种情况。

### 4.2.抑制“`unchecked`”警告

如果我们不能消除“`unchecked cast`”警告，并且我们确信引发警告的代码是类型安全的，我们可以使用`SuppressWarnings(“unchecked”)`注释取消警告[。](/web/20221130142140/https://www.baeldung.com/java-unchecked-conversion#1-suppressing-the-warning)

**当我们使用@ `SuppressWarning(“unchecked”)`注释时，我们应该总是把它放在尽可能小的范围内。**

让我们以 [`ArrayList`](/web/20221130142140/https://www.baeldung.com/java-arraylist) 类中的`remove()`方法为例来看看:

```
public E remove(int index) {
    Objects.checkIndex(index, size);
    final Object[] es = elementData;

    @SuppressWarnings("unchecked") E oldValue = (E) es[index];
    fastRemove(es, index);

    return oldValue;
}
```

### 4.3.在使用原始类型集合之前进行类型安全检查

正如我们所了解的，`@SuppressWarning(“unchecked”)`注释仅仅隐藏了警告消息，而没有真正检查强制转换是否是类型安全的。

如果我们不确定转换一个原始类型是否是类型安全的，**我们应该[在我们真正使用数据之前](/web/20221130142140/https://www.baeldung.com/java-unchecked-conversion#2-checking-type-conversion-before-using-the-raw-type-collection)检查类型，这样我们可以更早的`ClassCastException`**。

## 5.结论

在本文中，我们已经了解了“`unchecked cast`”编译器警告的含义。

此外，我们已经解决了这个警告的原因以及如何解决潜在的问题。

和往常一样，这篇文章中的代码都可以在 GitHub 的[上找到。](https://web.archive.org/web/20221130142140/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-oop-generics)