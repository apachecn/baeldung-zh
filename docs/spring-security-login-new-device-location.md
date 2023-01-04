# 通知用户从新设备或位置登录

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-security-login-new-device-location>

[This article is part of a series:](javascript:void(0);)[• Spring Security Registration Tutorial](/web/20220626205221/https://www.baeldung.com/spring-security-registration)
[• The Registration Process With Spring Security](/web/20220626205221/https://www.baeldung.com/registration-with-spring-mvc-and-spring-security)
[• Registration – Activate a New Account by Email](/web/20220626205221/https://www.baeldung.com/registration-verify-user-by-email)
[• Spring Security Registration – Resend Verification Email](/web/20220626205221/https://www.baeldung.com/spring-security-registration-verification-email)
[• Registration with Spring Security – Password Encoding](/web/20220626205221/https://www.baeldung.com/spring-security-registration-password-encoding-bcrypt)
[• The Registration API becomes RESTful](/web/20220626205221/https://www.baeldung.com/registration-restful-api)
[• Spring Security – Reset Your Password](/web/20220626205221/https://www.baeldung.com/spring-security-registration-i-forgot-my-password)
[• Registration – Password Strength and Rules](/web/20220626205221/https://www.baeldung.com/registration-password-strength-and-rules)
[• Updating your Password](/web/20220626205221/https://www.baeldung.com/updating-your-password/)

## 1。简介

在本教程中，我们将演示如何通过**验证** **如果** **我们的** **用户****正在** **中****一个** **新** **设备/位置**。

我们将向他们发送登录通知，让他们知道我们在他们的帐户上检测到了不熟悉的活动。

## 2。用户的位置和设备详情

我们需要两样东西:用户的位置，以及他们用来登录的设备的信息。

考虑到我们使用 HTTP 与用户交换消息，我们将不得不完全依赖传入的 HTTP 请求及其元数据来检索这些信息。

幸运的是，HTTP 头的唯一目的就是携带这种信息。

### 2.1。设备位置

在我们估计用户的位置之前，我们需要获得他们的原始 IP 地址。

为此，我们可以使用:

*   [`X-Forwarded-For`](https://web.archive.org/web/20220626205221/https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Forwarded-For)–事实上的标准报头，用于标识通过 HTTP 代理或负载平衡器连接到 web 服务器的客户端的原始 IP 地址
*   [`ServletRequest.getRemoteAddr()`](https://web.archive.org/web/20220626205221/https://docs.oracle.com/javaee/6/api/javax/servlet/ServletRequest.html#getRemoteAddr())–一个实用程序方法，返回发送请求的客户端或最后一个代理的原始 IP

从 HTTP 请求中提取用户的 IP 地址不太可靠，因为它们可能被篡改。然而，让我们在我们的教程中简化这一点，并假设不会是这种情况。

一旦我们获得了 IP 地址，我们就可以通过 [`geolocation`](https://web.archive.org/web/20220626205221/https://en.wikipedia.org/wiki/Geolocation) 将其转换为现实世界的位置。

### 2.2。设备详情

与发起的 IP 地址类似，还有一个 HTTP 报头，它携带用于发送请求的设备的信息，称为 [`User-Agent`](https://web.archive.org/web/20220626205221/https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/User-Agent) 。

简而言之，它携带的信息允许我们**识别******`application`****`type`****`operating`****`system`****`software`****`vendor/version`********请求** **用户** **代理**。******

 ****下面是一个可能的例子:

```java
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_0) AppleWebKit/537.36 
  (KHTML, like Gecko) Chrome/71.0.3578.98 Safari/537.36
```

在我们上面的例子中，设备在`Mac` `OS` `X` `10.14`上运行，并使用`Chrome` `71.0`发送请求。

我们不会从头开始实现一个`User-Agent`解析器，而是求助于已经测试过并且更加可靠的现有解决方案。

## 3。检测新设备或位置

现在我们已经介绍了我们需要的信息，让我们修改我们的 [`AuthenticationSuccessHandler`](/web/20220626205221/https://www.baeldung.com/spring_redirect_after_login) 以便在用户登录后执行验证:

```java
public class MySimpleUrlAuthenticationSuccessHandler 
  implements AuthenticationSuccessHandler {
    //...
    @Override
    public void onAuthenticationSuccess(
      final HttpServletRequest request,
      final HttpServletResponse response,
      final Authentication authentication)
      throws IOException {
        handle(request, response, authentication);
        //...
        loginNotification(authentication, request);
    }

    private void loginNotification(Authentication authentication, 
      HttpServletRequest request) {
        try {
            if (authentication.getPrincipal() instanceof User) { 
                deviceService.verifyDevice(((User)authentication.getPrincipal()), request); 
            }
        } catch(Exception e) {
            logger.error("An error occurred verifying device or location");
            throw new RuntimeException(e);
        }
    }
    //...
}
```

我们简单地向新组件添加了一个调用:`DeviceService`。该组件将封装我们识别新设备/位置并通知用户所需的一切。

然而，在我们继续讨论我们的`DeviceService`之前，让我们创建我们的`DeviceMetadata`实体来持久化我们的用户数据:

```java
@Entity
public class DeviceMetadata {
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;
    private Long userId;
    private String deviceDetails;
    private String location;
    private Date lastLoggedIn;
    //...
}
```

及其`Repository`:

```java
public interface DeviceMetadataRepository extends JpaRepository<DeviceMetadata, Long> {
    List<DeviceMetadata> findByUserId(Long userId);
}
```

有了我们的`Entity`和`Repository`,我们可以开始收集我们需要的信息来记录我们用户的设备及其位置。

## 4。提取我们用户的位置

在我们估计用户的地理位置之前，我们需要提取他们的 IP 地址:

```java
private String extractIp(HttpServletRequest request) {
    String clientIp;
    String clientXForwardedForIp = request
      .getHeader("x-forwarded-for");
    if (nonNull(clientXForwardedForIp)) {
        clientIp = parseXForwardedHeader(clientXForwardedForIp);
    } else {
        clientIp = request.getRemoteAddr();
    }
    return clientIp;
}
```

如果请求中有一个`X-Forwarded-For`头，我们将使用它来提取它们的 IP 地址；否则，我们将使用`getRemoteAddr()`方法。

一旦我们有了他们的 IP 地址，我们就可以在`Maxmind` 的帮助下[估计他们的位置:](/web/20220626205221/https://www.baeldung.com/geolocation-by-ip-with-maxmind)

```java
private String getIpLocation(String ip) {
    String location = UNKNOWN;
    InetAddress ipAddress = InetAddress.getByName(ip);
    CityResponse cityResponse = databaseReader
      .city(ipAddress);

    if (Objects.nonNull(cityResponse) &&
      Objects.nonNull(cityResponse.getCity()) &&
      !Strings.isNullOrEmpty(cityResponse.getCity().getName())) {
        location = cityResponse.getCity().getName();
    }    
    return location;
}
```

## 5。 **用户** **设备** **详情**

因为`User-Agent`头包含了我们需要的所有信息，所以提取它只是一个问题。正如我们前面提到的，在`User-Agent`解析器的帮助下(本例中是 [`uap-java`](https://web.archive.org/web/20220626205221/https://search.maven.org/search?q=g:com.github.ua-parser%20AND%20a:uap-java) ，获取这些信息变得非常简单:

```java
private String getDeviceDetails(String userAgent) {
    String deviceDetails = UNKNOWN;

    Client client = parser.parse(userAgent);
    if (Objects.nonNull(client)) {
        deviceDetails = client.userAgent.family
          + " " + client.userAgent.major + "." 
          + client.userAgent.minor + " - "
          + client.os.family + " " + client.os.major
          + "." + client.os.minor; 
    }
    return deviceDetails;
}
```

## 6。发送登录通知

要向我们的用户发送登录通知，我们需要将提取的信息与过去的数据进行比较，以检查我们过去是否在该位置见过该设备。

让我们来看看我们的`DeviceService.` `verify` `Device()`方法:

```java
public void verifyDevice(User user, HttpServletRequest request) {

    String ip = extractIp(request);
    String location = getIpLocation(ip);

    String deviceDetails = getDeviceDetails(request.getHeader("user-agent"));

    DeviceMetadata existingDevice
      = findExistingDevice(user.getId(), deviceDetails, location);

    if (Objects.isNull(existingDevice)) {
        unknownDeviceNotification(deviceDetails, location,
          ip, user.getEmail(), request.getLocale());

        DeviceMetadata deviceMetadata = new DeviceMetadata();
        deviceMetadata.setUserId(user.getId());
        deviceMetadata.setLocation(location);
        deviceMetadata.setDeviceDetails(deviceDetails);
        deviceMetadata.setLastLoggedIn(new Date());
        deviceMetadataRepository.save(deviceMetadata);
    } else {
        existingDevice.setLastLoggedIn(new Date());
        deviceMetadataRepository.save(existingDevice);
    }
}
```

提取信息后，我们将其与现有的`DeviceMetadata`条目进行比较，以检查是否有包含相同信息的条目:

```java
private DeviceMetadata findExistingDevice(
  Long userId, String deviceDetails, String location) {
    List<DeviceMetadata> knownDevices
      = deviceMetadataRepository.findByUserId(userId);

    for (DeviceMetadata existingDevice : knownDevices) {
        if (existingDevice.getDeviceDetails().equals(deviceDetails) 
          && existingDevice.getLocation().equals(location)) {
            return existingDevice;
        }
    }
    return null;
}
```

如果没有，我们需要向用户发送通知，让他们知道我们在他们的帐户中检测到了不熟悉的活动。然后，我们保存信息。

否则，我们只需更新熟悉设备的`lastLoggedIn`属性。

## 7 .**。结论**

在本文中，我们演示了如何在检测到用户帐户中有不熟悉的活动时发送登录通知。

本教程的完整实现可以在 Github 上找到[。](https://web.archive.org/web/20220626205221/https://github.com/Baeldung/spring-security-registration/)****