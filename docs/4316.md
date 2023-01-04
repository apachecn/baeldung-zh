# Java 中的变量作用域

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-variable-scope>

## 1.概观

与任何编程语言一样，在 Java 中，每个变量都有一个作用域。这是可以使用变量并且变量有效的程序段。

在本教程中，我们将介绍 Java 中可用的作用域，并讨论它们之间的区别。

## 2.类别范围

在类的括号(`{}`)内用`private`访问修饰符声明但在任何方法外声明的每个变量都有类范围。因此，**这些变量可以在类中的任何地方使用，但不能在类外使用**:

```
public class ClassScopeExample {
    private Integer amount = 0;
    public void exampleMethod() {
        amount++;
    }
    public void anotherExampleMethod() {
        Integer anotherAmount = amount + 4;
    }
}
```

我们可以看到`ClassScopeExample`有一个类变量`amount` ，可以在该类的方法中访问。

如果我们不使用`private`，它将可以从整个包中访问。查看[访问修饰符文章](/web/20221129020122/https://www.baeldung.com/java-access-modifiers)了解更多信息。

## 3.方法范围

当一个变量在一个方法中被声明时，它有方法作用域和**，它只在同一个方法中有效:**

```
public class MethodScopeExample {
    public void methodA() {
        Integer area = 2;
    }
    public void methodB() {
        // compiler error, area cannot be resolved to a variable
        area = area + 2;
    }
}
```

在`methodA`中，我们创建了一个名为`area`的方法变量。因此，我们可以在`methodA`中使用`area`，但是不能在其他地方使用。

## 4.循环范围

如果我们在一个循环中声明一个变量，它将有一个循环范围，并且**将只在循环中可用:**

```
public class LoopScopeExample {
    List<String> listOfNames = Arrays.asList("Joe", "Susan", "Pattrick");
    public void iterationOfNames() {
        String allNames = "";
        for (String name : listOfNames) {
            allNames = allNames + " " + name;
        }
        // compiler error, name cannot be resolved to a variable
        String lastNameUsed = name;
    }
}
```

我们可以看到方法`iterationOfNames`有一个名为`name`的方法变量。该变量只能在循环内部使用，在循环外部无效。

## 5.括号范围

**我们可以使用括号** ( `{}`)在任何地方定义额外的作用域:

```
public class BracketScopeExample {    
    public void mathOperationExample() {
        Integer sum = 0;
        {
            Integer number = 2;
            sum = sum + number;
        }
        // compiler error, number cannot be solved as a variable
        number++;
    }
}
```

变量`number`只在括号内有效。

## 6.范围和变量隐藏

假设我们有一个类变量，我们想声明一个同名的方法变量:

```
public class NestedScopesExample {
    String title = "Baeldung";
    public void printTitle() {
        System.out.println(title);
        String title = "John Doe";
        System.out.println(title);
    }
}
```

我们第一次打印`title`时，它会打印“Baeldung”。之后，声明一个同名的方法变量，并给它赋值“John Doe `“.`

标题方法变量覆盖了再次访问`class`变量`title`的可能性。这就是为什么第二次，它会打印“无名氏`“`。

**令人困惑，对吗？这被称为`variable shadowing`** ，并不是一个好的做法。最好使用前缀`this `来访问`title`类变量，比如`this.title`。

## 7.结论

我们学习了 Java 中存在的不同作用域。

和往常一样，代码可以在 [GitHub](https://web.archive.org/web/20221129020122/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-syntax-2) 上获得。