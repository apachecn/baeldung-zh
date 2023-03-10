# 在 Java 中切换布尔变量

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-toggle-boolean>

## 1.概观

Boolean 是 Java 中的一种基本数据类型。通常，它只能有两个值，`true`或`false`。

在本教程中，我们将讨论如何切换给定的布尔变量。

## 2.问题简介

这个问题很简单。简单地说，我们想要反转一个布尔变量的值。比如切换后`true`变成了`false`。

但是，**我们要注意，Java 中有两种“不同”的布尔类型，[原语](/web/20220913100020/https://www.baeldung.com/java-primitives-vs-objects) `boolean`和[装箱](/web/20220913100020/https://www.baeldung.com/java-wrapper-classes) `Boolean`。**因此，理想的切换方法应该适用于这两种类型。

在本教程中，我们将讨论如何实现这样一个方法。

此外，为了简单起见，我们将使用单元测试断言来验证我们的实现是否按预期工作。

接下来，让我们从切换一个原始的`boolean`变量开始，因为这是我们最终的`toggle()`方法的基础。

## 3.切换原始`boolean`变量

**切换原始`boolean`变量最直接的方法是使用[非操作符](/web/20220913100020/https://www.baeldung.com/java-operators#3-the-logical-complement-operator) ( `!`)。**

让我们创建一个测试来看看它是如何工作的:

```java
boolean b = true;
b = !b;
assertFalse(b);

b = !b;
assertTrue(b); 
```

如果我们运行这个测试，它通过了。因此，每次我们对一个`boolean`变量执行 NOT 操作时，它的值都会被反转。

或者，[XOR 运算符](/web/20220913100020/https://www.baeldung.com/java-operators#3-the-bitwise-xor-operator) ( `^`)也可以反转一个布尔值。在考虑实现之前，让我们快速了解一下 XOR 运算符是如何工作的。

**给定两个`boolean`变量`b1`和`b2`，只有当`b1`和`b2`的值**不同时，`b1 ^ b2`才是`true`，例如:

*   `true ^ true = false`
*   `false ^ false = false`
*   `true ^ false = true`

因此，我们可以利用异或的特性，**执行`b ^ true`来反转`b`的值**:

*   `b = true -> b ^ true` 变成了 `true ^ true = false`
*   `b = false -> b ^ true`变成了 `false ^ true = true`

既然我们已经理解了 XOR 的逻辑，那么将其翻译成 Java 代码对我们来说并不是一项具有挑战性的任务:

```java
boolean b = true;
b ^= true;
assertFalse(b);

b ^= true;
assertTrue(b); 
```

不出所料，当我们运行时，测试通过了。

## 4.创建`toggle()`方法

我们已经看到一个原始的`boolean`变量只能有两个值:`true`和`false`。但是，与原语`boolean`、**不同，装箱的`Boolean`变量可以容纳`null`、**、`.`

当我们对一个`Boolean`变量执行 NOT 或 XOR 运算时，Java 会自动取消`Boolean`到`boolean`的装箱。但是如果我们不妥善处理`null`事件，我们会遇到`NullPointerException`:

```java
assertThatThrownBy(() -> {
    Boolean b = null;
    b = !b;
}).isInstanceOf(NullPointerException.class); 
```

如果我们执行上面的测试，它就通过了。不幸的是，这意味着`NullPointerException`发生在我们执行`!b`的时候。

所以接下来，让我们创建空安全的`toggle()`方法来处理`Boolean`和`boolean`变量:

```java
static Boolean toggle(Boolean b) {
    if (b == null){
        return b;
    }
    return !b;
} 
```

这里，我们首先执行一个空的`–`检查，然后使用 NOT 操作符来反转这个值。当然，如果我们愿意，在空值检查之后，我们也可以使用 XOR 方法来反转`b`的值。

最后，让我们创建一个测试来验证我们的`toggle()`方法是否适用于所有情况:

```java
// boxed Boolean
Boolean b = true;
b = ToggleBoolean.toggle(b);
assertFalse(b);

b = ToggleBoolean.toggle(b);
assertTrue(b);

b = null;
b = ToggleBoolean.toggle(b);
assertNull(b);

// primitive boolean
boolean bb = true;
bb = ToggleBoolean.toggle(bb);
assertFalse(bb);
bb = ToggleBoolean.toggle(bb);
assertTrue(bb); 
```

正如上面的测试所示，我们用一个`Boolean`变量和一个`boolean`变量测试了`toggle()`方法。此外，我们已经测试了`Boolean`变量`b=null`的场景。

当我们执行它时，测试就通过了。因此，我们的`toggle()`方法按预期工作。

## 5.结论

在本文中，我们学习了如何构建一个空安全的方法来切换给定的`boolean/Boolean`变量。

和往常一样，与本文相关的代码可以从 GitHub 上的[处获得。](https://web.archive.org/web/20220913100020/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-5)