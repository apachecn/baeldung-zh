# Java 属性入门

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-properties>

## 1。概述

大多数 Java 应用程序在某些时候需要使用属性，通常是在编译代码之外将简单的参数存储为键值对。

所以这种语言对属性有一流的支持——`java.util.Properties`——一个为处理这种类型的配置文件而设计的实用程序类。

这就是我们将在本文中关注的内容。

## 2。加载属性

### 2.1。从属性文件

让我们从一个从属性文件加载键值对的例子开始；我们正在加载类路径中的两个文件:

`app.properties:`

```java
version=1.0
name=TestApp
date=2016-11-12
```

和`catalog`:

```java
c1=files
c2=images
c3=videos
```

注意，虽然属性文件推荐使用后缀“`.properties`”，但这并不是必须的。

我们现在可以非常简单地将它们加载到一个`Properties`实例中:

```java
String rootPath = Thread.currentThread().getContextClassLoader().getResource("").getPath();
String appConfigPath = rootPath + "app.properties";
String catalogConfigPath = rootPath + "catalog";

Properties appProps = new Properties();
appProps.load(new FileInputStream(appConfigPath));

Properties catalogProps = new Properties();
catalogProps.load(new FileInputStream(catalogConfigPath));

String appVersion = appProps.getProperty("version");
assertEquals("1.0", appVersion);

assertEquals("files", catalogProps.getProperty("c1"));
```

只要文件的内容符合属性文件格式要求，它就可以被`Properties`类正确解析。以下是[属性文件格式](https://web.archive.org/web/20221127172540/https://en.wikipedia.org/wiki/.properties)的更多细节。

### 2.2。从 XML 文件加载

除了属性文件，`Properties`类还可以加载符合特定 DTD 规范的 XML 文件。

下面是一个从 XML 文件加载键值对的例子—`icons.xml`:

```java
<?xml version="1.0" encoding="utf-8" ?>
<!DOCTYPE properties SYSTEM "http://java.sun.com/dtd/properties.dtd">
<properties>
    <comment>xml example</comment>
    <entry key="fileIcon">icon1.jpg</entry>
    <entry key="imageIcon">icon2.jpg</entry>
    <entry key="videoIcon">icon3.jpg</entry>
</properties>
```

现在，让我们加载它:

```java
String rootPath = Thread.currentThread().getContextClassLoader().getResource("").getPath();
String iconConfigPath = rootPath + "icons.xml";
Properties iconProps = new Properties();
iconProps.loadFromXML(new FileInputStream(iconConfigPath));

assertEquals("icon1.jpg", iconProps.getProperty("fileIcon"));
```

## 3。获取属性

我们可以使用`getProperty(String key)`和`getProperty(String key, String defaultValue)`通过它的键来获取值。

如果键值对存在，这两个方法都将返回相应的值。但是如果没有这样的键值对，前者将返回 null，后者将返回`defaultValue`来代替。

示例代码:

```java
String appVersion = appProps.getProperty("version");
String appName = appProps.getProperty("name", "defaultName");
String appGroup = appProps.getProperty("group", "baeldung");
String appDownloadAddr = appProps.getProperty("downloadAddr");

assertEquals("1.0", appVersion);
assertEquals("TestApp", appName);
assertEquals("baeldung", appGroup);
assertNull(appDownloadAddr);
```

注意，虽然`Properties`类从`Hashtable`类继承了`get()`方法，但我不建议你用它来获取值。因为它的`get()`方法将返回一个只能转换为`String`的`Object`值，而`getProperty()`方法已经为您正确处理了原始的`Object`值。

下面的代码将抛出一个`Exception`:

```java
float appVerFloat = (float) appProps.get("version");
```

## 4。设置属性

我们可以使用`setProperty()`方法来更新一个已有的键-值对或者添加一个新的键-值对。

示例代码:

```java
appProps.setProperty("name", "NewAppName"); // update an old value
appProps.setProperty("downloadAddr", "www.baeldung.com/downloads"); // add new key-value pair

String newAppName = appProps.getProperty("name");
assertEquals("NewAppName", newAppName);

String newAppDownloadAddr = appProps.getProperty("downloadAddr");
assertEquals("www.baeldung.com/downloads", newAppDownloadAddr);
```

注意，虽然`Properties`类从`Hashtable`类继承了`put()`方法和`putAll()`方法，但我不建议你使用它们，原因和使用`get()`方法一样:只有`String`值可以在`Properties`中使用。

下面的代码不会如你所愿，当你用`getProperty()`获取它的值时，它会返回`null`:

```java
appProps.put("version", 2);
```

## 5。移除属性

如果你想删除一个键值对，你可以使用`remove()`方法。

示例代码:

```java
String versionBeforeRemoval = appProps.getProperty("version");
assertEquals("1.0", versionBeforeRemoval);

appProps.remove("version");    
String versionAfterRemoval = appProps.getProperty("version");
assertNull(versionAfterRemoval);
```

## 6。商店

### 6.1。存储到属性文件

`Properties`类提供了一个`store()`方法来输出键值对。

示例代码:

```java
String newAppConfigPropertiesFile = rootPath + "newApp.properties";
appProps.store(new FileWriter(newAppConfigPropertiesFile), "store to properties file");
```

第二个参数用于注释。如果你不想写任何注释，就用 null。

### 6.2。存储到 XML 文件

`Properties`类还提供了一个`storeToXML()`方法，以 XML 格式输出键值对。

示例代码:

```java
String newAppConfigXmlFile = rootPath + "newApp.xml";
appProps.storeToXML(new FileOutputStream(newAppConfigXmlFile), "store to xml file");
```

第二个参数与`store()`方法中的参数相同。

## 7。其他常见操作

`Properties`类还提供了一些其他的方法来操作这些属性。

示例代码:

```java
appProps.list(System.out); // list all key-value pairs

Enumeration<Object> valueEnumeration = appProps.elements();
while (valueEnumeration.hasMoreElements()) {
    System.out.println(valueEnumeration.nextElement());
}

Enumeration<Object> keyEnumeration = appProps.keys();
while (keyEnumeration.hasMoreElements()) {
    System.out.println(keyEnumeration.nextElement());
}

int size = appProps.size();
assertEquals(3, size);
```

## 8。默认属性列表

一个`Properties`对象可以包含另一个`Properties`对象作为其默认属性列表。如果在原始属性列表中找不到属性键，将搜索默认属性列表。

除了"`app.properties`"之外，我们的类路径中还有另一个文件"`default.properties`":

默认属性:

```java
site=www.google.com
name=DefaultAppName
topic=Properties
category=core-java
```

示例代码:

```java
String rootPath = Thread.currentThread().getContextClassLoader().getResource("").getPath();

String defaultConfigPath = rootPath + "default.properties";
Properties defaultProps = new Properties();
defaultProps.load(new FileInputStream(defaultConfigPath));

String appConfigPath = rootPath + "app.properties";
Properties appProps = new Properties(defaultProps);
appProps.load(new FileInputStream(appConfigPath));

assertEquals("1.0", appVersion);
assertEquals("TestApp", appName);
assertEquals("www.google.com", defaultSite);
```

## 9.属性和编码

默认情况下，属性文件应该是 ISO-8859-1 (Latin-1)编码的，所以通常不应该使用带有 ISO-8859-1 之外的字符的属性。

如有必要，我们可以借助 JDK native2ascii 工具或文件显式编码等工具来解决这一限制。

对于 XML 文件，`loadFromXML()`方法和`storeToXML()`方法默认使用 UTF-8 字符编码。

然而，当读取不同编码的 XML 文件时，我们可以在`DOCTYPE`声明中指定；编写也足够灵活——我们可以在`storeToXML()` API 的第三个参数中指定编码。

## 10.结论

在本文中，我们讨论了基本的`Properties`类用法，包括如何使用`Properties`加载和存储属性和 XML 格式的键-值对，如何操作`Properties`对象中的键-值对，比如检索值、更新值、获取其大小，以及如何为`Properties`对象使用默认列表。

该示例的完整源代码可以在这个 [GitHub 项目](https://web.archive.org/web/20221127172540/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java)中找到。