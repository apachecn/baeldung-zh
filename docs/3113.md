# Java 中的多态性

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-polymorphism>

## 1。概述

所有面向对象编程(OOP)语言都需要展现四个基本特征:抽象、封装、继承和[多态性](/web/20220823134752/https://www.baeldung.com/cs/polymorphism)。

在本文中，我们涵盖了两种核心类型的多态性:`**static or compile-time polymorphism** and **dynamic or runtime** **polymorphism**`。静态多态在[编译时](/web/20220823134752/https://www.baeldung.com/cs/compile-load-execution-time)执行，而动态多态在[运行时](/web/20220823134752/https://www.baeldung.com/cs/runtime-vs-compile-time)实现。

## 2。 **静态多态**

根据[维基百科](https://web.archive.org/web/20220823134752/https://en.wikipedia.org/wiki/Template_metaprogramming#Static_polymorphism)的说法，静态多态是对**多态的模仿，它在编译时被解析，因此消除了运行时虚拟表查找**。

例如，我们在文件管理器应用程序中的`TextFile`类可以有三个与`read()`方法签名相同的方法:

```
public class TextFile extends GenericFile {
    //...

    public String read() {
        return this.getContent()
          .toString();
    }

    public String read(int limit) {
        return this.getContent()
          .toString()
          .substring(0, limit);
    }

    public String read(int start, int stop) {
        return this.getContent()
          .toString()
          .substring(start, stop);
    }
}
```

在代码编译期间，编译器会验证所有对`read`方法的调用是否至少对应于上面定义的三种方法中的一种。

## 3。动态多态性

使用动态多态， **Java 虚拟机(JVM)处理当子类被分配给其父表单**时要执行的适当方法的检测。这是必要的，因为子类可能会覆盖父类中定义的部分或全部方法。

在一个假设的文件管理器应用程序中，让我们为所有文件定义一个名为`GenericFile`的父类:

```
public class GenericFile {
    private String name;

    //...

    public String getFileInfo() {
        return "Generic File Impl";
    }
}
```

我们还可以实现一个`ImageFile` 类，它扩展了`GenericFile`，但是覆盖了`getFileInfo()`方法并附加了更多信息:

```
public class ImageFile extends GenericFile {
    private int height;
    private int width;

    //... getters and setters

    public String getFileInfo() {
        return "Image File Impl";
    }
}
```

当我们创建一个`ImageFile`的实例并将其分配给一个`GenericFile`类时，隐式转换就完成了。然而，JVM 保留了对`ImageFile`实际形式的引用。

**上面的构造类似于方法覆盖。**我们可以通过以下方式调用`getFileInfo()`方法来确认这一点:

```
public static void main(String[] args) {
    GenericFile genericFile = new ImageFile("SampleImageFile", 200, 100, 
      new BufferedImage(100, 200, BufferedImage.TYPE_INT_RGB)
      .toString()
      .getBytes(), "v1.0.0");
    logger.info("File Info: \n" + genericFile.getFileInfo());
}
```

正如所料，`genericFile.getFileInfo()`触发了`ImageFile`类的`getFileInfo()`方法，如下图所示:

```
File Info: 
Image File Impl
```

## 4。Java 中的其他多态特性

除了 Java 中这两种主要类型的多态性，Java 编程语言中还有其他一些表现出多态性的特征。让我们来讨论其中的一些特征。

### 4.1。强制

多态强制处理由编译器完成的隐式类型转换，以防止类型错误。整数和字符串串联就是一个典型的例子:

```
String str = “string” + 2;
```

### 4.2。操作员过载

操作符或方法重载指的是同一符号或操作符的多态特征，根据上下文具有不同的含义(形式)。

例如，加号(+)可用于数学加法以及`String`连接。在任一情况下，只有上下文(即参数类型)决定符号的解释:

```
String str = "2" + 2;
int sum = 2 + 2;
System.out.printf(" str = %s\n sum = %d\n", str, sum);
```

输出:

```
str = 22
sum = 4
```

### 4.3。多态参数

参数多态性允许类中的参数或方法的名称与不同的类型相关联。下面是一个典型的例子，我们将`content`定义为`String`，然后定义为`Integer`:

```
public class TextFile extends GenericFile {
    private String content;

    public String setContentDelimiter() {
        int content = 100;
        this.content = this.content + content;
    }
}
```

同样需要注意的是，多态参数的**声明会导致一个被称为** **变量隐藏**的问题，其中一个参数的局部声明总是覆盖另一个同名参数的全局声明。

为了解决这个问题，通常建议使用全局引用，如关键字`this`来指向局部上下文中的全局变量。

### 4.4。多态亚型

多态子类型方便地使我们可以为一个类型分配多个子类型，并期望对该类型的所有调用都触发子类型中的可用定义。

例如，如果我们有一个`GenericFile`的集合，并且我们对它们中的每一个调用`getInfo()`方法，我们可以预期输出是不同的，这取决于集合中的每一项是从哪个子类型派生的:

```
GenericFile [] files = {new ImageFile("SampleImageFile", 200, 100, 
  new BufferedImage(100, 200, BufferedImage.TYPE_INT_RGB).toString() 
  .getBytes(), "v1.0.0"), new TextFile("SampleTextFile", 
  "This is a sample text content", "v1.0.0")};

for (int i = 0; i < files.length; i++) {
    files[i].getInfo();
}
```

**子类型多态性通过组合** **向上转换和后期绑定**成为可能。向上转换包括将继承层次结构从超类型转换为子类型:

```
ImageFile imageFile = new ImageFile();
GenericFile file = imageFile;
```

上面的结果是不能在新的向上转换`GenericFile`上调用`ImageFile-`特定的方法。但是，子类型中的方法会覆盖父类型中定义的类似方法。

为了解决在向上造型为超类型时不能调用特定于子类型的方法的问题，我们可以将继承从超类型向下造型为子类型。这通过以下方式实现:

```
ImageFile imageFile = (ImageFile) file;
```

**后期绑定** **策略帮助编译器决定在向上转换**后触发哪个方法。在上面例子中 i `mageFile#getInfo` vs `file#getInfo`的情况下，编译器保留了对`ImageFile`的`getInfo`方法的引用。

## 5。多态性问题

让我们看看多态性中的一些模糊性，如果没有正确检查，它们可能会导致运行时错误。

### 5.1。向下浇铸时的类型识别

回想一下，我们之前在执行向上转换后，失去了对某些特定于子类型的方法的访问。尽管我们能够通过向下转换解决这个问题，但这并不能保证实际的类型检查。

例如，如果我们执行向上转换和随后的向下转换:

```
GenericFile file = new GenericFile();
ImageFile imageFile = (ImageFile) file;
System.out.println(imageFile.getHeight());
```

我们注意到编译器允许将一个`GenericFile`向下转换成一个`ImageFile`，即使这个类实际上是一个`GenericFile`而不是一个`ImageFile`。

因此，如果我们试图调用`imageFile`类上的`getHeight()`方法，我们会得到一个`ClassCastException`,因为`GenericFile`没有定义`getHeight()`方法:

```
Exception in thread "main" java.lang.ClassCastException:
GenericFile cannot be cast to ImageFile
```

为了解决这个问题，JVM 执行运行时类型信息(RTTI)检查。我们也可以尝试使用关键字`instanceof`进行显式类型识别，就像这样:

```
ImageFile imageFile;
if (file instanceof ImageFile) {
    imageFile = file;
}
```

以上有助于避免运行时出现`ClassCastException`异常。可以使用的另一个选项是将石膏包裹在`try`和`catch`块中，并抓住`ClassCastException.`

应该注意的是，由于有效验证类型是否正确需要时间和资源，因此 **RTTI 检查是昂贵的**。此外，频繁使用关键字`instanceof`几乎总是暗示着糟糕的设计。

### 5.2。脆弱基类问题

根据维基百科的说法，如果对基类进行看似安全的修改可能导致派生类失灵，那么基类或超类就被认为是脆弱的。

让我们考虑一个名为`GenericFile`的超类及其子类`TextFile`的声明:

```
public class GenericFile {
    private String content;

    void writeContent(String content) {
        this.content = content;
    }
    void toString(String str) {
        str.toString();
    }
}
```

```
public class TextFile extends GenericFile {
    @Override
    void writeContent(String content) {
        toString(content);
    }
}
```

当我们修改`GenericFile`类时:

```
public class GenericFile {
    //...

    void toString(String str) {
        writeContent(str);
    }
}
```

我们观察到上面的修改使`TextFile`在`writeContent()`方法中处于无限递归中，最终导致堆栈溢出。

为了解决脆弱的基类问题，我们可以使用`final`关键字来防止子类覆盖`writeContent()`方法。适当的文档也会有所帮助。最后但并非最不重要的一点是，组成通常应该优先于继承。

## 6。结论

在本文中，我们讨论了多态性的基本概念，重点是优点和缺点。

和往常一样，这篇文章的源代码可以在 GitHub 的 [找到。](https://web.archive.org/web/20220823134752/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-oop-inheritance)