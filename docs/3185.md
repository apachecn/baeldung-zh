# 从命令行调用 SOAP Web 服务

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/call-soap-command-line>

## 1.概观

在本教程中，我们将展示如何使用不同的命令行界面(CLI)过程来消费 SOAP web 服务。

## 2.SOAP Web 服务

为了运行本文中的示例，[我们将使用在上一篇文章](/web/20221129015121/https://www.baeldung.com/spring-boot-soap-web-service)中开发的 SOAP web 服务。简而言之，这个服务有一个客户端可以访问的端点，在请求中提供一个国家名称。该服务用国家的首都、人口和货币的名称进行回复。

## 3.卷曲

让我们从 cURL 开始，因为它可能是使用最广泛的通过网络协议传输数据的命令行工具。为了测试 SOAP web 服务，我们只需要在请求体中使用 SOAP 信封发出 HTTP 请求。

对于我们的 web 服务，一个简单的 HTTP POST 请求是:

```
curl -v --request POST --header "Content-Type: text/xml;charset=UTF-8" \
--data \
'<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:gs="http://www.baeldung.com/springsoap/gen"> \
<soapenv:Header/> \ 
<soapenv:Body> \ 
<gs:getCountryRequest> <gs:name>Poland</gs:name> </gs:getCountryRequest> \ 
</soapenv:Body> \ 
</soapenv:Envelope>' \
http://localhost:8080/ws
```

我们不需要指定我们正在使用 HTTP，因为它是 cURL 中的默认协议。因为我们正在测试请求，所以我们通过`-v`选项使用详细模式。

在 SOAP 信封中，我们指定了国家(波兰)，并用 SOAP 服务器 URL 结束命令。我们已经在我们的计算机上本地安装了服务器，使用了我们之前文章中的[示例。](/web/20221129015121/https://www.baeldung.com/spring-boot-soap-web-service)

由于我们使用了`-v`选项，我们得到了一个详细的响应:

```
* Connected to localhost (::1) port 8080 (#0)
> POST /ws HTTP/1.1
> Host: localhost:8080
> User-Agent: curl/7.55.1
> Accept: */*
> Content-Type: text/xml;charset=UTF-8
> Content-Length: 282
>
* upload completely sent off: 282 out of 282 bytes
< HTTP/1.1 200
< Accept: text/xml, text/html, image/gif, image/jpeg, *; q=.2, */*; q=.2
< SOAPAction: ""
< Content-Type: text/xml;charset=utf-8
< Content-Length: 407
< Date: Sun, 18 Jul 2021 23:46:38 GMT
<
<SOAP-ENV:Envelope xmlns:SOAP-ENV="http://schemas.xmlsoap.org/soap/envelope/"><SOAP-ENV:Header/><SOAP-ENV:Body><ns2:getCountryResponse xmlns:ns
2="http://www.baeldung.com/springsoap/gen"><ns2:country><ns2:name>Poland</ns2:name><ns2:population>38186860</ns2:population><ns2:capital>Warsaw
</ns2:capital><ns2:currency>PLN</ns2:currency></ns2:country></ns2:getCountryResponse></SOAP-ENV:Body></SOAP-ENV:Envelope>* Connection #0 to hos
t localhost left intact
```

SOAP web 服务的请求和响应消息可能很长，因此将它们存储在文件中会更方便。**如果我们将请求体保存在`request.xml`中，并将响应的输出重定向到文件`response.xml`，在这种情况下，命令非常简单**:

```
curl --header "Content-Type: text/xml;charset=UTF-8" -d @request.xml -o response.xml http://localhost:8080/ws
```

一般来说，没有必要像我们之前那样在命令中指定 POST，因为它是由 cURL 推断出来的。

如果我们需要在终端中读取响应，最好是用`xmllint`通过**管道命令来获得正确格式化的 XML 响应**:

```
curl --request POST --header "Content-Type: text/xml;charset=UTF-8" -d @request.xml http://localhost:8080/ws | xmllint --format -
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   725  100   407  100   318    407    318  0:00:01 --:--:--  0:00:01 15425<?xml version="1.0"?>
<SOAP-ENV:Envelope xmlns:SOAP-ENV="http://schemas.xmlsoap.org/soap/envelope/">
  <SOAP-ENV:Header/>
  <SOAP-ENV:Body>
    <ns2:getCountryResponse xmlns:ns2="http://www.baeldung.com/springsoap/gen">
      <ns2:country>
        <ns2:name>Poland</ns2:name>
        <ns2:population>38186860</ns2:population>
        <ns2:capital>Warsaw</ns2:capital>
        <ns2:currency>PLN</ns2:currency>
      </ns2:country>
    </ns2:getCountryResponse>
  </SOAP-ENV:Body>
</SOAP-ENV:Envelope> 
```

## 4.Wget

让我们使用 Wget 发出同样的请求:

```
wget --post-file=request.xml --header="Content-Type: text/xml" http://localhost:8080/ws -O response.xml 
```

回应是:

```
Resolving localhost (localhost)... ::1, 127.0.0.1
Connecting to localhost (localhost)|::1|:8080... connected.
HTTP request sent, awaiting response... 200
Length: 407 [text/xml]
Saving to: ‘response.xml’ 
```

语法类似于 cURL，我们可以像以前一样将请求和响应主体存储在文件中。

## 5.HTTPie

Wget 和 cURL 是快速测试 SOAP 服务器的非常有用的命令。它们在所有主要的操作系统发行版中都有。它们也可以很容易地与 shell 脚本集成。

HTTPie 的优势在于它提供了一种非常直观的方式来与 Web 服务进行交互。正如[文档](https://web.archive.org/web/20221129015121/https://httpie.io/docs)中所述:“HTTPie 是为测试、调试以及通常与 API&HTTP 服务器的交互而设计的。它们使用简单自然的语法，并提供格式化和彩色化的输出。”

让我们发出之前发出的简单请求，这次使用 HTTPie:

```
echo '<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:gs="http://www.baeldung.com/springsoap/gen"> \
<soapenv:Header/> \ 
<soapenv:Body> \ 
<gs:getCountryRequest> <gs:name>Poland</gs:name> </gs:getCountryRequest> \ 
</soapenv:Body> \ 
</soapenv:Envelope>' | \
http -b POST http://localhost:8080/ws 'Content-Type:text/xml'
```

如果我们想从文件中提取请求体:

```
http -b POST http://localhost:8080/ws 'Content-Type:text/xml' < request.xml
```

这就像使用文件输入重定向一样简单。

有关如何使用 HTTPie 的更多详细信息，请参考文档。

## 6.摘要

我们已经看到了如何使用 cURL、Wget 和 HTTPie 从命令行快速调用 SOAP Web 服务的简单示例。我们还对这三种工具以及何时使用它们进行了简单的比较。