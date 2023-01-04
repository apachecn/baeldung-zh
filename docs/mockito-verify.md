# 莫奇托验证食谱

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/mockito-verify>

## 1。概述

这本食谱说明了**如何在各种用例中使用 Mockito verify** 。

食谱的**格式是以例子为中心的**并且实用——不需要多余的细节和解释。

我们将**模仿一个简单的 list** 实现:

```java
public class MyList extends AbstractList<String> {

    @Override
    public String get(final int index) {
        return null;
    }
    @Override
    public int size() {
        return 0;
    }
} 
```

## 延伸阅读:

## [使用 Mockito 模拟异常抛出](/web/20221123000650/https://www.baeldung.com/mockito-exceptions)

Learn to configure a method call to throw an exception in Mockito.[Read more](/web/20221123000650/https://www.baeldung.com/mockito-exceptions) →

## [Mockito 的 Java 8 特性](/web/20221123000650/https://www.baeldung.com/mockito-java-8)

Overview of Java 8 support in Mockito framework, including Streams and default interface methods[Read more](/web/20221123000650/https://www.baeldung.com/mockito-java-8) →

## [使用 PowerMock 模仿私有方法](/web/20221123000650/https://www.baeldung.com/powermock-private-method)

Learn how PowerMock can be used to extend the capability of Mockito for mocking and verification of private methods in the class under test.[Read more](/web/20221123000650/https://www.baeldung.com/powermock-private-method) →

## 2。烹饪书

**验证模拟上的简单调用:**

```java
List<String> mockedList = mock(MyList.class);
mockedList.size();
verify(mockedList).size();
```

**验证与模拟的交互次数:**

```java
List<String> mockedList = mock(MyList.class);
mockedList.size();
verify(mockedList, times(1)).size();
```

**验证没有与整个模拟发生交互:**

```java
List<String> mockedList = mock(MyList.class);
verifyNoInteractions(mockedList);
```

**验证没有与特定方法发生交互:**

```java
List<String> mockedList = mock(MyList.class);
verify(mockedList, times(0)).size();
```

**验证没有意外的交互—这应该会失败:**

```java
List<String> mockedList = mock(MyList.class);
mockedList.size();
mockedList.clear();
verify(mockedList).size();
verifyNoMoreInteractions(mockedList);
```

**验证交互顺序:**

```java
List<String> mockedList = mock(MyList.class);
mockedList.size();
mockedList.add("a parameter");
mockedList.clear();

InOrder inOrder = Mockito.inOrder(mockedList);
inOrder.verify(mockedList).size();
inOrder.verify(mockedList).add("a parameter");
inOrder.verify(mockedList).clear();
```

**验证没有发生交互:**

```java
List<String> mockedList = mock(MyList.class);
mockedList.size();
verify(mockedList, never()).clear();
```

**验证交互至少发生了一定次数:**

```java
List<String> mockedList = mock(MyList.class);
mockedList.clear();
mockedList.clear();
mockedList.clear();

verify(mockedList, atLeast(1)).clear();
verify(mockedList, atMost(10)).clear();
```

**用确切的参数验证交互:**

```java
List<String> mockedList = mock(MyList.class);
mockedList.add("test");
verify(mockedList).add("test");
```

**验证与灵活/任意参数的交互:**

```java
List<String> mockedList = mock(MyList.class);
mockedList.add("test");
verify(mockedList).add(anyString());
```

**使用参数捕获验证交互:**

```java
List<String> mockedList = mock(MyList.class);
mockedList.addAll(Lists.<String> newArrayList("someElement"));

ArgumentCaptor<List<String>> argumentCaptor = ArgumentCaptor.forClass(List.class);
verify(mockedList).addAll(argumentCaptor.capture());

List<String> capturedArgument = argumentCaptor.getValue();
assertThat(capturedArgument).contains("someElement");
```

## 3。结论

本指南的目标是让这些信息在网上随时可用。我已经出版了几本类似的关于[谷歌番石榴](/web/20221123000650/https://www.baeldung.com/guava-collections "Guava Collections Cookbook")和[汉堡](/web/20221123000650/https://www.baeldung.com/hamcrest-collections-arrays "My Hamcrest Collections cookbook")的开发食谱。

所有这些例子和代码片段的实现都可以在 GitHub 上找到[。这是一个基于 Maven 的项目，因此应该很容易导入和运行。](https://web.archive.org/web/20221123000650/https://github.com/eugenp/tutorials/tree/master/testing-modules/mockito-simple "Github Project exemplifying Mockito.verify")