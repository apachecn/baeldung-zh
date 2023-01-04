# Java 中的平衡括号算法

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-balanced-brackets-algorithm>

## 1.概观

平衡括号，也称为平衡括号，是一个常见的编程问题。

在本教程中，我们将验证给定字符串中的括号是否平衡。

这种类型的字符串是所谓的 [Dyck 语言](https://web.archive.org/web/20221127043744/https://en.wikipedia.org/wiki/Dyck_language)的一部分。

## 2.问题陈述

中括号被认为是以下任何字符-"("，")"，"["，"]"，" { "，" } "。

如果左括号、“(”、“[”、“{”、**分别出现在相应的右括号**、“”、“]”和“}”的左边，则一组括号**被认为是匹配对。**

然而，如果包含括号对的字符串不匹配，则该字符串**不平衡。**

类似地，**包含像 a-z、A-Z、0-9 这样的无括号字符**或者像#、$、@ **这样的其他特殊字符的字符串也被认为是不平衡的**。

例如，如果输入为“{[(])}”，则一对方括号“[]”会括起一个不平衡的左圆括号“(”。同样，这对圆括号“()”括起一个不对称的右方括号“]”。因此，输入字符串“{[(])}”是不平衡的。

因此，包含括号字符的字符串被认为是平衡的，如果:

1.  对应的左括号出现在每个对应的右括号的左边
2.  括在平衡括号内的括号也是平衡的
3.  它不包含任何非括号字符

有几个特殊情况需要记住: **`null`被认为是不平衡的，而空字符串被认为是平衡的**。

为了进一步说明我们对平衡括号的定义，让我们看一些平衡括号的例子:

```java
()
[()]
{[()]}
([{{[(())]}}])
```

还有一些不平衡的:

```java
abc[](){}
{{[]()}}}}
{[(])}
```

现在我们对我们的问题有了更好的了解，让我们看看如何解决它！

## 3.解决方法

解决这个问题有不同的方法。在本教程中，我们将研究两种方法:

1.  使用`String`类的方法
2.  使用`Deque`实现

## 4.基本设置和验证

让我们首先创建一个方法，如果输入平衡，它将返回`true`,如果输入不平衡，它将返回`false`:

```java
public boolean isBalanced(String str)
```

让我们考虑输入字符串的基本验证:

1.  如果一个`null`输入被传递，那么它是不平衡的。
2.  对于要平衡的字符串，成对的左括号和右括号应该匹配。因此，可以肯定地说，长度为奇数的输入字符串将是不平衡的，因为它至少包含一个不匹配的括号。
3.  根据问题陈述，应在括号内检查平衡行为。因此，任何包含非括号字符的输入字符串都是不平衡的字符串。

给定这些规则，我们可以实现验证:

```java
if (null == str || ((str.length() % 2) != 0)) {
    return false;
} else {
    char[] ch = str.toCharArray();
    for (char c : ch) {
        if (!(c == '{' || c == '[' || c == '(' || c == '}' || c == ']' || c == ')')) {
            return false;
        }
    }
}
```

既然输入字符串已经过验证，我们可以继续解决这个问题。

## 5.使用`String.replaceAll`方法

在这种方法中，我们将遍历输入字符串，使用`[String.replaceAll](/web/20221127043744/https://www.baeldung.com/java-remove-replace-string-part#string-api).`从字符串中删除“()”、“[]”和“{}”的出现。我们继续这一过程，直到在输入字符串中没有发现更多的出现。

一旦这个过程完成，如果字符串的长度为零，那么所有匹配的括号对都被删除，输入字符串就平衡了。但是，如果长度不为零，则字符串中仍存在一些不匹配的左括号或右括号。因此，输入字符串是不平衡的。

让我们看看完整的实现:

```java
while (str.contains("()") || str.contains("[]") || str.contains("{}")) {
    str = str.replaceAll("\\(\\)", "")
      .replaceAll("\\[\\]", "")
      .replaceAll("\\{\\}", "");
}
return (str.length() == 0);
```

## 6.使用`Deque`

[`Deque`](/web/20221127043744/https://www.baeldung.com/java-queue#3-deques) 是`Queue`的一种形式，在队列两端提供添加、检索和查看操作。我们将利用这个数据结构的[后进先出(LIFO)](/web/20221127043744/https://www.baeldung.com/java-lifo-thread-safe) 顺序特性来检查输入字符串中的余额。

首先，让我们构造我们的`Deque`:

```java
Deque<Character> deque = new LinkedList<>();
```

注意，我们在这里使用了一个 [`LinkedList`](/web/20221127043744/https://www.baeldung.com/java-linkedlist) ，因为它为`Deque`接口提供了一个实现。

现在我们的`deque`已经构造好了，我们将逐个循环输入字符串的每个字符。如果这个字符是一个左括号，那么我们将把它作为第一个元素添加到`Deque`:

```java
if (ch == '{' || ch == '[' || ch == '(') { 
    deque.addFirst(ch); 
}
```

但是，如果这个字符是一个右括号，那么我们将对`LinkedList`进行一些检查。

首先，我们检查`LinkedList`是否为空。空的`list`意味着右括号不匹配。因此，输入字符串是不平衡的。所以我们返回`false`。

然而，如果`LinkedList`不为空，那么我们使用`peekFirst`方法查看它的最后一个字符。如果它可以与右括号配对，那么我们使用`removeFirst`方法从`list`中移除这个最顶端的字符，并继续循环的下一次迭代:

```java
if (!deque.isEmpty() 
    && ((deque.peekFirst() == '{' && ch == '}') 
    || (deque.peekFirst() == '[' && ch == ']') 
    || (deque.peekFirst() == '(' && ch == ')'))) { 
    deque.removeFirst(); 
} else { 
    return false; 
}
```

在循环结束时，所有的字符都经过了平衡检查，所以我们可以返回`true`。下面是基于`Deque`的方法的完整实现:

```java
Deque<Character> deque = new LinkedList<>();
for (char ch: str.toCharArray()) {
    if (ch == '{' || ch == '[' || ch == '(') {
        deque.addFirst(ch);
    } else {
        if (!deque.isEmpty()
            && ((deque.peekFirst() == '{' && ch == '}')
            || (deque.peekFirst() == '[' && ch == ']')
            || (deque.peekFirst() == '(' && ch == ')'))) {
            deque.removeFirst();
        } else {
            return false;
        }
    }
}
return deque.isEmpty();
```

## 7.结论

在本教程中，我们讨论了平衡括号的问题陈述，并使用两种不同的方法解决了它。

和往常一样，代码可以在 Github 的[上获得。](https://web.archive.org/web/20221127043744/https://github.com/eugenp/tutorials/tree/master/algorithms-modules/algorithms-miscellaneous-6)