# Spring Boot——使用彩色创业横幅

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-color-banner>

## 1.介绍

Spring Boot 的一个可爱特征是它的创业公司 T2 的横幅 T3。多年来，Spring Boot 已经发展到支持各种类型的横幅。例如，在 [Spring Boot 1.3](https://web.archive.org/web/20221226055109/https://github.com/spring-projects/spring-boot/wiki/spring-boot-1.3-release-notes#ansi-color-bannertxt-files) 中为横幅添加了文本和背景色支持。

在这个快速教程中，我们将看看 Spring Boot 的颜色横幅支持以及如何使用它。

## 2.更改背景颜色

为了给 Spring Boot 的横幅,**添加背景色，我们只需要使用`AnsiBackground`类**给`banner.txt`的行加上想要的颜色代码。

例如，让我们创建一个`banner.txt`文件，使整个背景为红色:

```java
${AnsiBackground.RED}
  ___         _   _      _ 
 / __|  ___  | | (_)  __| |
 \__ \ / _ \ | | | | / _` |
 |___/ \___/ |_| |_| \__,_|
${AnsiBackground.DEFAULT}
```

[![spring boot color banner solid background](img/442c32c6279b0b9d265ebe476779103b.png)](/web/20221226055109/https://www.baeldung.com/wp-content/uploads/2019/12/spring-boot-color-banner-solid-background.jpg)

事实上，**我们可以在一个横幅**中使用任意多的背景颜色。

例如，我们可以将每一行设置为它自己的背景颜色。我们只需在每一行前面加上想要的颜色:

```java
${AnsiBackground.RED}    ____             _             __
${AnsiBackground.BLUE}   / __ \  ____ _   (_)   ____    / /_   ____  _      __
${AnsiBackground.YELLOW}  / /_/ / / __ `/  / /   / __ \  / __ \ / __ \| | /| / /
${AnsiBackground.GREEN} / _, _/ / /_/ /  / /   / / / / / /_/ // /_/ /| |/ |/ /
${AnsiBackground.MAGENTA}/_/ |_|  \__,_/  /_/   /_/ /_/ /_.___/ \____/ |__/|__/
${AnsiBackground.DEFAULT} 
```

[![spring boot color banner rainbow background](img/8be7a143f8295f05c4ba49e3cb67bff1.png)](/web/20221226055109/https://www.baeldung.com/wp-content/uploads/2019/12/spring-boot-color-banner-rainbow-background.jpg)

重要的是要记住，我们所有的应用程序日志都将使用在`banner.txt`中指定的最后一种背景颜色。因此，**最好总是用默认颜色**来结束`banner.txt`文件。

## 3.更改文本颜色

要改变文本的颜色，我们可以使用`AnsiColor`类。就像`AnsiBackground`类一样，它有预定义的颜色常量供我们选择。

我们只需给每组字符加上所需颜色的前缀:

```java
${AnsiColor.RED}.------.${AnsiColor.BLACK}.------.
${AnsiColor.RED}|A.--. |${AnsiColor.BLACK}|K.--. |
${AnsiColor.RED}| (\/) |${AnsiColor.BLACK}| (\/) |
${AnsiColor.RED}| :\/: |${AnsiColor.BLACK}| :\/: |
${AnsiColor.RED}| '--'A|${AnsiColor.BLACK}| '--'K|
${AnsiColor.RED}`------'${AnsiColor.BLACK}`------'
${AnsiColor.DEFAULT}
```

[![spring boot color text](img/a285c4e34098dd4289abad0e6efe1ff4.png)](/web/20221226055109/https://www.baeldung.com/wp-content/uploads/2019/12/spring-boot-color-text.jpg)

与背景颜色一样，**横幅的最后一行总是将颜色重置为默认颜色**，这一点很重要。

## 4.ANSI 8 位颜色

Spring Boot 2.2 的新特性之一是支持 [ANSI 8 位颜色](https://web.archive.org/web/20221226055109/https://en.wikipedia.org/wiki/ANSI_escape_code#8-bit)。**不再局限于几种预定义的颜色，我们可以使用 256 种颜色来指定文本和背景颜色。**

为了利用新的颜色，`AnsiColor`和`AnsiBackground`属性现在都接受一个数值，而不是颜色名称:

```java
${AnsiColor.1}${AnsiBackground.233}  ______  __________ .___ ___________
${AnsiBackground.235} /  __  \ \______   \|   |\__    ___/
${AnsiBackground.237} >      <  |    |  _/|   |  |    |
${AnsiBackground.239}/   --   \ |    |   \|   |  |    |
${AnsiBackground.241}\______  / |______  /|___|  |____|
${AnsiBackground.243}       \/         \/
${AnsiBackground.DEFAULT}${AnsiColor.DEFAULT}
```

[![spring boot color banner 8 bit ansi](img/f579e465e0398a83c837c2e8280889cd.png)](/web/20221226055109/https://www.baeldung.com/wp-content/uploads/2019/12/spring-boot-color-banner-8-bit-ansi.jpg)

注意，我们可以任意混合文本和背景属性。我们甚至可以在同一横幅中混合使用新的 8 位颜色代码和旧的颜色常量。

## 5.结论

在本文中，我们看到了如何改变 Spring Boot 横幅的文本和背景颜色。

我们还看到了新版本的 Spring Boot 是如何支持 ANSI 8 位色码的。