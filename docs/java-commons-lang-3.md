# Apache Commons Lang 3 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-commons-lang-3>

## 1。概述

Apache Commons Lang 3 library**是一个流行的、功能齐全的实用程序类包，旨在扩展 Java API 的功能**。

该库的清单非常丰富，从字符串、数组和数字操作、反射和并发，到几个有序数据结构的实现，如 pairs 和 triples(一般称为 [tuples](https://web.archive.org/web/20221229040443/https://en.wikipedia.org/wiki/Tuple) )。

在本教程中，**我们将深入探讨该库最有用的实用程序类**。

## 2。Maven 依赖关系

像往常一样，要开始使用 Apache Commons Lang 3，我们首先需要添加 [Maven 依赖项](https://web.archive.org/web/20221229040443/https://search.maven.org/search?q=g:org.apache.commons%20AND%20a:commons-lang3&core=gav):

```java
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-lang3</artifactId>
    <version>3.12.0</version>
</dependency>
```

## 3。`StringUtils`班

在这篇介绍性综述中，我们将涉及的第一个实用程序类是`[StringUtils](https://web.archive.org/web/20221229040443/https://commons.apache.org/proper/commons-lang/apidocs/org/apache/commons/lang3/StringUtils.html).`

顾名思义， **`StringUtils`允许我们执行一系列空安全的`trings`操作，补充/扩展`[java.lang.String](https://web.archive.org/web/20221229040443/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/String.html)`提供的开箱即用的**操作。

让我们开始展示对给定的`string`执行几项检查的一组实用方法，例如确定`string`是空白、空、小写、大写、字母数字等等:

```java
@Test
public void whenCalledisBlank_thenCorrect() {
    assertThat(StringUtils.isBlank(" ")).isTrue();
}

@Test
public void whenCalledisEmpty_thenCorrect() {
    assertThat(StringUtils.isEmpty("")).isTrue();
}

@Test
public void whenCalledisAllLowerCase_thenCorrect() {
    assertThat(StringUtils.isAllLowerCase("abd")).isTrue();
}

@Test
public void whenCalledisAllUpperCase_thenCorrect() {
    assertThat(StringUtils.isAllUpperCase("ABC")).isTrue();
}

@Test
public void whenCalledisMixedCase_thenCorrect() {
    assertThat(StringUtils.isMixedCase("abC")).isTrue();
}

@Test
public void whenCalledisAlpha_thenCorrect() {
    assertThat(StringUtils.isAlpha("abc")).isTrue();
}

@Test
public void whenCalledisAlphanumeric_thenCorrect() {
    assertThat(StringUtils.isAlphanumeric("abc123")).isTrue();
} 
```

当然，`StringUtils`类实现了许多其他方法，为了简单起见，我们在这里省略了这些方法。

对于其他一些检查或应用某种类型的转换算法到给定的`string`的方法，请[查看本教程](/web/20221229040443/https://www.baeldung.com/string-processing-commons-lang)。

我们上面提到的那些非常简单，所以单元测试应该是不言自明的。

## 4。`ArrayUtils`班

**[`ArrayUtils`](https://web.archive.org/web/20221229040443/https://commons.apache.org/proper/commons-lang/javadocs/api-3.6/index.html?org/apache/commons/lang3/ArrayUtils.html)类实现了一批实用方法，允许我们处理和检查许多不同形状和形式的数组**。

让我们从`the toString()`方法的两个重载实现开始，当`array`为空时，它返回给定`array` 的一个`string`表示和一个特定的`string`:

```java
@Test
public void whenCalledtoString_thenCorrect() {
    String[] array = {"a", "b", "c"};
    assertThat(ArrayUtils.toString(array))
      .isEqualTo("{a,b,c}");
}

@Test
public void whenCalledtoStringIfArrayisNull_thenCorrect() {
    assertThat(ArrayUtils.toString(null, "Array is null"))
      .isEqualTo("Array is null");
} 
```

接下来，我们有`hasCode()`和`toMap()`方法。

前者为一个`array,`生成一个定制的 [hashCode](/web/20221229040443/https://www.baeldung.com/java-hashcode) 实现，而后者将一个`array`转换成一个`[Map](https://web.archive.org/web/20221229040443/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Map.html)`:

```java
@Test
public void whenCalledhashCode_thenCorrect() {
    String[] array = {"a", "b", "c"};
    assertThat(ArrayUtils.hashCode(array))
      .isEqualTo(997619);
}

@Test
public void whenCalledtoMap_thenCorrect() {
    String[][] array = {{"1", "one", }, {"2", "two", }, {"3", "three"}};
    Map map = new HashMap();
    map.put("1", "one");
    map.put("2", "two");
    map.put("3", "three");
    assertThat(ArrayUtils.toMap(array))
      .isEqualTo(map);
}
```

最后，让我们看看`isSameLength()` 和`indexOf()`方法。

前者用于检查两个数组是否具有相同的长度，后者用于获取给定元素的索引:

```java
@Test
public void whenCalledisSameLength_thenCorrect() {
    int[] array1 = {1, 2, 3};
    int[] array2 = {1, 2, 3};
    assertThat(ArrayUtils.isSameLength(array1, array2))
      .isTrue();
}

@Test
public void whenCalledIndexOf_thenCorrect() {
    int[] array = {1, 2, 3};
    assertThat(ArrayUtils.indexOf(array, 1, 0))
      .isEqualTo(0);
} 
```

与`StringUtils`类一样，`ArrayUtils`实现了更多的附加方法。你可以在[这个教程](/web/20221229040443/https://www.baeldung.com/array-processing-commons-lang)中了解更多。

在这种情况下，我们只展示了最具代表性的。

## 5。`NumberUtils`班

Apache Commons Lang 3 的另一个关键组件是 [NumberUtils](https://web.archive.org/web/20221229040443/https://commons.apache.org/proper/commons-lang/apidocs/org/apache/commons/lang3/math/NumberUtils.html) 类。

正如所料，**该类提供了大量的实用方法，旨在处理和操作数字类型**。

让我们看看`compare()`方法的重载实现，它比较不同原语的相等性，比如`int`和`long`:

```java
@Test
public void whenCalledcompareWithIntegers_thenCorrect() {
    assertThat(NumberUtils.compare(1, 1))
      .isEqualTo(0);
}

@Test
public void whenCalledcompareWithLongs_thenCorrect() {
    assertThat(NumberUtils.compare(1L, 1L))
      .isEqualTo(0);
}
```

此外，还有对`byte`和`short`进行操作的`compare()`的实现，其工作方式与上面的例子非常相似。

接下来是 `createNumber()`和`isDigit()`方法。

第一个允许我们创建一个`string`的数字表示，而第二个检查一个`string`是否只由数字组成:

```java
@Test
public void whenCalledcreateNumber_thenCorrect() {
    assertThat(NumberUtils.createNumber("123456"))
      .isEqualTo(123456);
}

@Test
public void whenCalledisDigits_thenCorrect() {
    assertThat(NumberUtils.isDigits("123456")).isTrue();
} 
```

在查找所提供数组的 mix 和 max 值时，`NumberUtils`类通过重载实现`min()`和`max()`方法为这些操作提供了强大的支持:

```java
@Test
public void whenCalledmaxwithIntegerArray_thenCorrect() {
    int[] array = {1, 2, 3, 4, 5, 6};
    assertThat(NumberUtils.max(array))
      .isEqualTo(6);
}

@Test
public void whenCalledminwithIntegerArray_thenCorrect() {
    int[] array = {1, 2, 3, 4, 5, 6};
    assertThat(NumberUtils.min(array)).isEqualTo(1);
}

@Test
public void whenCalledminwithByteArray_thenCorrect() {
    byte[] array = {1, 2, 3, 4, 5, 6};
    assertThat(NumberUtils.min(array))
      .isEqualTo((byte) 1);
}
```

## 6。 **`Fraction`级**

当我们使用一支笔和一张纸时，处理分数是非常好的。但是，在编写代码时，我们必须经历这个过程的复杂性吗？不完全是。

**`[Fraction](https://web.archive.org/web/20221229040443/https://commons.apache.org/proper/commons-lang/apidocs/org/apache/commons/lang3/math/Fraction.html)`类轻松完成分数的加减乘除**:

```java
@Test
public void whenCalledgetFraction_thenCorrect() {
    assertThat(Fraction.getFraction(5, 6)).isInstanceOf(Fraction.class);
}

@Test
public void givenTwoFractionInstances_whenCalledadd_thenCorrect() {
    Fraction fraction1 = Fraction.getFraction(1, 4);
    Fraction fraction2 = Fraction.getFraction(3, 4);
    assertThat(fraction1.add(fraction2).toString()).isEqualTo("1/1");
}

@Test
public void givenTwoFractionInstances_whenCalledsubstract_thenCorrect() {
    Fraction fraction1 = Fraction.getFraction(3, 4);
    Fraction fraction2 = Fraction.getFraction(1, 4);
    assertThat(fraction1.subtract(fraction2).toString()).isEqualTo("1/2");
}

@Test
public void givenTwoFractionInstances_whenCalledmultiply_thenCorrect() {
    Fraction fraction1 = Fraction.getFraction(3, 4);
    Fraction fraction2 = Fraction.getFraction(1, 4);
    assertThat(fraction1.multiplyBy(fraction2).toString()).isEqualTo("3/16");
}
```

虽然分数运算肯定不是我们在日常开发工作中必须处理的最常见的任务，但是`Fraction`类为以简单明了的方式执行这些运算提供了宝贵的支持。

## 7。`SystemUtils`班

有时，我们需要获得一些关于底层 Java 平台或操作系统的不同属性和变量的动态信息。

**Apache Commons Lang 3 提供了`[SystemUtils](https://web.archive.org/web/20221229040443/https://commons.apache.org/proper/commons-lang/apidocs/org/apache/commons/lang3/SystemUtils.html) class`以一种无痛的方式完成这个**。

例如，让我们考虑一下`getJavaHome()`、`getUserHome()`和`isJavaVersionAtLeast()`方法:

```java
@Test
public void whenCalledgetJavaHome_thenCorrect() {
    assertThat(SystemUtils.getJavaHome())
      .isEqualTo(new File("path/to/java/jdk"));
}

@Test
public void whenCalledgetUserHome_thenCorrect() {
    assertThat(SystemUtils.getUserHome())
      .isEqualTo(new File("path/to/user/home"));
}

@Test
public void whenCalledisJavaVersionAtLeast_thenCorrect() {
    assertThat(SystemUtils.isJavaVersionAtLeast(JavaVersion.JAVA_RECENT)).isTrue();
}
```

还有一些额外的实用方法由`SystemUtils`类实现。为了使例子简短，我们省略了它们。

## 8。惰性初始化和构建器类

Apache Commons Lang 3 最吸引人的方面之一是实现了一些众所周知的设计模式，包括[惰性初始化](https://web.archive.org/web/20221229040443/https://en.wikipedia.org/wiki/Lazy_initialization)和[构建器](https://web.archive.org/web/20221229040443/https://en.wikipedia.org/wiki/Builder_pattern)模式。

例如，假设我们已经创建了一个昂贵的`User`类(为了简洁起见，这里没有显示),并希望将其实例化推迟到真正需要的时候。

在这种情况下，我们需要做的就是扩展参数化的 [LazyInitializer](https://web.archive.org/web/20221229040443/https://commons.apache.org/proper/commons-lang/apidocs/index.html?org/apache/commons/lang3/concurrent/LazyInitializer.html) 抽象类并覆盖它的`initialize()`方法:

```java
public class UserInitializer extends LazyInitializer<User> {

    @Override
    protected User initialize() {
        return new User("John", "[[email protected]](/web/20221229040443/https://www.baeldung.com/cdn-cgi/l/email-protection)");
    }
}
```

现在，如果我们想在需要的时候获取代价高昂的`User`对象，我们只需调用`UserInitializer's get()`方法:

```java
@Test 
public void whenCalledget_thenCorrect() 
  throws ConcurrentException { 
    UserInitializer userInitializer = new UserInitializer(); 
    assertThat(userInitializer.get()).isInstanceOf(User.class); 
}
```

**`get()`方法是实例字段的双重检查习语(线程安全)的实现，如** [**约书亚·布洛赫的《有效的 Java》，第 71 项**](https://web.archive.org/web/20221229040443/https://www.pearson.com/us/higher-education/program/Bloch-Effective-Java-3rd-Edition/PGM1763855.html) 所述:

```java
private volatile User instance;

User get() { 
    if (instance == null) { 
        synchronized(this) { 
            if (instance == null) 
                instance = new User("John", "[[email protected]](/web/20221229040443/https://www.baeldung.com/cdn-cgi/l/email-protection)"); 
            }
        } 
    } 
    return instance; 
}
```

此外，Apache Commons Lang 3 实现了 [HashCodeBuilder](https://web.archive.org/web/20221229040443/https://commons.apache.org/proper/commons-lang/javadocs/api-3.1/org/apache/commons/lang3/builder/HashCodeBuilder.html) 类，它允许我们基于典型的 fluent API，通过向构建器提供不同的参数来生成`hashCode()`实现:

```java
@Test
public void whenCalledtoHashCode_thenCorrect() {
    int hashcode = new HashCodeBuilder(17, 37)
      .append("John")
      .append("[[email protected]](/web/20221229040443/https://www.baeldung.com/cdn-cgi/l/email-protection)")
      .toHashCode();
    assertThat(hashcode).isEqualTo(1269178828);
}
```

我们可以用 [`BasicThreadFactory`](https://web.archive.org/web/20221229040443/https://commons.apache.org/proper/commons-lang/javadocs/api-3.1/org/apache/commons/lang3/concurrent/BasicThreadFactory.html) 类做一些类似的事情，创建带有命名模式和优先级的守护线程:

```java
@Test
public void whenCalledBuilder_thenCorrect() {
    BasicThreadFactory factory = new BasicThreadFactory.Builder()
      .namingPattern("workerthread-%d")
      .daemon(true)
      .priority(Thread.MAX_PRIORITY)
      .build();
    assertThat(factory).isInstanceOf(BasicThreadFactory.class);
}
```

## 9。`ConstructorUtils`班

**倒影是阿帕奇公地 Lang 3 的一等公民。**

该库包括几个反射类，允许我们反射性地访问和操作类字段和方法。

例如，假设我们已经实现了一个简单的`User`域类:

```java
public class User {

    private String name;
    private String email;

    // standard constructors / getters / setters / toString
}
```

假设它的参数化构造函数是`public`，我们可以很容易地用`[ConstructorUtils](https://web.archive.org/web/20221229040443/https://commons.apache.org/proper/commons-lang/apidocs/org/apache/commons/lang3/reflect/ConstructorUtils.html)`类访问它:

```java
@Test
public void whenCalledgetAccessibleConstructor_thenCorrect() {
    assertThat(ConstructorUtils
      .getAccessibleConstructor(User.class, String.class, String.class))
      .isInstanceOf(Constructor.class);
} 
```

除了通过构造函数进行标准的类实例化，我们还可以通过调用`invokeConstructor()`和`invokeExactConstructor()`方法来反射性地创建`User`实例:

```java
@Test
public void whenCalledinvokeConstructor_thenCorrect() 
  throws Exception {
      assertThat(ConstructorUtils.invokeConstructor(User.class, "name", "email"))
        .isInstanceOf(User.class);
}

@Test
public void whenCalledinvokeExactConstructor_thenCorrect() 
  throws Exception {
      String[] args = {"name", "email"};
      Class[] parameterTypes= {String.class, String.class};
      assertThat(ConstructorUtils.invokeExactConstructor(User.class, args, parameterTypes))
        .isInstanceOf(User.class);
} 
```

## 10。`FieldUtils`班

类似地，**我们可以使用 [`FieldUtils`](https://web.archive.org/web/20221229040443/https://commons.apache.org/proper/commons-lang/apidocs/org/apache/commons/lang3/reflect/FieldUtils.html) 类的方法来反射性地读/写类字段**。

让我们假设我们想要得到一个`User` 类的字段，或者最终得到一个该类从超类继承的字段。

在这种情况下，我们可以调用`getField()`方法:

```java
@Test
public void whenCalledgetField_thenCorrect() {
    assertThat(FieldUtils.getField(User.class, "name", true).getName())
      .isEqualTo("name");
} 
```

或者，**如果我们想要使用更严格的反射范围，并且只获取在`User` 类中声明的字段，而不是从超类**中继承的字段，我们只需使用`getDeclaredField()`方法:

```java
@Test
public void whenCalledgetDeclaredFieldForceAccess_thenCorrect() {
    assertThat(FieldUtils.getDeclaredField(User.class, "name", true).getName())
      .isEqualTo("name");
}
```

此外，我们可以使用`getAllFields()`方法来获取反射类的字段数量，并使用 `writeField()`和`writeDeclaredField()`方法向声明的字段或层次结构中定义的字段写入一个值:

```java
@Test
public void whenCalledgetAllFields_thenCorrect() {
    assertThat(FieldUtils.getAllFields(User.class).length)
      .isEqualTo(2);  
}

@Test
public void whenCalledwriteField_thenCorrect() 
  throws IllegalAccessException {
    FieldUtils.writeField(user, "name", "Julie", true);
    assertThat(FieldUtils.readField(user, "name", true))
      .isEqualTo("Julie");     
}

@Test
public void givenFieldUtilsClass_whenCalledwriteDeclaredField_thenCorrect() throws IllegalAccessException {
    FieldUtils.writeDeclaredField(user, "name", "Julie", true);
    assertThat(FieldUtils.readField(user, "name", true))
      .isEqualTo("Julie");    
}
```

## 11。`MethodUtils`班

同样，我们可以在`[MethodUtils](https://web.archive.org/web/20221229040443/https://commons.apache.org/proper/commons-lang/apidocs/org/apache/commons/lang3/reflect/MethodUtils.html)`类中使用类方法的反射。

在这种情况下，`User`类的`getName()`方法的可见性是`public`。因此，我们可以用`getAccessibleMethod()`方法访问它:

```java
@Test
public void whenCalledgetAccessibleMethod_thenCorrect() {
    assertThat(MethodUtils.getAccessibleMethod(User.class, "getName"))
      .isInstanceOf(Method.class);
} 
```

说到反射调用方法，我们可以使用`invokeExactMethod()`和`invokeMethod()`方法:

```java
@Test
public 
  void whenCalledinvokeExactMethod_thenCorrect() 
  throws Exception {
    assertThat(MethodUtils.invokeExactMethod(new User("John", "[[email protected]](/web/20221229040443/https://www.baeldung.com/cdn-cgi/l/email-protection)"), "getName"))
     .isEqualTo("John");
}

@Test
public void whenCalledinvokeMethod_thenCorrect() 
  throws Exception {
    User user = new User("John", "[[email protected]](/web/20221229040443/https://www.baeldung.com/cdn-cgi/l/email-protection)");
    Object method = MethodUtils.invokeMethod(user, true, "setName", "John");
    assertThat(user.getName()).isEqualTo("John");
}
```

## 12。`MutableObject`班

虽然[不变性](https://web.archive.org/web/20221229040443/https://en.wikipedia.org/wiki/Immutable_object)是好的面向对象软件的一个关键特征，我们应该在每种可能的情况下默认使用，但不幸的是，有时我们需要处理易变的对象。

此外，创建可变类需要大量样板代码，这些代码可以由大多数 ide 通过自动生成的 setters 生成。

为此，Apache Commons Lang 3 提供了`[MutableObject](https://web.archive.org/web/20221229040443/https://commons.apache.org/proper/commons-lang/javadocs/api-3.1/org/apache/commons/lang3/mutable/MutableObject.html)`类，这是一个简单的包装类，用于以最小的代价创建可变对象:

```java
@BeforeClass
public static void setUpMutableObject() {
    mutableObject = new MutableObject("Initial value");
}

@Test
public void whenCalledgetValue_thenCorrect() {
    assertThat(mutableObject.getValue()).isInstanceOf(String.class);
}

@Test
public void whenCalledsetValue_thenCorrect() {
    mutableObject.setValue("Another value");
    assertThat(mutableObject.getValue()).isEqualTo("Another value");
}

@Test
public void whenCalledtoString_thenCorrect() {
    assertThat(mutableObject.toString()).isEqualTo("Another value");    
} 
```

当然，这只是一个如何使用`MutableObject`类的例子。

作为经验法则，**我们应该总是努力创建不可变的类，或者在最坏的情况下，只提供所需级别的可变性**。

## 13。`MutablePair`班

有趣的是，Apache Commons Lang 3 以对和三元组的形式为元组提供了强大的支持。

因此，让我们假设我们需要创建一个可变的有序元素对。

在这种情况下，我们将使用`[MutablePair](https://web.archive.org/web/20221229040443/https://commons.apache.org/proper/commons-lang/apidocs/org/apache/commons/lang3/tuple/MutablePair.html)`类:

```java
private static MutablePair<String, String> mutablePair;

@BeforeClass
public static void setUpMutablePairInstance() {
    mutablePair = new MutablePair<>("leftElement", "rightElement");
}

@Test
public void whenCalledgetLeft_thenCorrect() {
    assertThat(mutablePair.getLeft()).isEqualTo("leftElement");
}

@Test
public void whenCalledgetRight_thenCorrect() {
    assertThat(mutablePair.getRight()).isEqualTo("rightElement");
}

@Test
public void whenCalledsetLeft_thenCorrect() {
    mutablePair.setLeft("newLeftElement");
    assertThat(mutablePair.getLeft()).isEqualTo("newLeftElement");
} 
```

这里最值得强调的相关细节是该类的 clean API。

它允许我们通过标准的 setter/getter 来设置和访问由该对包装的左右对象。

## 14。`ImmutablePair`班

毫不奇怪，`MutablePair`类也有一个不可变的对应实现，叫做`[ImmutablePair](https://web.archive.org/web/20221229040443/https://commons.apache.org/proper/commons-lang/javadocs/api-3.1/org/apache/commons/lang3/tuple/ImmutablePair.html)`:

```java
private static ImmutablePair<String, String> immutablePair = new ImmutablePair<>("leftElement", "rightElement");

@Test
public void whenCalledgetLeft_thenCorrect() {
    assertThat(immutablePair.getLeft()).isEqualTo("leftElement");
}

@Test
public void whenCalledgetRight_thenCorrect() {
    assertThat(immutablePair.getRight()).isEqualTo("rightElement");
}

@Test
public void whenCalledof_thenCorrect() {
    assertThat(ImmutablePair.of("leftElement", "rightElement"))
      .isInstanceOf(ImmutablePair.class);
}

@Test(expected = UnsupportedOperationException.class)
public void whenCalledSetValue_thenThrowUnsupportedOperationException() {
    immutablePair.setValue("newValue");
} 
```

正如我们对一个不可变类的期望，任何通过`setValue()`方法改变 pair 内部状态的尝试都会导致抛出一个`UnsupportedOperationException`异常。

## 15.**`Triple`班**

这里要看的最后一个实用程序类是`[Triple](https://web.archive.org/web/20221229040443/https://commons.apache.org/proper/commons-lang/apidocs/org/apache/commons/lang3/tuple/Triple.html)`。

由于类是抽象的，我们可以通过使用 [`of()`](https://web.archive.org/web/20221229040443/https://commons.apache.org/proper/commons-lang/apidocs/org/apache/commons/lang3/tuple/Triple.html#of-L-M-R-) 静态工厂方法来创建`Triple`实例:

```java
@BeforeClass
public static void setUpTripleInstance() {
    triple = Triple.of("leftElement", "middleElement", "rightElement");
}

@Test
public void whenCalledgetLeft_thenCorrect() {
    assertThat(triple.getLeft()).isEqualTo("leftElement");
}

@Test
public void whenCalledgetMiddle_thenCorrect() {
    assertThat(triple.getMiddle()).isEqualTo("middleElement");
}

@Test
public void whenCalledgetRight_thenCorrect() {
    assertThat(triple.getRight()).isEqualTo("rightElement");
}
```

通过`[MutableTriple](https://web.archive.org/web/20221229040443/https://commons.apache.org/proper/commons-lang/apidocs/org/apache/commons/lang3/tuple/ImmutableTriple.html)`和 [`ImmutableTriple`](https://web.archive.org/web/20221229040443/https://commons.apache.org/proper/commons-lang/apidocs/org/apache/commons/lang3/tuple/ImmutableTriple.html) 类，也有可变和不可变三元组的具体实现。

我们可以通过参数化的构造函数来创建它们的实例，而不是使用静态的工厂方法。

在这种情况下，我们将跳过它们，因为它们的 API 看起来与`MutablePair`和`ImmutablePair`类非常相似。

## 16。结论

在本教程中，我们深入了解了 Apache Commons Lang 3 提供的一些最有用的实用程序类**。**

 **该库实现了许多其他值得一看的实用程序类**。**在这里，我们只是展示了最有用的，基于一个非常固执己见的标准。

完整的库 API 请查看[官方 Javadocs](https://web.archive.org/web/20221229040443/https://commons.apache.org/proper/commons-lang/javadocs/api-release/index.html) 。

像往常一样，本教程中显示的所有代码示例都可以在 [GitHub](https://web.archive.org/web/20221229040443/https://github.com/eugenp/tutorials/tree/master/libraries-apache-commons) 上获得。**