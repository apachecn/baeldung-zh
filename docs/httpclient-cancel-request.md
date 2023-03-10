# Apache http client–取消请求

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/httpclient-cancel-request>

## 1。概述

这个快速教程展示了如何用 Apache HttpClient 取消一个 HTTP 请求。

这对于潜在的长时间运行的请求或大型下载文件尤其有用，否则它们会不必要地消耗带宽和连接。

如果你想更深入地了解 HttpClient 并学习其他很酷的东西，你可以直接阅读 HttpClient 教程**[。](/web/20220920100108/https://www.baeldung.com/httpclient-guide "Cool basic and more advanced things you can do with the HttpClient 4")**

## 2。中止获取请求

要中止正在进行的请求，客户端只需使用:

```java
request.abort();
```

这将确保客户端不必消耗请求的全部内容来释放连接:

```java
@Test
public void whenRequestIsCanceled_thenCorrect() 
  throws ClientProtocolException, IOException {
    HttpClient instance = HttpClients.custom().build();
    HttpGet request = new HttpGet(SAMPLE_URL);
    HttpResponse response = instance.execute(request);

    try {
        System.out.println(response.getStatusLine());
        request.abort();
    } finally {
        response.close();
    }
}
```

## 3。结论

本文演示了如何中止 HTTP 客户端正在进行的请求。停止长时间运行的请求的另一个选择是确保它们将[超时](/web/20220920100108/https://www.baeldung.com/httpclient-timeout "HttpClient Timeout")。

所有这些示例和代码片段的实现都可以在[我的 GitHub 项目](https://web.archive.org/web/20220920100108/https://github.com/eugenp/tutorials/tree/master/apache-httpclient "Github Project exemplifying Live HttpClient 4.3 examples") 中找到——这是一个基于 Eclipse 的项目，所以应该很容易导入和运行。