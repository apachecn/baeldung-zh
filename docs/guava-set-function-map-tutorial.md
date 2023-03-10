# 番石榴套装+功能=地图

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/guava-set-function-map-tutorial>

## 1。概述

在本教程中——我们将举例说明**番石榴**的`collect`包中许多有用的特性之一:**如何将一个函数应用于一个番石榴`Set`并获得一个`Map`T6。**

我们将讨论两种方法——基于内置的 guava 操作创建一个不可变地图和一个动态地图，然后实现一个自定义的动态`Map`实现。

## 2。设置

首先，我们将添加**番石榴**库作为`pom.xml:`中的依赖项

```java
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>31.0.1-jre</version>
</dependency>
```

一个小提示——你可以在这里检查是否有一个[新版本。](https://web.archive.org/web/20220122051352/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22com.google.guava%22%20AND%20a%3A%22guava%22)

## 3。`Function`映射

让我们首先定义将应用于集合元素的函数:

```java
 Function<Integer, String> function = new Function<Integer, String>() {
        @Override
        public String apply(Integer from) {
            return Integer.toBinaryString(from.intValue());
        }
    };
```

该函数只是将一个`Integer`的值转换成它的二进制`String`表示。

## 4。`toMap()`番石榴

Guava 提供了一个关于`Map`实例的静态实用程序类。其中，它有两个操作，通过应用已定义的番石榴树的`Function`，可以将一个`Set`转换为一个`Map`。

下面的代码片段展示了如何创建一个不可变的`Map`:

```java
Map<Integer, String> immutableMap = Maps.toMap(set, function);
```

以下测试断言集合被正确转换:

```java
@Test
public void givenStringSetAndSimpleMap_whenMapsToElementLength_thenCorrect() {
    Set set = new TreeSet(Arrays.asList(32, 64, 128));
    Map<Integer, String> immutableMap = Maps.toMap(set, function);
    assertTrue(immutableMap.get(32).equals("100000")
      && immutableMap.get(64).equals("1000000")
      && immutableMap.get(128).equals("10000000"));
} 
```

创建的映射的问题是，如果将一个元素添加到源集中，那么派生的映射就不会更新。

## 4。`asMap()`番石榴

如果我们使用前面的例子并使用`Maps.asMap`方法创建地图:

```java
Map<Integer, String> liveMap = Maps.asMap(set, function);
```

我们将得到**一个实时地图视图**，这意味着对原始设置的更改也将反映在地图上:

```java
@Test
public void givenStringSet_whenMapsToElementLength_thenCorrect() {
    Set<Integer> set = new TreeSet<Integer>(Arrays.asList(32, 64, 128));
    Map<Integer, String> liveMap = Maps.asMap(set, function);
    assertTrue(liveMap.get(32).equals("100000")
            && liveMap.get(64).equals("1000000")
            && liveMap.get(128).equals("10000000"));

    set.add(256);
    assertTrue(liveMap.get(256).equals("100000000") && liveMap.size() == 4);
} 
```

注意，尽管我们通过一个集合添加了一个元素，并在 map 中查找它，但是测试仍然正确地断言。

## 5。`Map` 建筑定制现场

当我们谈论一个`Set`的`Map` 视图时，我们基本上是用一个**番石榴** `Function`来扩展`Set`的能力。

在 live `Map` 视图中，对`Set`的更改应该会实时更新`Map`和`EntrySet`。我们将创建自己的泛型`Map`，子类 `AbstractMap<K,V` `>`，就像这样:

```java
public class GuavaMapFromSet<K, V> extends AbstractMap<K, V> {
    public GuavaMapFromSet(Set<K> keys, 
        Function<? super K, ? extends V> function) { 
    }
}
```

值得注意的是，`AbstractMap`的所有子类的主契约都实现了`entrySet`方法，就像我们已经做的那样。然后，我们将在下面的小节中查看代码的两个关键部分。

### 5.1。条目

我们的`Map`中的另一个属性是`entries`，代表我们的`EntrySet:`

```java
private Set<Entry<K, V>> entries;
```

使用来自**构造函数**的输入`Set` 将始终初始化`entries`字段

```java
public GuavaMapFromSet(Set<K> keys,Function<? super K, ? extends V> function) {
    this.entries=keys;
}
```

这里有一个简单的注意事项——为了保持一个实时视图，我们将在输入`Set`中使用相同的`iterator`用于随后的`Map`的`EntrySet.`

在履行`AbstractMap<K,V` `>`的契约时，我们实现了`entrySet`方法，然后返回`entries`:

```java
@Override
public Set<java.util.Map.Entry<K, V>> entrySet() {
    return this.entries;
}
```

### 5.2。缓存

该`Map`存储通过**将`Function`应用于`Set:`** 获得的值

```java
private WeakHashMap<K, V> cache;
```

## 6。`Set Iterator`

我们将把输入`Set`的`iterator`用于随后的`Map`的`EntrySet`。为此，我们使用了一个定制的`EntrySet` 和一个定制的`Entry`类。

### 6.1。`Entry`班

首先，让我们看看`Map`中的单个条目看起来会是什么样子:

```java
private class SingleEntry implements Entry<K, V> {
    private K key;
    public SingleEntry( K key) {
        this.key = key;
    }
    @Override
    public K getKey() {
        return this.key;
    }
    @Override
    public V getValue() {
        V value = GuavaMapFromSet.this.cache.get(this.key);
  if (value == null) {
      value = GuavaMapFromSet.this.function.apply(this.key);
      GuavaMapFromSet.this.cache.put(this.key, value);
  }
  return value;
    }
    @Override
    public V setValue( V value) {
        throw new UnsupportedOperationException();
    }
}
```

显然，在这段代码中，我们不允许从`Map` 视图 `as`中修改`Set`，对`setValue`的调用会抛出`UnsupportedOperationException.`

密切关注`getValue`——这是我们的**实时视图**功能的关键。我们在我们的`Map`中检查当前的`key` ( `Set`元素)的`cache`。

如果我们找到了密钥，我们就返回它，否则**我们将我们的函数应用于当前密钥并获得一个值**，然后将它存储在`cache`中。

这样，每当`Set` 有一个新元素时，地图是最新的，因为新值是动态计算的。

### 6.2。`EntrySet`

我们现在将实现`EntrySet`:

```java
private class MyEntrySet extends AbstractSet<Entry<K, V>> {
    private Set<K> keys;
    public MyEntrySet(Set<K> keys) {
        this.keys = keys;
    }
    @Override
    public Iterator<Map.Entry<K, V>> iterator() {
        return new LiveViewIterator();
    }
    @Override
    public int size() {
        return this.keys.size();
    }
}
```

通过重写`iterator`和`size`方法，我们已经完成了扩展`AbstractSet`的约定。但是还有更多。

记住这个`EntrySet`的实例将在我们的`Map` 视图中形成`entries`。通常，map 的`EntrySet`只是为每次迭代返回一个完整的`Entry`。

然而，在我们的例子中，我们需要使用来自输入`Set` 的`iterator`来维护我们的实时视图*。*我们知道它只会返回`Set`的元素，所以我们还需要一个自定义`iterator`。

### 6.3。`Iterator`

下面是我们的`iterator`对上面`EntrySet`的实现:

```java
public class LiveViewIterator implements Iterator<Entry<K, V>> {
    private Iterator<K> inner;

    public LiveViewIterator () {
        this.inner = MyEntrySet.this.keys.iterator();
    }

    @Override
    public boolean hasNext() {
        return this.inner.hasNext();
    }
    @Override
    public Map.Entry<K, V> next() {
        K key = this.inner.next();
        return new SingleEntry(key);
    }
    @Override
    public void remove() {
        throw new UnsupportedOperationException();
    }
}
```

`LiveViewIterator` 必须驻留在`MyEntrySet`类中，这样，我们可以在初始化时共享`Set`的`iterator`。

当使用`iterator`遍历`GuavaMapFromSet`的条目时，对`next`的调用只是从`Set`的`iterator`中检索密钥并构造一个`SingleEntry`。

## 7 .**。将所有这些放在一起**

在将我们在本教程中所涉及的内容拼接在一起之后，让我们用之前的示例中的`liveMap`变量来替换它，并用我们的自定义地图来替换它:

```java
@Test
public void givenIntSet_whenMapsToElementBinaryValue_thenCorrect() {
    Set<Integer> set = new TreeSet<>(Arrays.asList(32, 64, 128));
    Map<Integer, String> customMap = new GuavaMapFromSet<Integer, String>(set, function);

    assertTrue(customMap.get(32).equals("100000")
      && customMap.get(64).equals("1000000")
      && customMap.get(128).equals("10000000"));
}
```

改变输入`Set`的内容，我们将看到`Map` 实时更新:

```java
@Test
public void givenStringSet_whenMapsToElementLength_thenCorrect() {
    Set<Integer> set = new TreeSet<Integer>(Arrays.asList(32, 64, 128));
    Map<Integer, String> customMap = Maps.asMap(set, function);

    assertTrue(customMap.get(32).equals("100000")
      && customMap.get(64).equals("1000000")
      && customMap.get(128).equals("10000000"));

    set.add(256);
    assertTrue(customMap.get(256).equals("100000000") && customMap.size() == 4);
}
```

## 8。结论

在本教程中，我们已经看到了利用**番石榴**操作和**通过应用`Function`** 从`Set`获得`Map`视图的不同方式。

所有这些例子和代码片段的完整实现**可以在[我的番石榴 github 项目](https://web.archive.org/web/20220122051352/https://github.com/eugenp/tutorials/tree/master/guava-modules/guava-collections-set "The Github Project with the impl of all examples using Guava Collections")** 中找到——这是一个基于 Eclipse 的项目，所以它应该很容易导入和运行。