# 资源包指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-resourcebundle>

## 1。概述

许多软件开发人员在其职业生涯中面临着开发多语言系统或应用程序的机会。这些通常面向来自不同地区或不同语言区域的最终用户。

维护和扩展这些应用程序总是具有挑战性。同时操作各种本地化特定数据的能力通常是至关重要的。应用程序数据的修改应该尽可能简单，不需要重新编译。这就是为什么我们通常避免硬编码标签或按钮名称。

幸运的是，我们可以依靠 Java 提供的这个类，它可以帮助我们解决上面提到的所有问题。

**简单地说，`ResourceBundle`使我们的应用程序能够从包含特定于地区的数据的不同文件中加载数据。**

### 1.1。`ResourceBundles`

我们应该知道的第一件事是，一个资源包**中的所有文件必须在同一个包/目录中，并且有一个公共的基本名称**。它们可能有特定于区域设置的后缀，表示由下划线符号分隔的语言、国家或平台。

重要的是，如果已经有语言代码，我们可以附加国家代码，或者如果语言和国家代码存在，我们可以附加平台代码。

让我们看看文件名示例:

*   `ExampleResource`
*   `ExampleResource_en`
*   `ExampleResource_en_US`
*   `ExampleResource_en_US_UNIX`

每个数据包的默认文件总是没有任何后缀-`ExampleResource`。由于`ResourceBundle`有两个子类: **`PropertyResourceBundle`** 和 **`ListResourceBundle`** ，我们可以在属性文件和 java 文件中互换保存数据。

每个文件必须有一个特定于语言环境的名称和一个适当的文件扩展名，例如`ExampleResource_en_US.properties`或`Example_en.java`。

### 1.2。属性文件-`PropertyResourceBundle`

属性文件由`PropertyResourceBundle.`表示，它们以区分大小写的键值对的形式存储数据。

让我们分析一个样本属性文件:

```
# Buttons
continueButton continue
cancelButton=cancel

! Labels
helloLabel:hello 
```

正如我们所看到的，有三种不同的定义键值对的方式。

都是等价的，但是第一种可能是最受`Java`程序员欢迎的。值得一提的是，我们也可以将注释放在属性文件中。评论总是以`#`或`!`开头。

### 1.3。Java 文件-`ListResourceBundle`

首先，为了存储我们特定于语言的数据，我们需要创建一个扩展`ListResourceBundle`并覆盖`getContents()`方法的类。类名约定与属性文件相同。

对于每个`Locale,`，我们需要创建单独的 Java 类。

下面是一个示例类:

```
public class ExampleResource_pl_PL extends ListResourceBundle {

    @Override
    protected Object[][] getContents() {
        return new Object[][] {
          {"currency", "polish zloty"},
          {"toUsdRate", new BigDecimal("3.401")},
          {"cities", new String[] { "Warsaw", "Cracow" }} 
        };
    }
}
```

与属性文件相比，Java 文件有一个主要的优势，那就是可以保存我们想要的任何对象——不仅仅是`Strings.`

另一方面，每次修改或引入新的特定于地区的 java 类都需要重新编译应用程序，而属性文件无需任何额外的工作就可以扩展。

## 2。使用资源包

我们已经知道如何定义资源包，所以我们准备使用它。

让我们来看看这段简短的代码:

```
Locale locale = new Locale("pl", "PL");
ResourceBundle exampleBundle = ResourceBundle.getBundle("package.ExampleResource", locale);

assertEquals(exampleBundle.getString("currency"), "polish zloty");
assertEquals(exampleBundle.getObject("toUsdRate"), new BigDecimal("3.401")); 
assertArrayEquals(exampleBundle.getStringArray("cities"), new String[]{"Warsaw", "Cracow"});
```

首先，我们可以定义我们的`Locale`，除非我们不想使用默认的。

之后，我们调用一个`ResourceBundle`的静态工厂方法。我们需要将包名及其包/目录和地区作为参数传递给**。**

还有一个工厂方法，如果默认的语言环境可以的话，它只需要一个包名。一旦有了对象，我们就可以通过它们的键来检索值。

另外，这个例子表明我们可以使用`getString(String key)`、`getObject(String key),` 和`getStringArray(String key)` 来获得我们想要的值。

## 3。选择合适的捆绑包资源

**如果我们想使用捆绑包资源，了解`Java`如何选择捆绑包文件是很重要的。**

让我们想象一下，我们使用一个需要波兰语标签的应用程序，但是您的默认`JVM`地区是`Locale.US`。

开始时，应用程序会在类路径中寻找适合您所要求的语言环境的文件。它从最具体的名称开始，即包含平台、国家和语言的名称。

然后，它去更一般。如果没有匹配，那么它将返回默认的语言环境，这次不进行平台检查。

如果不匹配，它将尝试读取默认包。当我们查看所选文件名的顺序时，一切都应该清楚了:

*   `Label_pl_PL_UNIX`
*   `Label_pl_PL`
*   `Label_pl`
*   `Label_en_US`
*   `Label_en`
*   `Label`

我们应该记住，每个名称都代表`.java`和`.properties`文件，但是前者优先于后者。当没有合适的文件时，会抛出一个`MissingResourceException`。

## 4。继承

资源包概念的另一个优点是属性继承。这意味着包含在不太具体的文件中的键-值对被那些在继承树中较高的文件继承。

假设我们有三个属性文件:

```
#resource.properties
cancelButton = cancel

#resource_pl.properties
continueButton = dalej

#resource_pl_PL.properties
backButton = cofnij
```

为`Locale(“pl”, “PL”)`检索的资源包将在结果中返回所有三个键/值。值得一提的是，就属性继承而言，**没有回退到默认的区域设置包**。

而且，**和`PropertyResourceBundles`不在一个层次上。**

因此，如果在类路径中找到了属性文件，那么键值对只能从属性文件中继承。同样的规则也适用于 Java 文件。

## 5。定制

上面我们所学的都是关于`ResourceBundle`的默认实现。然而，我们有一种方法可以改变它的行为。

我们通过扩展`ResourceBoundle.Control`并覆盖它的方法来做到这一点。

例如，我们可以改变在缓存中保存值的时间，或者确定缓存应该重新加载的条件。

为了更好地理解，让我们准备一个简短的方法作为示例:

```
public class ExampleControl extends ResourceBundle.Control {

    @Override
    public List<Locale> getCandidateLocales(String s, Locale locale) {
        return Arrays.asList(new Locale("pl", "PL"));
    }
}
```

该方法的目的是改变在类路径中选择文件的方式。正如我们所见，`ExampleControl`将只返回波兰语`Locale`，不管默认的或定义的`Locale`是什么。

## 6。UTF-8

由于仍有许多应用程序使用`JDK 8`或更老的版本，值得知道的是`Java` 9 `ListResourceBundles`之前的**比`PropertyResourceBundles`多了一个优势。由于 Java 文件可以存储字符串对象，所以它们可以保存任何受`UTF-16`编码支持的字符。**

相反，`PropertyResourceBundle`默认使用`ISO 8859-1`编码加载文件，这比`UTF-8`的字符少(这给我们的波兰语示例带来了问题)。

为了保存超过`UTF-8`的字符，我们可以使用`Native-To-ASCII`转换器—`native2ascii`。它通过将所有不符合 ISO 8859-1 的字符编码成`\uxxxx`符号来转换它们。

下面是一个命令示例:

```
native2ascii -encoding UTF-8 utf8.properties nonUtf8.properties
```

让我们看看编码改变前后的属性:

```
#Before
polishHello=cześć

#After
polishHello=cze\u015b\u0107
```

幸运的是，这种不便在 Java 9 中不再存在。`JVM`以`UTF-8`编码读取属性文件，使用非拉丁字符没有问题。

## 7。结论

包含了我们开发多语言应用程序所需的大部分内容。我们介绍的特性使得对不同地区的操作变得非常简单。

我们还避免硬编码值，允许我们通过简单地添加新的`Locale`文件来扩展支持的`Locales`，允许我们的应用程序被平滑地修改和维护。

与往常一样，GitHub 上的[中提供了示例代码。](https://web.archive.org/web/20221208143839/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java)