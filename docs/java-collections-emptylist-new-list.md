# Collections.emptyList()与新列表实例

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-collections-emptylist-new-list>

## 1.介绍

在这个简短的教程中，我们将说明`Collections.emptyList()`和新列表实例之间的区别。

## 2.不变

`java.util.Collections.emptyList()`和新列表(如`new ArrayList<>()`)的核心区别是不变性。

`Collections.emptyList()`返回一个不可修改的列表(`java.util.Collections.EmptyList`)。

创建新列表实例时，您可以根据实施情况对其进行修改:

```java
@Test
public void givenArrayList_whenAddingElement_addsNewElement() {	 	 
    List<String> mutableList = new ArrayList<>();	 	 
    mutableList.add("test");	 	 

    assertEquals(mutableList.size(), 1);	 	 
    assertEquals(mutableList.get(0), "test");	 	 
}

@Test(expected = UnsupportedOperationException.class)	 	 
public void givenCollectionsEmptyList_whenAdding_throwsException() {	 	 
    List<String> immutableList = Collections.emptyList();	 	 
    immutableList.add("test");	 	 
}
```

## 3.对象创建

**`Collection.emptyList()`只创建一次新的空列表实例**，如源代码所示:

```java
public static final List EMPTY_LIST = new EmptyList<>();

public static final <T> List<T> emptyList() {
    return (List<T>) EMPTY_LIST;
}
```

## 4.可读性

当你想显式地创建一个空列表时，那么`Collections.emptyList()`更好地表达了原意，例如 `new ArrayList<>()`。

## 5.结论

在这篇中肯的文章中，我们重点关注了`Collections.emptyList()`和一个新的 list 实例之间的区别。

与往常一样，GitHub 上有完整的源代码[。](https://web.archive.org/web/20220801225348/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-list-3)