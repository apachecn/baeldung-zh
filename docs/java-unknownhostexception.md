# java.net.UnknownHostException:服务器的主机名无效

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-unknownhostexception>

## 1.介绍

在本教程中，我们将通过一个例子来了解`UnknownHostException`的成因。我们还将讨论防止和处理异常的可能方法。

## 2.什么时候抛出异常？

**`UnknownHostException`表示主机名的 IP 地址无法确定。** **这可能是因为主机名:**中的一个错别字

```java
String hostname = "http://locaihost";
URL url = new URL(hostname);
HttpURLConnection con = (HttpURLConnection) url.openConnection();
con.getResponseCode();
```

上面的代码抛出了一个`UnknownHostException`，因为拼错的`locaihost`没有指向任何 IP 地址。

**`UnknownHostException`的另一个可能原因是 DNS 传播延迟或 DNS 配置错误。**

一个新的 DNS 条目可能需要 48 小时才能传播到整个互联网。

## 3.如何预防？

首先防止异常发生比事后处理要好。防止异常的几个技巧是:

1.  **再次检查主机名:**确保没有错别字，并修剪所有空格。
2.  **检查系统的 DNS 设置:**确保 DNS 服务器已启动且可访问，如果主机名是新的，则等待 DNS 服务器跟上。

## 4.怎么处理？

`UnknownHostException`扩展`IOException`，为[检查异常](/web/20221206072157/https://www.baeldung.com/java-checked-unchecked-exceptions#checked)。类似于任何其他被检查的异常，我们必须要么抛出它，要么用一个`try-catch`块包围它。

让我们来处理例子中的异常:

```java
try {
    con.getResponseCode();
} catch (UnknownHostException e) {
    con.disconnect();
}
```

**当`UnknownHostException`发生时，关闭连接是一个很好的做法。**大量浪费的开放连接会导致应用程序耗尽内存。

## 5.结论

在本文中，我们了解了`UnknownHostException`的原因、预防方法以及处理方法。

和往常一样，代码可以在 Github 的[上获得。](https://web.archive.org/web/20221206072157/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-exceptions-2)