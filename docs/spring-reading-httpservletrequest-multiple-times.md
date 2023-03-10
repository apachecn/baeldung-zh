# 在春天多次读取 HttpServletRequest

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-reading-httpservletrequest-multiple-times>

## 1.介绍

在本教程中，我们将学习如何使用 Spring 多次读取来自`HttpServletRequest`的主体。

`HttpServletRequest`是一个接口，它公开了读取主体的`getInputStream()`方法。默认情况下，**来自这个`InputStream`的数据只能被**读取一次。

## 2。Maven 依赖关系

我们首先需要的是合适的 [`spring-webmvc`](https://web.archive.org/web/20220626193107/https://search.maven.org/artifact/org.springframework/spring-webmvc/5.2.0.RELEASE/jar) 和 [`javax.servlet`](https://web.archive.org/web/20220626193107/https://search.maven.org/artifact/javax.servlet/javax.servlet-api/4.0.1/jar) 依赖关系:

```java
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-webmvc</artifactId>
    <version>5.2.0.RELEASE</version>
</dependency>
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>javax.servlet-api</artifactId>
    <version>4.0.1</version>
</dependency> 
```

此外，因为我们使用了`application/json` 内容类型，所以需要 [`jackson-databind`](https://web.archive.org/web/20220626193107/https://search.maven.org/artifact/com.fasterxml.jackson.core/jackson-databind/2.10.0/bundle) 依赖关系:

```java
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.10.0</version>
</dependency>
```

Spring 使用这个库与 JSON 相互转换。

## 3.春天的`ContentCachingRequestWrapper`

Spring 提供了一个`ContentCachingRequestWrapper` 类。这个类提供了一个方法，`getContentAsByteArray()` 来多次读取正文`.`

但是这个类有一个限制:**我们不能使用`getInputStream()`和`getReader()`方法多次读取主体。**

这个类通过使用`InputStream`来缓存请求体。如果我们读取其中一个过滤器中的`InputStream`，那么过滤器链中的其他后续过滤器就不能再读取它了。由于这一限制，该类并不适用于所有情况。

为了克服这个限制，现在让我们来看看一个更通用的解决方案。

## 4.延伸`HttpServletRequest`

让我们创建一个新的**类——`CachedBodyHttpServletRequest –` ，它扩展了`HttpServletRequestWrapper`** `.` 这样，我们不需要覆盖`HttpServletRequest` 接口`.`的所有抽象方法

`HttpServletRequestWrapper`类有两个抽象方法`getInputStream()`和`getReader()`。我们将重写这两个方法，并创建一个新的构造函数。

### 4.1.构造函数

首先，让我们创建一个构造函数。在其中，我们将从实际的`InputStream` 中读取主体，并将其存储在一个`byte[]`对象中:

```java
public class CachedBodyHttpServletRequest extends HttpServletRequestWrapper {

    private byte[] cachedBody;

    public CachedBodyHttpServletRequest(HttpServletRequest request) throws IOException {
        super(request);
        InputStream requestInputStream = request.getInputStream();
        this.cachedBody = StreamUtils.copyToByteArray(requestInputStream);
    }
}
```

因此，我们将能够多次读取正文。

### 4.2.`getInputStream()`

接下来，让我们重写`getInputStream()`方法。我们将使用这个方法来读取原始体并将它转换成一个对象。

在这个方法中，我们将**创建并返回一个`CachedBodyServletInputStream` 类的新对象**(一个`ServletInputStream)`的实现:

```java
@Override
public ServletInputStream getInputStream() throws IOException {
    return new CachedBodyServletInputStream(this.cachedBody);
}
```

### 4.3.`getReader()`

然后，我们将覆盖`getReader()`方法。这个方法返回一个`BufferedReader` 对象:

```java
@Override
public BufferedReader getReader() throws IOException {
    ByteArrayInputStream byteArrayInputStream = new ByteArrayInputStream(this.cachedBody);
    return new BufferedReader(new InputStreamReader(byteArrayInputStream));
}
```

## 5.`ServletInputStream`的实施

让我们创建一个**类——`CachedBodyServletInputStream`——它将实现`ServletInputStream`** 。在这个类中，我们将创建一个新的构造函数并覆盖`isFinished()`、`isReady()`和`read()`方法。

### 5.1.构造函数

首先，让我们创建一个新的接受字节数组的构造函数。

在其中，我们将使用该字节数组创建一个新的**实例。**之后，我们将它赋给全局变量`cachedBodyInputStream:`

```java
public class CachedBodyServletInputStream extends ServletInputStream {

    private InputStream cachedBodyInputStream;

    public CachedBodyServletInputStream(byte[] cachedBody) {
        this.cachedBodyInputStream = new ByteArrayInputStream(cachedBody);
    }
}
```

### 5.2.`read()`

然后，我们将覆盖`read()` 方法`.` 。在这个方法中，我们将调用`ByteArrayInputStream#read:`

```java
@Override
public int read() throws IOException {
    return cachedBodyInputStream.read();
}
```

### 5.3.`isFinished()`

然后，我们将覆盖`isFinished()`方法。该方法指示`InputStream`是否有更多数据要读取。当没有字节可供读取时，它返回`true`:

```java
@Override
public boolean isFinished() {
    return cachedBody.available() == 0;
}
```

### 5.4.`isReady()`

类似地，我们将覆盖`isReady()`方法。该方法指示`InputStream`是否准备好读取。

因为我们已经将`InputStream`复制到一个字节数组中，所以我们将返回`true`来表示它总是可用的:

```java
@Override
public boolean isReady() {
    return true;
}
```

## 6.该过滤器

最后，让我们创建一个新的过滤器来利用`CachedBodyHttpServletRequest` 类。这里我们将扩展 Spring 的`OncePerRequestFilter` 类。这个类有一个抽象方法`doFilterInternal()`。

在这个方法中，我们将从实际的请求对象中**创建一个`CachedBodyHttpServletRequest` 类的对象:**

```java
CachedBodyHttpServletRequest cachedBodyHttpServletRequest =
  new CachedBodyHttpServletRequest(request);
```

然后我们将**把这个新的请求包装器对象传递给过滤器链**。因此，对`getInputStream`()方法的所有后续调用都将调用被覆盖的方法:

```java
filterChain.doFilter(cachedContentHttpServletRequest, response);
```

## 7 .**。结论**

在本教程中，我们快速浏览了`ContentCachingRequestWrapper`类。我们也看到了它的局限性。

然后，我们创建了一个`HttpServletRequestWrapper` 类的新实现。我们重写了`getInputStream()`方法来返回一个`ServletInputStream `类的对象。

最后，我们创建了一个新的过滤器，将请求包装器对象传递给过滤器链。因此，我们能够多次读取请求。

这些例子的完整源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220626193107/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-mvc-basics-3)