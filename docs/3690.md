# Java 中方法的签名包括返回类型吗？

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-method-signature-return-type>

## 1.概观

方法签名只是 Java 中整个方法定义的一个子集。因此，签名的确切结构可能会引起混淆。

在本教程中，我们将学习方法签名的元素及其在 Java 编程中的含义。

## 2.方法签名

[Java](/web/20221208143921/https://www.baeldung.com/java-methods)中的方法支持重载，也就是说可以在同一个类或者类的层次结构中定义多个同名的方法。因此，编译器必须能够静态绑定客户端代码引用的方法。由于这个原因，方法**签名惟一地标识了每个方法**。

根据[甲骨文](https://web.archive.org/web/20221208143921/https://docs.oracle.com/javase/tutorial/java/javaOO/methods.html)，方法**签名由名称和参数类型**组成。因此，方法声明的所有其他元素，如修饰符、返回类型、参数名、异常列表和主体都不是签名的一部分。

让我们仔细看看[方法重载](/web/20221208143921/https://www.baeldung.com/java-method-overload-override)以及它与方法签名的关系。

## 3.过载错误

让我们考虑下面的代码`:`

```
public void print() {
    System.out.println("Signature is: print()");
}

public void print(int parameter) {
    System.out.println("Signature is: print(int)");
}
```

正如我们所看到的，由于方法有不同的参数类型列表，代码可以编译。实际上，编译器可以确定性地将任何调用绑定到一个或另一个。

现在，让我们通过添加以下方法来测试重载是否合法:

```
public int print() { 
    System.out.println("Signature is: print()"); 
    return 0; 
}
```

当我们编译时，我们得到一个“类中已经定义了方法”的错误。这证明方法**返回类型不是方法签名**的一部分。

让我们用同样的方法来尝试修改:

```
private final void print() { 
    System.out.println("Signature is: print()");
}
```

我们仍然看到同样的“类中已经定义了方法”错误。因此，方法**签名不依赖于修饰符**。

通过更改抛出的异常进行重载可以通过添加以下内容来测试:

```
public void print() throws IllegalStateException { 
    System.out.println("Signature is: print()");
    throw new IllegalStateException();
}
```

我们再次看到“方法已经在类中定义”错误，表明**抛出声明不能是签名**的一部分。

我们可以测试的最后一件事是改变参数名是否允许重载。让我们添加以下方法:

```
public void print(int anotherParameter) { 
    System.out.println("Signature is: print(int)");
}
```

不出所料，我们得到了同样的编译错误。这意味着**参数名不会影响方法签名**。

## 3.泛型和类型擦除

用通用参数**、[型擦除](/web/20221208143921/https://www.baeldung.com/java-type-erasure)改变有效签名**。实际上，它可能会导致与另一个使用泛型类型上限而不是泛型标记的方法发生冲突。

让我们考虑下面的代码:

```
public class OverloadingErrors<T extends Serializable> {

    public void printElement(T t) {
        System.out.println("Signature is: printElement(T)");
    }

    public void printElement(Serializable o) {
        System.out.println("Signature is: printElement(Serializable)");
    }
}
```

即使签名看起来不同，编译器也无法在类型擦除后静态绑定正确的方法。

我们可以看到，由于类型擦除，编译器用上限`Serializable,`替换了`T`。因此，它与显式使用`Serializable`的方法相冲突。

当泛型没有绑定时，我们会看到基本类型`Object` 的相同结果。

## 4.参数列表和多态性

方法签名考虑了确切的类型。这意味着我们可以重载一个参数类型是子类或超类的方法。

然而，我们必须特别注意，因为**静态绑定将尝试使用多态性、自动装箱和类型提升**来匹配。

让我们来看看下面的代码:

```
public Number sum(Integer term1, Integer term2) {
    System.out.println("Adding integers");
    return term1 + term2;
}

public Number sum(Number term1, Number term2) {
    System.out.println("Adding numbers");
    return term1.doubleValue() + term2.doubleValue();
}

public Number sum(Object term1, Object term2) {
    System.out.println("Adding objects");
    return term1.hashCode() + term2.hashCode();
}
```

上面的代码完全合法，可以编译。当调用这些方法时可能会产生混淆，因为我们不仅需要知道我们调用的确切方法签名，还需要知道 Java 如何基于实际值静态绑定。

让我们探索一些最终绑定到`sum(Integer, Integer)`的方法调用:

```
StaticBinding obj = new StaticBinding(); 
obj.sum(Integer.valueOf(2), Integer.valueOf(3)); 
obj.sum(2, 3); 
obj.sum(2, 0x1);
```

对于第一次调用，我们在第二次调用中有了确切的参数类型`Integer, Integer.` ，Java 将为我们自动将`int`装箱为`Integer` 最后，Java 将通过类型提升的方式将字节值`0x1` 转换为`int`，然后自动装箱为`Integer. `

类似地，我们有以下绑定到`sum(Number, Number)`的调用:

```
obj.sum(2.0d, 3.0d);
obj.sum(Float.valueOf(2), Float.valueOf(3));
```

在第一次调用时，我们将`double`值自动装箱到`Double.`，然后通过多态，`Double`与`Number.`完全匹配，`Float`与第二次调用的`Number`匹配。

让我们观察一下，`Float`和`Double`都继承自`Number`和`Object.` 的事实，然而，默认绑定是到`Number`。这是因为 Java 会自动匹配与方法签名匹配的最近的超类型。

现在让我们考虑下面的方法调用:

```
obj.sum(2, "John");
```

在这个例子中，我们有一个用于第一个参数的`int` 到`Integer`自动框。然而，这个方法名没有`sum(Integer, String)`重载。因此，Java 将遍历所有的参数超类型，从最近的父类型转换到`Object`，直到找到匹配。在这种情况下，它绑定到`sum(Object, Object). `

**要改变默认绑定，我们可以使用如下的显式参数转换**:

```
obj.sum((Object) 2, (Object) 3);
obj.sum((Number) 2, (Number) 3);
```

## 5.Vararg 参数

现在让我们把注意力转向 **`[varargs](/web/20221208143921/https://www.baeldung.com/java-varargs)`如何影响方法的有效签名**和静态绑定。

这里我们有一个使用`varargs`的重载方法:

```
public Number sum(Object term1, Object term2) {
    System.out.println("Adding objects");
    return term1.hashCode() + term2.hashCode();
}

public Number sum(Object term1, Object... term2) {
    System.out.println("Adding variable arguments: " + term2.length);
    int result = term1.hashCode();
    for (Object o : term2) {
        result += o.hashCode();
    }
    return result;
}
```

那么方法的有效签名是什么呢？我们已经看到`sum(Object, Object)`是第一个的签名。变量参数本质上是数组，所以编译后第二个变量的有效签名是`sum(Object, Object[]). `

一个棘手的问题是，当我们只有两个参数时，如何选择方法绑定？

让我们考虑以下调用:

```
obj.sum(new Object(), new Object());
obj.sum(new Object(), new Object(), new Object());
obj.sum(new Object(), new Object[]{new Object()});
```

显然，第一个调用将绑定到`sum(Object, Object)` ，第二个将绑定到`sum(Object, Object[]). `为了强制 Java 调用带有两个对象的第二个方法，我们必须像第三个调用一样将其包装在一个数组中。

这里要注意的最后一点是，声明以下方法将与 vararg 版本冲突:

```
public Number sum(Object term1, Object[] term2) {
    // ...
}
```

## 6.结论

在本教程中，我们了解到方法签名由名称和参数类型列表组成。修饰符、返回类型、参数名和异常列表不能区分重载方法，因此不是签名的一部分。

我们还研究了类型擦除和变量如何隐藏有效的方法签名，以及如何覆盖 Java 的静态方法绑定。

像往常一样，本文中显示的所有代码示例都可以在 GitHub 上的[处获得。](https://web.archive.org/web/20221208143921/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-oop-methods)