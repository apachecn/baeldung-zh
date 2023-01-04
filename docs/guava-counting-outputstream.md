# 使用番石榴计数输出流

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/guava-counting-outputstream>

## 1。概述

在本教程中，我们将看看`CountingOutputStream`类以及如何使用它。

这个类可以在像`[Apache Commons](https://web.archive.org/web/20220523235704/https://commons.apache.org/proper/commons-io/javadocs/api-release/org/apache/commons/io/output/CountingOutputStream.html)`或`[Google Guava](https://web.archive.org/web/20220523235704/https://google.github.io/guava/releases/24.0-jre/api/docs/com/google/common/io/CountingOutputStream.html)`这样的流行库中找到。我们将把重点放在 Guava 库中的实现上。

## 2。`CountingOutputStream`

### 2.1。Maven 依赖关系

`CountingOutputStream`是谷歌番石榴包的一部分。

让我们从将依赖项添加到`pom.xml`开始:

```
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>31.0.1-jre</version>
</dependency>
```

依赖关系的最新版本可以在[这里](https://web.archive.org/web/20220523235704/https://search.maven.org/classic/#search|gav|1|g%3A%22com.google.guava%22%20AND%20a%3A%22guava%22)查看。

### 2.2。课程详情

该类扩展了 [`java.io.FilterOutputStream`](https://web.archive.org/web/20220523235704/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/io/FilterOutputStream.html) ，覆盖了`write()`和`close()`方法，并提供了新方法`getCount()`。

构造函数将另一个`OutputStream`对象作为输入参数。**在写入数据的同时，该类计算写入这个`OutputStream`的字节数。**

为了得到计数，我们可以简单地调用`getCount()`来返回当前的字节数:

```
/** Returns the number of bytes written. */
public long getCount() {
    return count;
}
```

## 3。用例

让我们在一个实际用例中使用`CountingOutputStream`。为了举例，我们将把代码放入 JUnit 测试中，使其可执行。

在我们的例子中，我们将向一个`OutputStream`写入数据，并检查我们是否达到了一个`MAX`字节的限制。

一旦达到限制，我们希望通过抛出一个异常来中断执行:

```
public class GuavaCountingOutputStreamUnitTest {
    static int MAX = 5;

    @Test(expected = RuntimeException.class)
    public void givenData_whenCountReachesLimit_thenThrowException()
      throws Exception {

        ByteArrayOutputStream out = new ByteArrayOutputStream();
        CountingOutputStream cos = new CountingOutputStream(out);

        byte[] data = new byte[1024];
        ByteArrayInputStream in = new ByteArrayInputStream(data);

        int b;
        while ((b = in.read()) != -1) {
            cos.write(b);
            if (cos.getCount() >= MAX) {
                throw new RuntimeException("Write limit reached");
            }
        }
    }
} 
```

## 4。结论

在这篇简短的文章中，我们已经了解了`CountingOutputStream`类及其用法。该类提供了额外的方法`getCount()`，该方法返回目前为止写入`OutputStream`的字节数。

最后，和往常一样，讨论中使用的代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220523235704/https://github.com/eugenp/tutorials/tree/master/guava-modules/guava-io)