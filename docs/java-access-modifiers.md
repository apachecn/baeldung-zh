# Java 中的访问修饰符

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-access-modifiers>

## 1。概述

在本教程中，我们将学习 Java 中的访问修饰符，它们用于设置类、变量、方法和构造函数的访问级别。

**简单来说，有四个访问修饰符:** `public`、`private`、`protected`、`default`(无关键字)。

在我们开始之前，让我们注意一个顶级类只能使用`public`或`default`访问修饰符。在成员级别，我们可以使用所有四个。

## 2。默认

当我们不显式使用任何关键字时，Java 会设置一个对给定类、方法或属性的`default`访问。默认的访问修饰符也叫做`package-private`，这意味着**所有的成员在同一个包**中是可见的，但是不能从其他包中访问:

```
package com.baeldung.accessmodifiers;

public class SuperPublic {
    static void defaultMethod() {
        ...
    }
}
```

`defaultMethod()`可在同一包的另一个类中访问:

```
package com.baeldung.accessmodifiers;

public class Public {
    public Public() {
        SuperPublic.defaultMethod(); // Available in the same package.
    }
}
```

但是，它在其他包中不可用。

## 3。公共

如果我们将关键字`public`添加到一个类、方法或属性中，那么**我们就可以让全世界**都可以使用它，也就是说，所有包中的所有其他类都可以使用它。这是限制最少的访问修饰符:

```
package com.baeldung.accessmodifiers;

public class SuperPublic {
    public static void publicMethod() {
        ...
    }
}
```

`publicMethod()`在另一个包装中提供:

```
package com.baeldung.accessmodifiers.another;

import com.baeldung.accessmodifiers.SuperPublic;

public class AnotherPublic {
    public AnotherPublic() {
        SuperPublic.publicMethod(); // Available everywhere. Let's note different package.
    }
}
```

关于`public`关键字在应用于类、接口、嵌套公共类或接口和方法时如何表现的更多细节，请参见[专题文章](/web/20220831174039/https://www.baeldung.com/java-public-keyword)。

## 4。私人

任何带有`private`关键字**的方法、属性或构造函数只能从同一个类**中访问。这是最严格的访问修饰符，是封装概念的核心。所有数据将对外界隐藏:

```
package com.baeldung.accessmodifiers;

public class SuperPublic {
    static private void privateMethod() {
        ...
    }

     private void anotherPrivateMethod() {
         privateMethod(); // available in the same class only.
    }
}
```

这篇更详细的文章将展示`private`关键字在应用于字段、构造函数、方法和内部类时的行为。

## 5。受保护的

在`public`和`private`访问级别之间，有`protected`访问修饰符。

如果我们用关键字`protected`声明一个方法、属性或构造函数，我们可以从**相同的包(和`package-private`访问级别一样)中访问成员，此外还可以从它的类**的所有子类中访问成员，即使它们位于其他包中:

```
package com.baeldung.accessmodifiers;

public class SuperPublic {
    static protected void protectedMethod() {
        ...
    }
}
```

`protectedMethod()`在子类中可用(不考虑包):

```
package com.baeldung.accessmodifiers.another;

import com.baeldung.accessmodifiers.SuperPublic;

public class AnotherSubClass extends SuperPublic {
    public AnotherSubClass() {
        SuperPublic.protectedMethod(); // Available in subclass. Let's note different package.
    }
}
```

[专用文章](/web/20220831174039/https://www.baeldung.com/java-protected-access-modifier)描述了更多关于关键字在字段、方法、构造函数、内部类中的使用，以及在同一包或不同包中的可访问性。

## 6。比较

下表总结了可用的访问修饰符。我们可以看到，不管使用什么访问修饰符，一个类总是可以访问它的成员:

| 修饰语 | 班级 | 包裹 | 亚纲 | 世界 |
| 

```
public
```

 | Y | Y | Y | Y |
| 

```
protected
```

 | Y | Y | Y | 普通 |
| 

```
default
```

 | Y | Y | 普通 | 普通 |
| 

```
private
```

 | Y | 普通 | 普通 | 普通 |

## 7 .**。结论**

在这篇短文中，我们回顾了 Java 中的访问修饰符。

对任何给定的成员使用尽可能严格的访问级别来防止误用是一种好的做法。除非有充分的理由，否则我们应该总是使用`private`访问修饰符。

`Public`只有当成员是 API 的一部分时，才应使用访问级别。

与往常一样，Github 上的[提供了代码示例。](https://web.archive.org/web/20220831174039/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-oop-modifiers)