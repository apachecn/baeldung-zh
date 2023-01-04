# Java 中的 CDI(上下文和依赖注入)介绍

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-ee-cdi>

## 1。概述

CDI(Contexts and Dependency Injection)是 Java EE 6 及更高版本中包含的标准[依赖注入](https://web.archive.org/web/20220817121616/https://en.wikipedia.org/wiki/Dependency_injection)框架。

它允许我们通过特定领域的生命周期上下文来管理有状态组件的生命周期，并以类型安全的方式将组件(服务)注入客户端对象。

在本教程中，我们将深入了解 CDI 最相关的特性，并实现在客户端类中注入依赖项的不同方法。

## 2。DYDI(自己动手依赖注入)

简而言之，根本不需要借助任何框架就可以实现 DI。

这种方法通常被称为 DYDI(自己动手依赖注入)。

使用 DYDI，我们通过简单的旧工厂/构建器将所需的依赖关系传递给客户端类，从而将应用程序代码与对象创建隔离开来。

下面是一个基本的 DYDI 实现的样子:

```
public interface TextService {
    String doSomethingWithText(String text);
    String doSomethingElseWithText(String text);    
}
```

```
public class SpecializedTextService implements TextService { ... }
```

```
public class TextClass {
    private TextService textService;

    // constructor
}
```

```
public class TextClassFactory {

    public TextClass getTextClass() {
        return new TextClass(new SpecializedTextService(); 
    }    
}
```

当然，DYDI 适用于一些相对简单的用例。

如果我们的示例应用程序在规模和复杂性上增长，实现一个更大的互连对象网络，我们最终会被大量的对象图工厂所污染。

这将需要大量样板代码来创建对象图。这不是一个完全可扩展的解决方案。

我们能做得更好吗？当然可以。这正是 CDI 的用武之地。

## 3。一个简单的例子

CDI 将 DI 变成了一个简单的过程，归结起来就是用一些简单的注释来修饰服务类，并在客户端类中定义相应的注入点。

为了展示 CDI 如何在最基础的层面上实现 DI，让我们假设我们想要开发一个简单的图像文件编辑应用程序。能够打开、编辑、写入、保存图像文件等等。

### 3.1。`“beans.xml”`文件

首先，我们必须在`“src/main/resources/META-INF/”`文件夹中放置一个`“beans.xml”`文件。**即使这个文件根本不包含任何特定的 DI 指令，它也是启动和运行 c DI 所必需的**:

```
<beans  
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://java.sun.com/xml/ns/javaee 
  http://java.sun.com/xml/ns/javaee/beans_1_0.xsd">
</beans>
```

### 3.2。服务等级

接下来，让我们创建对 GIF、JPG 和 PNG 文件执行上述文件操作的服务类:

```
public interface ImageFileEditor {
    String openFile(String fileName);
    String editFile(String fileName);
    String writeFile(String fileName);
    String saveFile(String fileName);
}
```

```
public class GifFileEditor implements ImageFileEditor {

    @Override
    public String openFile(String fileName) {
        return "Opening GIF file " + fileName;
    }

    @Override
    public String editFile(String fileName) {
      return "Editing GIF file " + fileName;
    }

    @Override
    public String writeFile(String fileName) {
        return "Writing GIF file " + fileName;
    }

    @Override
    public String saveFile(String fileName) {
        return "Saving GIF file " + fileName;
    }
}
```

```
public class JpgFileEditor implements ImageFileEditor {
    // JPG-specific implementations for openFile() / editFile() / writeFile() / saveFile()
    ...
}
```

```
public class PngFileEditor implements ImageFileEditor {
    // PNG-specific implementations for openFile() / editFile() / writeFile() / saveFile()
    ...
}
```

### 3.3。客户端类

最后，让我们实现一个客户端类，它在构造函数中接受一个`ImageFileEditor`实现，让我们用`@Inject`注释定义一个注入点:

```
public class ImageFileProcessor {

    private ImageFileEditor imageFileEditor;

    @Inject
    public ImageFileProcessor(ImageFileEditor imageFileEditor) {
        this.imageFileEditor = imageFileEditor;
    }
}
```

简单地说， **`@Inject`注释是 CDI 的实际主力。它允许我们在客户端类中定义注入点。**

在这种情况下，`@Inject`指示 CDI 在构造函数中注入一个`ImageFileEditor`实现。

此外，还可以通过在字段(字段注入)和设置器(设置器注入)中使用`@Inject`注释来注入服务。我们稍后将研究这些选项。

### 3.4。建立带有焊缝的`ImageFileProcessor`目标图形

当然，我们需要确保 CDI 将正确的`ImageFileEditor`实现注入到`ImageFileProcessor`类构造函数中。

为此，首先，我们应该获得该类的一个实例。

**由于我们不依赖任何 Java EE 应用服务器来使用 CDI，我们将使用 Java SE 中的 CDI 参考实现 [Weld](https://web.archive.org/web/20220817121616/http://weld.cdi-spec.org/) 来完成这项工作**:

```
public static void main(String[] args) {
    Weld weld = new Weld();
    WeldContainer container = weld.initialize();
    ImageFileProcessor imageFileProcessor = container.select(ImageFileProcessor.class).get();

    System.out.println(imageFileProcessor.openFile("file1.png"));

    container.shutdown();
} 
```

这里，我们创建一个`WeldContainer`对象，然后获取一个`ImageFileProcessor`对象，最后调用它的`openFile()`方法。

不出所料，如果我们运行应用程序，CDI 会抛出一个`DeploymentException:`来大声抱怨

```
Unsatisfied dependencies for type ImageFileEditor with qualifiers @Default at injection point...
```

**我们得到这个异常是因为 CDI 不知道将什么`ImageFileEditor`实现注入到`ImageFileProcessor` 构造函数中。**

在 CDI 的术语**中，这被称为模糊注入异常**。

### 3.5。`@Default`和`@Alternative`标注

解决这种歧义很容易。**默认情况下，CDI 用`@Default`注释来注释一个接口的所有实现。**

因此，我们应该明确地告诉它应该将哪个实现注入到客户端类中:

```
@Alternative
public class GifFileEditor implements ImageFileEditor { ... }
```

```
@Alternative
public class JpgFileEditor implements ImageFileEditor { ... } 
```

```
public class PngFileEditor implements ImageFileEditor { ... }
```

在这种情况下，我们已经用`@Alternative`注释注释了`GifFileEditor`和`JpgFileEditor`，所以 CDI 现在知道`PngFileEditor`(默认用`@Default`注释注释)是我们想要注入的实现。

如果我们重新运行应用程序，这次它将按预期执行:

```
Opening PNG file file1.png 
```

此外，用`@Default`注释来注释`PngFileEditor`,并保留其他实现作为替代，将会产生与上面相同的结果。

简而言之，这显示了**我们如何通过简单地切换服务类**中的`@Alternative`注释来非常容易地交换实现的运行时注入。

## 4。现场注射

CDI 支持开箱即用的字段和 setter 注入。

下面是如何执行字段注入的(**使用`@Default`和`@Alternative`注释来限定服务的规则保持不变**):

```
@Inject
private final ImageFileEditor imageFileEditor;
```

## 5。设定注射

类似地，下面是如何进行 setter 注入:

```
@Inject 
public void setImageFileEditor(ImageFileEditor imageFileEditor) { ... }
```

## 6。`@Named`注解

到目前为止，我们已经学习了如何在客户端类中定义注入点，并使用 `@Inject`、`@Default`和`@Alternative`注释注入服务，这些注释涵盖了大多数用例。

然而，CDI 也允许我们用`@Named`注释执行服务注入。

**该方法通过将一个有意义的名称绑定到一个实现，提供了一种更具语义的注入服务的方式:**

```
@Named("GifFileEditor")
public class GifFileEditor implements ImageFileEditor { ... }

@Named("JpgFileEditor")
public class JpgFileEditor implements ImageFileEditor { ... }

@Named("PngFileEditor")
public class PngFileEditor implements ImageFileEditor { ... }
```

现在，我们应该重构`ImageFileProcessor`类中的注入点，以匹配一个命名的实现:

```
@Inject 
public ImageFileProcessor(@Named("PngFileEditor") ImageFileEditor imageFileEditor) { ... }
```

还可以使用命名实现执行字段和设置器注入，这看起来非常类似于使用`@Default`和`@Alternative`注释:

```
@Inject 
private final @Named("PngFileEditor") ImageFileEditor imageFileEditor;

@Inject 
public void setImageFileEditor(@Named("PngFileEditor") ImageFileEditor imageFileEditor) { ... }
```

## 7。`@Produces`注解

有时，一个服务在被注入以处理额外的依赖关系之前，需要完全初始化一些配置。

CDI 通过`@Produces`注释为这些情况提供了支持。

**`@Produces`允许我们实现工厂类，其职责是创建完全初始化的服务。**

为了理解`@Produces`注释是如何工作的，让我们重构`ImageFileProcessor`类，这样它可以在构造函数中接受一个额外的`TimeLogger`服务。

该服务将用于记录执行某个图像文件操作的时间:

```
@Inject
public ImageFileProcessor(ImageFileEditor imageFileEditor, TimeLogger timeLogger) { ... } 

public String openFile(String fileName) {
    return imageFileEditor.openFile(fileName) + " at: " + timeLogger.getTime();
}

// additional image file methods 
```

在这种情况下，`TimeLogger`类接受两个额外的服务，`SimpleDateFormat`和`Calendar`:

```
public class TimeLogger {

    private SimpleDateFormat dateFormat;
    private Calendar calendar;

    // constructors

    public String getTime() {
        return dateFormat.format(calendar.getTime());
    }
}
```

我们如何告诉 CDI 在哪里寻找一个完全初始化的`TimeLogger`对象？

我们只需创建一个`TimeLogger`工厂类，并用`@Produces`注释对其工厂方法进行注释:

```
public class TimeLoggerFactory {

    @Produces
    public TimeLogger getTimeLogger() {
        return new TimeLogger(new SimpleDateFormat("HH:mm"), Calendar.getInstance());
    }
}
```

每当我们得到一个`ImageFileProcessor`实例时，CDI 将扫描`TimeLoggerFactory`类，然后调用`getTimeLogger()`方法(因为它用`@Produces`注释进行了注释)，最后注入`Time Logger`服务。

如果我们用`Weld`运行重构的示例应用程序，它将输出以下内容:

```
Opening PNG file file1.png at: 17:46
```

## 8。自定义限定符

CDI 支持使用自定义限定符来限定依赖项和解决不明确的注入点。

自定义限定符是一个非常强大的功能。它们不仅将语义名称绑定到服务，还绑定了注入元数据。元数据，例如[保留策略](https://web.archive.org/web/20220817121616/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/annotation/RetentionPolicy.html)和合法注释目标([元素类型](https://web.archive.org/web/20220817121616/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/annotation/ElementType.html))。

让我们看看如何在我们的应用程序中使用自定义限定符:

```
@Qualifier
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.FIELD, ElementType.METHOD, ElementType.TYPE, ElementType.PARAMETER})
public @interface GifFileEditorQualifier {} 
```

```
@Qualifier
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.FIELD, ElementType.METHOD, ElementType.TYPE, ElementType.PARAMETER})
public @interface JpgFileEditorQualifier {} 
```

```
@Qualifier
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.FIELD, ElementType.METHOD, ElementType.TYPE, ElementType.PARAMETER})
public @interface PngFileEditorQualifier {} 
```

现在，让我们将自定义限定符绑定到`ImageFileEditor`实现:

```
@GifFileEditorQualifier
public class GifFileEditor implements ImageFileEditor { ... } 
```

```
@JpgFileEditorQualifier
public class JpgFileEditor implements ImageFileEditor { ... }
```

```
@PngFileEditorQualifier
public class PngFileEditor implements ImageFileEditor { ... } 
```

最后，让我们重构`ImageFileProcessor`类`:`中的注入点

```
@Inject
public ImageFileProcessor(@PngFileEditorQualifier ImageFileEditor imageFileEditor, TimeLogger timeLogger) { ... } 
```

如果我们再次运行我们的应用程序，它应该生成如上所示的输出。

自定义限定符为将名称和注释元数据绑定到实现提供了一种简洁的语义方法。

此外，**自定义限定符允许我们定义更严格的类型安全注入点(优于@Default 和@Alternative 注释的功能)**。

如果在类型层次结构中只有一个子类型是合格的，那么 CDI 将只注入子类型，而不是基类型。

## 9。结论

毫无疑问， **CDI 使依赖注入成为一件容易的事情**，额外注释的成本对于有组织的依赖注入的收益来说是微不足道的。

有时候，DYDI 仍然比 CDI 有优势。比如在开发只包含简单对象图的相当简单的应用程序时。

和往常一样，本文中展示的所有代码示例都可以在 GitHub 上的[处获得。](https://web.archive.org/web/20220817121616/https://github.com/eugenp/tutorials/tree/master/cdi/src/main/java/com/baeldung/dependencyinjection)