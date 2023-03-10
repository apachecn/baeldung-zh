# Java 9 java.lang.Module API

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-9-module-api>

## 1。简介

继[Java 9 模块化指南](/web/20220526055931/https://www.baeldung.com/java-9-modularity)之后，在本文中，我们将探索与 Java 平台模块系统一起引入的`java.lang.Module` API。

这个 API 提供了一种以编程方式访问模块的方法，从模块中检索特定的信息，并且通常使用它和它的`Module` `Descriptor`。

## 2。读取模块信息

`Module`类表示命名和未命名的模块。**命名模块有名称，由 Java 虚拟机创建模块层时构建，**使用模块图作为定义。

未命名模块没有名字，每个`ClassLoader.` **都有一个名字。所有不在命名模块中的类型都是与它们的类装入器相关的未命名模块的成员。**

`Module`类有趣的地方在于它公开了允许我们从模块中检索信息的方法，比如模块名、模块类加载器和模块中的包。

让我们来看看如何找出一个模块是命名的还是未命名的。

### 2.1。命名或未命名

使用`isNamed()`方法我们可以识别一个模块是否被命名。

让我们看看如何判断一个给定的类，比如`HashMap`，是否是一个命名模块的一部分，以及如何检索它的名称:

```java
Module javaBaseModule = HashMap.class.getModule();

assertThat(javaBaseModule.isNamed(), is(true));
assertThat(javaBaseModule.getName(), is("java.base"));
```

现在让我们定义一个`Person`类:

```java
public class Person {
    private String name;

    // constructor, getters and setters
}
```

同样，正如我们对`HashMap`类所做的那样，我们可以检查`Person`类是否是命名模块的一部分:

```java
Module module = Person.class.getModule();

assertThat(module.isNamed(), is(false));
assertThat(module.getName(), is(nullValue()));
```

### 2.2。包装

当使用一个模块时，知道模块中哪些包是可用的可能很重要。

让我们看看如何检查给定的包，例如`java.lang.annotation`，是否包含在给定的模块中:

```java
assertTrue(javaBaseModule.getPackages().contains("java.lang.annotation"));
assertFalse(javaBaseModule.getPackages().contains("java.sql"));
```

### 2.3。注释

同样，对于包，**可以使用`getAnnotations()`方法**来检索模块中的注释。

如果命名模块中没有注释，该方法将返回一个空数组。

让我们看看`java.base`模块中有多少注释:

```java
assertThat(javaBaseModule.getAnnotations().length, is(0));
```

当在未命名的模块上调用时，`getAnnotations()`方法将返回一个空数组。

### 2.4 版。类装入器

感谢`Module`类中可用的`getClassLoader()`方法，我们可以检索给定模块的`ClassLoader`:

```java
assertThat(
  module.getClassLoader().getClass().getName(), 
  is("jdk.internal.loader.ClassLoaders$AppClassLoader")
);
```

### 2.5。层

另一个可以从模块中提取的有价值的信息是`ModuleLayer`，它表示 Java 虚拟机中的模块层。

模块层通知 JVM 可以从模块加载的类。通过这种方式，JVM 确切地知道每个类是哪个模块的成员。

一个`ModuleLayer`包含与其配置、父层和该层中可用的模块集相关的信息。

让我们看看如何检索给定模块的`ModuleLayer`:

```java
ModuleLayer javaBaseModuleLayer = javaBaseModule.getLayer();
```

一旦我们检索到`ModuleLayer`，我们就可以访问它的信息:

```java
assertTrue(javaBaseModuleLayer.configuration().findModule("java.base").isPresent());
```

一个特例是启动层，它是在 Java 虚拟机启动时创建的。引导层是唯一包含`java.base`模块的层。

## 3。`ModuleDescriptor`对付

**A `ModuleDescriptor`描述了一个命名的模块，并定义了获取其每个组件的方法。**

对于多个并发线程来说，对象是不可变且安全的。

让我们从如何检索一个`ModuleDescriptor.`开始

### 3.1。检索一个`ModuleDescriptor`

因为`ModuleDescriptor`与`Module`紧密相连，所以可以直接从`Module:`中检索它

```java
ModuleDescriptor moduleDescriptor = javaBaseModule.getDescriptor();
```

### 3.2。创造一个`ModuleDescriptor`

**也可以使用`ModuleDescriptor.Builder`类**或者通过读取模块声明的二进制形式`module-info.class`来创建模块描述符。

让我们看看如何使用`ModuleDescriptor.Builder` API 创建模块描述符:

```java
ModuleDescriptor.Builder moduleBuilder = ModuleDescriptor
  .newModule("baeldung.base");

ModuleDescriptor moduleDescriptor = moduleBuilder.build();

assertThat(moduleDescriptor.name(), is("baeldung.base"));
```

这样，我们创建了一个普通模块，但是如果我们想要创建一个开放模块或者自动模块，我们可以分别使用`newOpenModule()`或者`newAutomaticModule()`方法。

### 3.3。对模块进行分类

模块描述符描述一个正常的、开放的或自动的模块。

得益于`ModuleDescriptor`中可用的方法，可以识别模块的类型:

```java
ModuleDescriptor moduleDescriptor = javaBaseModule.getDescriptor();

assertFalse(moduleDescriptor.isAutomatic());
assertFalse(moduleDescriptor.isOpen());
```

### 3.4。检索需要

使用模块描述符，可以检索代表模块依赖关系的一组`Requires`。

使用`requires()`方法可以做到这一点:

```java
Set<Requires> javaBaseRequires = javaBaseModule.getDescriptor().requires();
Set<Requires> javaSqlRequires = javaSqlModule.getDescriptor().requires();

Set<String> javaSqlRequiresNames = javaSqlRequires.stream()
  .map(Requires::name)
  .collect(Collectors.toSet());

assertThat(javaBaseRequires, empty());
assertThat(javaSqlRequiresNames, hasItems("java.base", "java.xml", "java.logging")); 
```

**所有模块，除了`java`** 。 **`base`，有`java`** 。 **`base`模块作为**的依赖。

但是，如果该模块是自动模块，那么除了`java.base`之外，依赖关系集将为空。

### 3.5。检索提供了

使用`provides()`方法，可以检索模块提供的服务列表:

```java
Set<Provides> javaBaseProvides = javaBaseModule.getDescriptor().provides();
Set<Provides> javaSqlProvides = javaSqlModule.getDescriptor().provides();

Set<String> javaBaseProvidesService = javaBaseProvides.stream()
  .map(Provides::service)
  .collect(Collectors.toSet());

assertThat(javaBaseProvidesService, hasItem("java.nio.file.spi.FileSystemProvider"));
assertThat(javaSqlProvides, empty());
```

### 3.6。检索出口

使用`exports()`方法，我们可以发现模块是否导出包，特别是哪些包:

```java
Set<Exports> javaSqlExports = javaSqlModule.getDescriptor().exports();

Set<String> javaSqlExportsSource = javaSqlExports.stream()
  .map(Exports::source)
  .collect(Collectors.toSet());

assertThat(javaSqlExportsSource, hasItems("java.sql", "javax.sql"));
```

作为特例，如果模块是自动模块，则导出的包集将为空。

### 3.7。检索用途

使用`uses()`方法，可以检索模块的服务依赖集:

```java
Set<String> javaSqlUses = javaSqlModule.getDescriptor().uses();

assertThat(javaSqlUses, hasItem("java.sql.Driver"));
```

在模块是自动模块的情况下，依赖集将是空的。

### 3.8。检索打开

每当我们想要检索一个模块的打开包的列表时，我们可以使用`opens()`方法:

```java
Set<Opens> javaBaseUses = javaBaseModule.getDescriptor().opens();
Set<Opens> javaSqlUses = javaSqlModule.getDescriptor().opens();

assertThat(javaBaseUses, empty());
assertThat(javaSqlUses, empty());
```

如果模块是开放的或自动的，集合将为空。

## 4。处理模块

使用`Module` API，除了从模块中读取信息，我们还可以更新模块定义。

### 4.1。添加出口

让我们看看如何更新一个模块，从一个给定的模块导出给定的包:

```java
Module updatedModule = module.addExports(
  "com.baeldung.java9.modules", javaSqlModule);

assertTrue(updatedModule.isExported("com.baeldung.java9.modules"));
```

只有当调用方的模块是代码所属的模块时，才能做到这一点。

顺便提一下，如果模块已经导出了包，或者模块是一个打开的模块，则没有任何影响。

### 4.2。添加读数

当我们想要更新一个模块来读取一个给定的模块时，我们可以使用`addReads()`方法:

```java
Module updatedModule = module.addReads(javaSqlModule);

assertTrue(updatedModule.canRead(javaSqlModule));
```

如果我们添加模块本身，这个方法什么也不做，因为所有的模块都自己读取。

同样，如果该模块是一个未命名的模块，或者该模块已经读取了另一个模块，则该方法不执行任何操作。

### 4.3。添加打开

当我们想要更新一个已经打开包的模块到至少调用方模块时，我们可以使用`addOpens()`打开包到另一个模块:

```java
Module updatedModule = module.addOpens(
  "com.baeldung.java9.modules", javaSqlModule);

assertTrue(updatedModule.isOpen("com.baeldung.java9.modules", javaSqlModule));
```

如果包已经对给定的模块打开，则此方法无效。

### 4.4。添加用途

每当我们想要更新一个添加服务依赖的模块时，方法`addUses()`就是我们的选择:

```java
Module updatedModule = module.addUses(Driver.class);

assertTrue(updatedModule.canUse(Driver.class));
```

在未命名模块或自动模块上调用时，此方法不执行任何操作。

## 5。结论

在本文中，我们探索了`java.lang.Module` API 的使用，我们学习了如何检索模块的信息，如何使用`ModuleDescriptor`来访问关于模块的附加信息，以及如何操作它。

和往常一样，本文中的所有代码示例都可以在 GitHub 上找到[。](https://web.archive.org/web/20220526055931/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-9-jigsaw)