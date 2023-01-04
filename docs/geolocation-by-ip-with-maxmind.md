# Java 中的 IP 地理定位

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/geolocation-by-ip-with-maxmind>

## 1。简介

在本文中，我们将探讨如何使用 MaxMind GeoIP2 Java API 和免费的 GeoLite2 数据库从 IP 地址获取地理位置数据。

我们还将使用一个简单的 Spring MVC Web 演示应用程序来看到这一点。

## 2。入门

首先，您需要从 MaxMind 下载 GeoIP2 API 和 GeoLite2 数据库。

### 2.1。Maven 依赖关系

要在您的 Maven 项目中包含 MaxMind GeoIP2 API，请将以下内容添加到`pom.xml`文件中:

```java
<dependency>
    <groupId>com.maxmind.geoip2</groupId>
    <artifactId>geoip2</artifactId>
    <version>2.8.0</version>
</dependency>
```

要获得 API 的最新版本，可以在 [Maven Central](https://web.archive.org/web/20220815030644/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22com.maxmind.geoip2%22%20AND%20a%3A%22geoip2%22) 上找到。

### 2.2。下载数据库

接下来，你需要下载 [GeoLite2 数据库](https://web.archive.org/web/20220815030644/https://dev.maxmind.com/geoip/geoip2/geolite2/)。对于本教程，我们将使用 GeoLite2 城市数据库的二进制 gzip 版本。

在解压存档文件后，您将得到一个名为`GeoLite2-City.mmdb`的文件。这是一个专有的 MaxMind 二进制格式的 IP 到位置映射的数据库。

## 3。使用 GeoIP2 Java API

让我们使用 GeoIP2 Java API 从数据库中获取给定 IP 地址的位置数据。首先，让我们创建一个`DatabaseReader`来查询数据库:

```java
File database = new File(dbLocation);
DatabaseReader dbReader = new DatabaseReader.Builder(database).build();
```

接下来，让我们使用`city()` 方法来获取 IP 地址的城市数据:

```java
CityResponse response = dbReader.city(ipAddress);
```

`CityResponse`对象包含几条信息，而不仅仅是城市名。下面是一个 JUnit 测试示例，展示了如何打开数据库，获取 IP 地址的城市信息，并从`CityResponse`中提取这些信息:

```java
@Test
public void givenIP_whenFetchingCity_thenReturnsCityData() 
  throws IOException, GeoIp2Exception {
    String ip = "your-ip-address";
    String dbLocation = "your-path-to-mmdb";

    File database = new File(dbLocation);
    DatabaseReader dbReader = new DatabaseReader.Builder(database)
      .build();

    InetAddress ipAddress = InetAddress.getByName(ip);
    CityResponse response = dbReader.city(ipAddress);

    String countryName = response.getCountry().getName();
    String cityName = response.getCity().getName();
    String postal = response.getPostal().getCode();
    String state = response.getLeastSpecificSubdivision().getName();
}
```

## 4。在网络应用中使用 GeoIP

让我们看一个示例 web 应用程序，它从用户的公共 IP 地址获取地理位置数据，并在地图上显示位置。

我们将从一个基本的 Spring Web MVC 应用程序开始。然后我们将编写一个`Controller` 来接受 POST 请求中的 IP 地址，并返回一个 JSON 响应，其中包含从 GeoIP2 API 推导出的城市、纬度和经度。

最后，我们将编写一些 HTML 和 JavaScript，将用户的公共 IP 地址加载到表单中，向我们的`Controller`提交一个 Ajax POST 请求，并在 Google Maps 中显示结果。

### 4.1。响应实体类

让我们从定义保存地理位置响应的类开始:

```java
public class GeoIP {
    private String ipAddress;
    private String city;
    private String latitude;
    private String longitude;
    // constructors, getters and setters... 
}
```

### 4.2。服务等级

现在让我们编写使用 GeoIP2 Java API 和 GeoLite2 数据库获取地理位置数据的服务类:

```java
public class RawDBDemoGeoIPLocationService {
    private DatabaseReader dbReader;

    public RawDBDemoGeoIPLocationService() throws IOException {
        File database = new File("your-mmdb-location");
        dbReader = new DatabaseReader.Builder(database).build();
    }

    public GeoIP getLocation(String ip) 
      throws IOException, GeoIp2Exception {
        InetAddress ipAddress = InetAddress.getByName(ip);
        CityResponse response = dbReader.city(ipAddress);

        String cityName = response.getCity().getName();
        String latitude = 
          response.getLocation().getLatitude().toString();
        String longitude = 
          response.getLocation().getLongitude().toString();
        return new GeoIP(ip, cityName, latitude, longitude);
    }
}
```

### 4.3。弹簧控制器

让我们看看 Spring MVC 的`Controller`,它将“ipAddress”请求参数发送给我们的服务类，以获取地理位置响应数据:

```java
@RestController
public class GeoIPTestController {
    private RawDBDemoGeoIPLocationService locationService;

    public GeoIPTestController() throws IOException {
        locationService = new RawDBDemoGeoIPLocationService();
    }

    @PostMapping("/GeoIPTest")
    public GeoIP getLocation(
      @RequestParam(value="ipAddress", required=true) String ipAddress
    ) throws Exception {

        GeoIPLocationService<String, GeoIP> locationService 
          = new RawDBDemoGeoIPLocationService();
        return locationService.getLocation(ipAddress);
    }
}
```

### 4.4。HTML 表单

让我们添加调用 Spring `Controller,` 的前端代码，从包含 IP 地址的 HTML 表单开始:

```java
<body>
    <form id="ipForm" action="GeoIPTest" method="POST">
        <input type="text" name = "ipAddress" id = "ip"/>
        <input type="submit" name="submit" value="submit" /> 
    </form>
    ...
</body>
```

### 4.5。在客户端加载公共 IP 地址

现在，让我们使用 jQuery 和[ipify.org](https://web.archive.org/web/20220815030644/https://www.ipify.org/)JavaScript API，用用户的公共 IP 地址预填充“ipAddress”文本字段:

```java
<script src
   ="https://ajax.googleapis.com/ajax/libs/jquery/3.1.1/jquery.min.js">
</script>

<script type="text/javascript">
    $(document).ready (function () {
        $.get( "https://api.ipify.org?format=json", 
          function( data ) {
             $("#ip").val(data.ip) ;
        });
...
</script>
```

### 4.6。提交 Ajax POST 请求

当提交表单时，我们将向 Spring `Controller`发出一个 Ajax POST 请求，以检索带有地理位置数据的 JSON 响应:

```java
$( "#ipForm" ).submit(function( event ) {
    event.preventDefault();
    $.ajax({
        url: "GeoIPTest",
        type: "POST",
        contentType: 
         "application/x-www-form-urlencoded; charset=UTF-8", 
        data: $.param( {ipAddress : $("#ip").val()} ),
        complete: function(data) {},
        success: function(data) {
            $("#status").html(JSON.stringify(data));
            if (data.ipAddress !=null) {
                showLocationOnMap(data);
            }
        },
        error: function(err) {
            $("#status").html("Error:"+JSON.stringify(data));
            },
        });
});
```

### 4.7。JSON 响应示例

来自 Spring `Controller`的 JSON 响应将具有以下格式:

```java
{
    "ipAddress":"your-ip-address",
    "city":"your-city",
    "latitude":"your-latitude",
    "longitude":"your-longitude"
}
```

### 4.8。在谷歌地图上显示位置

要在谷歌地图上显示位置，您需要在 HTML 代码中包含谷歌地图 API:

```java
<script src="https://maps.googleapis.com/maps/api/js?key=YOUR-API-KEY" 
async defer></script>
```

您可以使用 Google 开发者控制台获得 Google Maps 的 API 密钥。

您还需要定义一个 HTML `<div>`标签来包含地图图像:

```java
<div id="map" style="height: 500px; width:100%; position:absolute"></div>
```

您可以使用以下 JavaScript 函数在 Google Maps 上显示坐标:

```java
function showLocationOnMap (location) {
    var map;
    map = new google.maps.Map(document.getElementById('map'), {
      center: {
        lat: Number(location.latitude), 
        lng: Number(location.longitude)},
        zoom: 15
    });
    var marker = new google.maps.Marker({
      position: {
        lat: Number(location.latitude), 
        lng: Number(location.longitude)},
        map: map,
        title: 
          "Public IP:"+location.ipAddress
            +" @ "+location.city
    });   
}
```

启动 web 应用程序后，打开地图页面的 URL:

```java
http://localhost:8080/spring-mvc-xml/GeoIpTest.jsp
```

您将看到文本框中加载了您的连接的当前公共 IP 地址:

[![Capture-2](img/9439034c7177f21dd3e6e1f00a37e92b.png)](/web/20220815030644/https://www.baeldung.com/wp-content/uploads/2016/11/Capture-2.jpg)

请注意，GeoIP2 和 ipify 都支持 IPv4 地址和 IPv6 地址。

当您提交表单时，您将看到 JSON 响应文本，包括与您的公共 IP 地址对应的城市、纬度和经度，在它下面，您将看到指向您所在位置的 Google 地图:

[![Capture-3](img/3953b3e859c3692b1f069f4bd3cbade6.png)](/web/20220815030644/https://www.baeldung.com/wp-content/uploads/2016/11/Capture-3.jpg)

## 5.结论

在本教程中，我们通过 JUnit 测试回顾了 MaxMind GeoIP2 Java API 和免费 MaxMind GeoLite2 City 数据库的用法。

然后我们构建了一个 Spring MVC `Controller`和服务来从一个 IP 地址获取地理位置数据(城市，纬度，经度)。

最后，我们构建了一个 HTML/JavaScript 前端来演示如何使用这个特性在 Google Maps 上显示用户的位置。

该产品包括 MaxMind 创建的 GeoLite2 数据，可从[http://www.maxmind.com](https://web.archive.org/web/20220815030644/https://www.maxmind.com/)获得。

本教程的代码可以在 Github 网站上找到。