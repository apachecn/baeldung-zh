# 使用 Postman 进行负载测试

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/postman-load-testing>

## 1.概观

负载测试是现代企业应用软件开发生命周期(SDLC)的重要组成部分。在本教程中，**我们将使用[邮递员集合](https://web.archive.org/web/20220627150323/https://www.postman.com/collection/)来执行一个简单的负载测试活动**。

## 2.设置

我们可以下载并安装与我们系统的操作系统兼容的桌面客户端。或者，我们可以创建一个免费的邮差账户并访问[网络客户端](https://web.archive.org/web/20220627150323/https://go.postman.co/home)。

现在，让我们创建一个名为“Google Apps 负载测试”的新集合，方法是导入几个以 Postman 的集合格式 v2.1 提供的示例 HTTP 请求:

```java
{
  "info": {
    "_postman_id": "ddbb5536-b6ad-4247-a715-52a5d518b648",
    "name": "Google Apps - Load Testing",
    "schema": "https://schema.getpostman.com/json/collection/v2.1.0/collection.json"
  },
  "item": [
    {
      "name": "Get Google",
      "event": [
        {
          "listen": "test",
          "script": {
            "exec": [
              ""
            ],
            "type": "text/javascript"
          }
        }
      ],
      "request": {
        "method": "GET",
        "header": [],
        "url": {
          "raw": "https://www.google.com",
          "protocol": "https",
          "host": [
            "www",
            "google",
            "com"
          ]
        }
      },
      "response": []
    },
    {
      "name": "Get Youtube",
      "event": [
        {
          "listen": "test",
          "script": {
            "exec": [
              ""
            ],
            "type": "text/javascript"
          }
        }
      ],
      "request": {
        "method": "GET",
        "header": [],
        "url": {
          "raw": "https://www.youtube.com/",
          "protocol": "https",
          "host": [
            "www",
            "youtube",
            "com"
          ],
          "path": [
            ""
          ]
        }
      },
      "response": []
    },
    {
      "name": "Get Google Translate",
      "event": [
        {
          "listen": "test",
          "script": {
            "exec": [
              ""
            ],
            "type": "text/javascript"
          }
        }
      ],
      "request": {
        "method": "GET",
        "header": [],
        "url": {
          "raw": "https://translate.google.com/",
          "protocol": "https",
          "host": [
            "translate",
            "google",
            "com"
          ],
          "path": [
            ""
          ]
        }
      },
      "response": []
    }
  ]
}
```

我们应该**在导入数据的同时使用“原始文本”选项** :
[![](img/d4dffcaa53d6e7e981a6a5675098a521.png)](/web/20220627150323/https://www.baeldung.com/wp-content/uploads/2022/06/Postman-Import-Collection.png) 
就是这样！我们只需要通过点击 Continue 操作来完成导入任务，我们将在 Postman 中准备好我们的测试集合。

## 3.使用邮差收款员

在这一节中，我们将探索如何**使用 Postman 的 Collection Runner 来执行“Google Apps-Load Testing”集合中的 API 请求**并执行基本的负载测试。

### 3.1.基本配置

我们可以通过右键单击集合来启动集合运行器:

[![](img/9c53f582d17acdf4bd66ce7577c48a61.png)](/web/20220627150323/https://www.baeldung.com/wp-content/uploads/2022/06/Postman-Collection-Runner.png)

在 Runner 模式中，让我们**通过指定执行顺序、迭代次数和连续 API 命中之间的延迟来配置运行**:

[![](img/70bc1c70edd97c00b929530fe34ac09b.png)](/web/20220627150323/https://www.baeldung.com/wp-content/uploads/2022/06/Postman-Runner-Mode.png)

接下来，让我们点击“运行 Google Apps 负载测试”,开始对集合中的 API 请求进行基本负载测试:

[![](img/58f6a8b732a775a1cbf4601812c569a7.png)](/web/20220627150323/https://www.baeldung.com/wp-content/uploads/2022/06/Postman-Basic-Load-Testing.png)

当运行者执行 API 请求时，我们可以看到跨越多次迭代的每个 API 命中的实时结果。

### 3.2.使用测试脚本的高级配置

使用 Postman GUI，我们能够控制 API 的执行顺序。然而，我们可以通过使用 Postman 的测试脚本特性**获得对执行流程的更好的控制。**

比方说，只有当对“Google API”的点击返回 HTTP `200`状态代码时，我们才希望在工作流中包含“Google Translate”API。不然我们要直接打“Youtube API”:

[![](img/0c3e59d335aaf1f449598cbaa31ab7cd.png)](/web/20220627150323/https://www.baeldung.com/wp-content/uploads/2022/06/Postman-Advanced-Flow.png)

我们首先在 Tests 部分为“Get Google”请求添加一个简单的条件语句:

```java
if (pm.response.code == 200) {
    postman.setNextRequest("Get Google Translate");
}
else {
    postman.setNextRequest("Get Youtube");
}
```

接下来，我们将“Get Youtube”设置为在“Get Google Translate”之后执行的后续请求:

```java
postman.setNextRequest("Get Youtube");
```

此外，我们知道“Get Youtube”是流程中的最后一个请求，所以我们将它后面的下一个请求设置为`null`:

```java
postman.setNextRequest(null);
```

最后，让我们看看测试脚本的完整集合:

```java
{
  "info": {
    "_postman_id": "ddbb5536-b6ad-4247-a715-52a5d518b648",
    "name": "Google Apps - Load Testing",
    "schema": "https://schema.getpostman.com/json/collection/v2.1.0/collection.json"
  },
  "item": [
    {
      "name": "Get Google",
      "event": [
        {
          "listen": "test",
          "script": {
            "exec": [
              "if (pm.response.code == 200) {",
              "  postman.setNextRequest(\"Get Google Translate\");",
              "}",
              "else {",
              "  postman.setNextRequest(\"Get Youtube\");",
              "}"
            ],
            "type": "text/javascript"
          }
        }
      ],
      "request": {
        "method": "GET",
        "header": [],
        "url": {
          "raw": "https://www.google.com",
          "protocol": "https",
          "host": [
            "www",
            "google",
            "com"
          ]
        }
      },
      "response": []
    },
    {
      "name": "Get Youtube",
      "event": [
        {
          "listen": "test",
          "script": {
            "exec": [
              "postman.setNextRequest(null);"
            ],
            "type": "text/javascript"
          }
        }
      ],
      "request": {
        "method": "GET",
        "header": [],
        "url": {
          "raw": "https://www.youtube.com/",
          "protocol": "https",
          "host": [
            "www",
            "youtube",
            "com"
          ],
          "path": [
            ""
          ]
        }
      },
      "response": []
    },
    {
      "name": "Get Google Translate",
      "event": [
        {
          "listen": "test",
          "script": {
            "exec": [
              "postman.setNextRequest(\"Get Youtube\");"
            ],
            "type": "text/javascript"
          }
        }
      ],
      "request": {
        "method": "GET",
        "header": [],
        "url": {
          "raw": "https://translate.google.com/",
          "protocol": "https",
          "host": [
            "translate",
            "google",
            "com"
          ],
          "path": [
            ""
          ]
        }
      },
      "response": []
    }
  ]
}
```

像前面一样，我们可以使用 Collection Runner 来执行这个定制流。

## 4.使用纽曼转轮

我们可以使用 [Newman](https://web.archive.org/web/20220627150323/https://github.com/postmanlabs/newman) CLI 实用程序通过命令行运行 Postman 集合。**采用这种方法为自动化打开了更广阔的机会**。

让我们使用它为现有的集合运行两次定制流程迭代:

```java
newman run -n2 "Custom Flow Google Apps - Load Testing.postman_collection.json"
```

一旦所有的迭代都结束了，我们会得到一个统计摘要，在这里我们可以看到请求的平均响应时间:
[![](img/1ba36311808b415becb283a56f3b8b11.png)](/web/20220627150323/https://www.baeldung.com/wp-content/uploads/2022/06/newman-result-summary.png)

我们必须注意到**我们在演示中故意使用较低的值，因为大多数现代服务都有速率限制和请求阻塞逻辑，这将开始阻塞我们对较高值或持续时间的请求**。

## 5.使用 Grafana k6

Postman 是制定请求收集和执行流程的最简单的方法。然而，当**使用 Postman 或 Newman 时，我们依次调用请求**。

在一个实际的场景中，我们需要测试我们的系统同时来自多个用户的请求。对于这样的用例，我们可以使用 [Grafana 的 k6](https://web.archive.org/web/20220627150323/https://k6.io/open-source/) 实用程序。

首先，我们需要将现有的 Postman 集合转换成 k6 兼容格式。我们可以为这个里程碑使用`[postman-to-k6](https://web.archive.org/web/20220627150323/https://github.com/apideck-libraries/postman-to-k6)`库:

```java
postman-to-k6 "Google Apps - Load Testing.json" -o k6-script.js
```

接下来，让我们对两个虚拟用户进行三秒钟的实时运行:

```java
k6 run --duration 3s --vus 2 k6-script.js
```

完成后，我们会得到一个详细的统计报告，显示平均响应时间、迭代次数等指标:
[![](img/fcbbcb67579cbf784fa525b92e180a92.png)](/web/20220627150323/https://www.baeldung.com/wp-content/uploads/2022/06/k6.png)

## 6.结论

在本教程中，我们**使用 GUI 和 Newman runner 利用 Postman 集合进行基本负载测试**。此外，我们还了解了 k6 实用程序，它可以用来对 Postman 集合中的请求进行高级负载测试。