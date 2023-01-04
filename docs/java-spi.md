# Java 服务提供商接口

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-spi>

## 1。概述

Java 6 引入了一个特性，用于发现和加载与给定接口匹配的实现:服务提供者接口(SPI)。

在本教程中，我们将介绍 Java SPI 的组件，并展示如何将它应用到实际用例中。

## 2。Java SPI 的术语和定义

Java SPI 定义了四个主要组件

### 2.1.服务

一组众所周知的编程接口和类，提供对某些特定应用程序功能或特性的访问。

### 2.2.服务提供商接口

充当服务的代理或端点的接口或抽象类。

如果服务是一个接口，那么它与服务提供者接口相同。

在 Java 生态系统中，服务和 SPI 一起被称为 API。

### 2.3.互联网服务商

SPI 的具体实现。服务提供者包含一个或多个实现或扩展服务类型的具体类。

服务提供者是通过我们放在资源目录`META-INF/services`中的提供者配置文件来配置和标识的。文件名是 SPI 的全限定名，其内容是 SPI 实现的全限定名。

服务提供者以扩展的形式安装，一个 jar 文件放在应用程序类路径、Java 扩展类路径或用户定义的类路径中。

### 2.4.服务加载器

SPI 的核心是 [`ServiceLoader`](https://web.archive.org/web/20221226084713/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/ServiceLoader.html) 类。这有助于发现和延迟加载实现。它使用上下文类路径来定位提供者实现，并将它们放在内部缓存中。

## 3。Java 生态系统中的 SPI 示例

Java 提供了许多 SPI。以下是服务提供者接口及其提供的服务的一些示例:

*   [`CurrencyNameProvider:`](https://web.archive.org/web/20221226084713/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/spi/CurrencyNameProvider.html) 为`Currency`类提供本地化的货币符号。
*   `[LocaleNameProvider](https://web.archive.org/web/20221226084713/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/spi/LocaleNameProvider.html):`为`Locale`类提供本地化名称。
*   [`TimeZoneNameProvider:`](https://web.archive.org/web/20221226084713/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/spi/TimeZoneNameProvider.html) 为`TimeZone`类提供本地化的时区名称。
*   `[DateFormatProvider](https://web.archive.org/web/20221226084713/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/text/spi/DateFormatProvider.html):`为指定的语言环境提供日期和时间格式。
*   `[NumberFormatProvider](https://web.archive.org/web/20221226084713/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/text/spi/NumberFormatProvider.html):`为`NumberFormat`类提供货币、整数和百分比值。
*   [`Driver:`](https://web.archive.org/web/20221226084713/https://docs.oracle.com/en/java/javase/11/docs/api/java.sql/java/sql/Driver.html) 从 4.0 版本开始，JDBC API 支持 SPI 模式。旧版本使用 [`Class.forName()`](https://web.archive.org/web/20221226084713/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Class.html#forName(java.lang.String)) 的方法来加载驱动程序。
*   [`PersistenceProvider:`](https://web.archive.org/web/20221226084713/https://docs.oracle.com/javaee/7/api/javax/persistence/spi/PersistenceProvider.html) 提供了 JPA API 的实现。
*   `[JsonProvider:](https://web.archive.org/web/20221226084713/https://docs.oracle.com/javaee/7/api/javax/json/spi/JsonProvider.html)`提供 JSON 处理对象。
*   `[JsonbProvider:](https://web.archive.org/web/20221226084713/https://javaee.github.io/javaee-spec/javadocs/javax/json/bind/spi/JsonbProvider.html)`提供 JSON 绑定对象。
*   [`Extension:`](https://web.archive.org/web/20221226084713/https://docs.oracle.com/javaee/7/api/javax/enterprise/inject/spi/Extension.html) 为 CDI 容器提供了扩展。
*   [`ConfigSourceProvider` :](https://web.archive.org/web/20221226084713/https://openliberty.io/docs/20.0.0.7/reference/javadoc/microprofile-1.2-javadoc.html#package=org/eclipse/microprofile/config/spi/package-frame.html&class=org/eclipse/microprofile/config/spi/ConfigSourceProvider.html) 提供检索配置属性的来源。

## 4。展示:一个货币汇率应用程序

现在我们已经了解了基础知识，让我们来描述设置汇率应用程序所需的步骤。

为了突出这些步骤，我们需要使用至少三个项目:`exchange-rate-api`、 `exchange-rate-impl,`和`exchange-rate-app.`

**第 4.1 小节。，我们将在第 4.2 小节中通过模块`exchange-rate-api,`讲述`Service`、`SPI`和`ServiceLoader`、**。我们将在`the exchange-rate-impl`模块中实现我们的`service **provider**`，最后，我们将通过`exchange-rate-app`模块将 4.3 小节中的所有内容集合在一起。

事实上，我们可以根据需要为`se` `rvice` `provider`提供任意多的模块，并使它们在模块`exchange-rate-app.`的类路径中可用

### 4.1。构建我们的 API

我们首先创建一个名为`exchange-rate-api`的 Maven 项目。名称以术语`api`结尾是个好习惯，但是我们可以随便叫它。

然后，我们创建一个表示汇率货币的模型类:

```java
package com.baeldung.rate.api;

public class Quote {
    private String currency;
    private LocalDate date;
    ...
}
```

然后我们通过创建接口`QuoteManager:`来定义用于检索报价的`Service`

```java
package com.baeldung.rate.api

public interface QuoteManager {
    List<Quote> getQuotes(String baseCurrency, LocalDate date);
}
```

接下来，我们为我们的服务创建一个`SPI`:

```java
package com.baeldung.rate.spi;

public interface ExchangeRateProvider {
    QuoteManager create();
}
```

最后，我们需要创建一个可供客户端代码使用的实用程序类`ExchangeRate.java`。这个类委托给`ServiceLoader`。

首先，我们调用静态工厂方法`load()` 来获得`ServiceLoader:`的实例

```java
ServiceLoader<ExchangeRateProvider> loader = ServiceLoader .load(ExchangeRateProvider.class); 
```

然后我们调用`iterate()`方法来搜索和检索所有可用的实现。

```java
Iterator<ExchangeRateProvider> = loader.iterator(); 
```

搜索结果被缓存，因此我们可以调用 `ServiceLoader.reload()`方法来发现新安装的实现:

```java
Iterator<ExchangeRateProvider> = loader.reload(); 
```

这是我们的实用程序类:

```java
public class ExchangeRate {

    ServiceLoader<ExchangeRateProvider> loader = ServiceLoader
      .load(ExchangeRateProvider.class);

    public Iterator<ExchangeRateProvider> providers(boolean refresh) {
        if (refresh) {
            loader.reload();
        }
        return loader.iterator();
    }
}
```

现在我们有了一个获取所有已安装实现的服务，我们可以在我们的客户端代码中使用所有这些实现来扩展我们的应用程序，或者通过选择一个首选实现来扩展一个应用程序。

**注意，这个实用程序类不一定是`api`项目的一部分。客户端代码可以选择调用`ServiceLoader methods itself.`**

### 4.2。构建服务提供商

现在让我们创建一个名为`exchange-rate-impl`的 Maven 项目，并将 API 依赖项添加到`pom.xml:`中

```java
<dependency>
    <groupId>com.baeldung</groupId>
    <artifactId>exchange-rate-api</artifactId>
    <version>1.0.0-SNAPSHOT</version>
</dependency>
```

然后我们创建一个实现 SPI 的类:

```java
public class YahooFinanceExchangeRateProvider 
  implements ExchangeRateProvider {

    @Override
    public QuoteManager create() {
        return new YahooQuoteManagerImpl();
    }
}
```

这里还有`QuoteManager`接口的实现:

```java
public class YahooQuoteManagerImpl implements QuoteManager {

    @Override
    public List<Quote> getQuotes(String baseCurrency, LocalDate date) {
        // fetch from Yahoo API
    }
}
```

为了被发现，我们创建一个提供者配置文件:

```java
META-INF/services/com.baeldung.rate.spi.ExchangeRateProvider 
```

该文件的内容是 SPI 实现的全限定类名:

```java
com.baeldung.rate.impl.YahooFinanceExchangeRateProvider 
```

### 4.3。将它们放在一起

最后，让我们创建一个名为`exchange-rate-app`的客户端项目，并将依赖项 exchange-rate-api 添加到类路径中:

```java
<dependency>
    <groupId>com.baeldung</groupId>
    <artifactId>exchange-rate-api</artifactId>
    <version>1.0.0-SNAPSHOT</version>
</dependency>
```

此时，我们可以从我们的应用程序中调用 SPI*:*

```java
ExchangeRate.providers().forEach(provider -> ... );
```

### 4.4.运行应用程序

现在让我们专注于构建我们的所有模块:

```java
mvn clean package 
```

然后，我们用`Java`命令运行我们的应用程序，而不考虑提供者:

```java
java -cp ./exchange-rate-api/target/exchange-rate-api-1.0.0-SNAPSHOT.jar:./exchange-rate-app/target/exchange-rate-app-1.0.0-SNAPSHOT.jar com.baeldung.rate.app.MainApp
```

现在，我们将在`java.ext.dirs`扩展中包含我们的提供者，并再次运行应用程序:

```java
java -Djava.ext.dirs=$JAVA_HOME/jre/lib/ext:./exchange-rate-impl/target:./exchange-rate-impl/target/depends -cp ./exchange-rate-api/target/exchange-rate-api-1.0.0-SNAPSHOT.jar:./exchange-rate-app/target/exchange-rate-app-1.0.0-SNAPSHOT.jar com.baeldung.rate.app.MainApp 
```

我们可以看到我们的提供者是加载的。

## 5。结论

既然我们已经通过定义良好的步骤探索了 Java SPI 机制，那么应该很清楚如何使用 Java SPI 来创建易于扩展或替换的模块。

尽管我们的示例使用 Yahoo 汇率服务来展示插入其他现有外部 API 的强大功能，但是生产系统不需要依赖第三方 API 来创建出色的 SPI 应用程序。

像往常一样，代码可以在 Github 上找到[。](https://web.archive.org/web/20221226084713/https://github.com/eugenp/tutorials/tree/master/java-spi)