# Java 泛型–> vs 扩展对象>

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-generics-vs-extends-object>

## 1.概观

在这个快速教程中，我们将看到 [Java 泛型](/web/20220628150041/https://www.baeldung.com/java-generics) 中`<?>`和`<? extends Object>`的**异同。**

然而，这是一个高级话题，在我们深入问题的关键之前，有必要对这个主题有一个基本的了解。

## 2.仿制药的背景

泛型是在 JDK 5 中引入的，以消除运行时错误并加强类型安全性。这种额外的类型安全消除了某些用例中的强制转换，并使程序员能够编写泛型算法，这两者都可以产生更可读的代码。

例如，在 JDK 5 之前，我们必须使用强制转换来处理列表元素。这反过来又产生了某种类型的运行时错误:

```java
List aList = new ArrayList();
aList.add(new Integer(1));
aList.add("a_string");

for (int i = 0; i < aList.size(); i++) {
    Integer x = (Integer) aList.get(i);
}
```

现在，这段代码有两个我们想要解决的问题:

*   我们需要一个显式的强制转换来从`aList`中提取值——该类型取决于左边的变量类型——在本例中为`Integer`
*   当我们试图将`a_string`转换为`Integer`时，我们将在第二次迭代中得到一个运行时错误

泛型为我们填补了这个角色:

```java
List<Integer> iList = new ArrayList<>();
iList.add(1);
iList.add("a_string"); // compile time error

for (int i = 0; i < iList.size(); i++) {
    int x = iList.get(i);
} 
```

编译器会告诉我们不可能将`a_string`添加到`Integer`类型的`List`中，这比在运行时发现要好。

此外，不需要显式强制转换，因为编译器已经知道`iList`保存`Integer` s。此外，由于取消装箱的魔力，我们甚至不需要`Integer` 类型，它的原始形式就足够了。

## 3.泛型中的通配符

问号或通配符在泛型中用于表示未知类型。它可以有三种形式:

*   `Unbounded Wildcards` : `List<?>`表示未知类型的列表
*   `Upper Bounded Wildcards` : `List<? extends Number>`表示`Number`或其子类型的列表，如`Integer`、`Double`
*   `Lower Bounded Wildcards` : `List<? super Integer>`表示`Integer`或其超类型`Number`和`Object`的列表

现在，由于`Object`是 Java 中所有类型的固有超类型，我们可能会认为它也可以表示未知类型。换句话说，`List<?>`和`List<Object>`可以达到同样的目的。但是他们没有。

让我们考虑这两种方法:

```java
public static void printListObject(List<Object> list) {    
    for (Object element : list) {        
        System.out.print(element + " ");    
    }        
}    

public static void printListWildCard(List<?> list) {    
    for (Object element: list) {        
        System.out.print(element + " ");    
    }     
} 
```

给出一个`Integer`列表，比如说:

```java
List<Integer> li = Arrays.asList(1, 2, 3);
```

`printListObject(li)`不会编译，我们会得到这个错误:

```java
The method printListObject(List<Object>) is not applicable for the arguments (List<Integer>)
```

而`printListWildCard(li)`将编译并将`1 2 3`输出到控制台。

## 4.`<?>` 和 `<? extends Object>`——相似之处

在上面的例子中，如果我们将`printListWildCard`的方法签名改为:

```java
public static void printListWildCard(List<? extends Object> list)
```

它的功能和`printListWildCard(List<?> list)`一样。这是因为`Object`是所有 Java 对象的超类型，基本上所有东西都扩展了`Object`。因此，`Integer`的一个`List`也得到了处理。

简而言之，**意味着`?`和`? extends Object`在这个例子**中是同义的。

虽然在大多数情况下这是正确的，**但是也有一些不同之处**。让我们在下一节看看它们。

## 5.`<?>` 和 `<? extends Object>`–区别

可验证的类型是那些在编译时不被删除的类型。换句话说，一个非可具体化类型的运行时表示比它的编译时对应的信息要少，因为它的一些会被删除。

一般来说，参数化类型是不可具体化的。这意味着`List<String>`和`Map<Integer, String>`是不可具体化的。编译器会删除它们的类型，并将它们分别视为`List`和`Map`。

这个规则的唯一例外是未绑定的通配符类型。**这意味着`List<?>`和`Map<?,?>`是可具体化的**。

另一方面， **`List<? extends Object>`是不可具体化的**。虽然很微妙，但这是一个显著的区别。

不可实现类型不能在某些情况下使用，比如在 [`instanceof`](/web/20220628150041/https://www.baeldung.com/java-instanceof) 操作符中或者作为数组的元素。

所以，如果我们写:

```java
List someList = new ArrayList<>();
boolean instanceTest = someList instanceof List<?>
```

这段代码编译后`instanceTest`就是`true`。

但是，如果我们对`List<? extends Object>`使用`instanceof`操作符:

```java
List anotherList = new ArrayList<>();
boolean instanceTest = anotherList instanceof List<? extends Object>;
```

那么第 2 行不会编译。

类似地，在下面的代码片段中，第 1 行编译了，但是第 2 行没有:

```java
List<?>[] arrayOfList = new List<?>[1];
List<? extends Object>[] arrayOfAnotherList = new List<? extends Object>[1]
```

## 6.结论

在这个简短的教程中，我们看到了`<?>`和`<? extends Object>`的异同。

虽然两者非常相似，但在可具体化或不可具体化方面还是有细微的差别。