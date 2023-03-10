# 使用 Mockito ArgumentCaptor

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/mockito-argumentcaptor>

## 1.概观

在本教程中，我们将介绍在单元测试中使用 Mockito `ArgumentCaptor`的一个常见用例。

或者，对于其他` Mockito.verify`用例，参见我们的 [Mockito 验证食谱](/web/20221108185122/https://www.baeldung.com/mockito-verify)。

## 延伸阅读:

## [摩奇托附加答案介绍](/web/20221108185122/https://www.baeldung.com/mockito-additionalanswers)

A quick and practical guide to Mockito's AdditionalAnswers.[Read more](/web/20221108185122/https://www.baeldung.com/mockito-additionalanswers) →

## [模拟参数匹配](/web/20221108185122/https://www.baeldung.com/mockito-argument-matchers)

Learn how to use the ArgumentMatcher and how it differs from the ArgumentCaptor.[Read more](/web/20221108185122/https://www.baeldung.com/mockito-argument-matchers) →

## [莫奇托验证食谱](/web/20221108185122/https://www.baeldung.com/mockito-verify)

**Mockito Verify** examples, usage and best practices.[Read more](/web/20221108185122/https://www.baeldung.com/mockito-verify) →

## 2.使用`ArgumentCaptor`

`ArgumentCaptor`允许我们捕捉传递给方法的参数来检查它。当我们不能访问我们想要测试的方法之外的参数时，这个**特别有用。**

例如，考虑一个带有我们想要测试的`send`方法的`EmailService`类:

```java
public class EmailService {

    private DeliveryPlatform platform;

    public EmailService(DeliveryPlatform platform) {
        this.platform = platform;
    }

    public void send(String to, String subject, String body, boolean html) {
        Format format = Format.TEXT_ONLY;
        if (html) {
            format = Format.HTML;
        }
        Email email = new Email(to, subject, body);
        email.setFormat(format);
        platform.deliver(email);
    }

    ...
}
```

在`EmailService`里。`send`，注意 *platform.deliver* 如何将新的`Email`作为参数。作为我们测试的一部分，我们想要检查新的`Email`的格式字段是否被设置为`Format.HTML`。为此，我们需要捕获并检查传递给 `platform.deliver`的参数。

让我们看看如何利用`ArgumentCaptor`来帮助我们。

### 2.1.设置单元测试

首先，让我们创建我们的单元测试类:

```java
@RunWith(MockitoJUnitRunner.class)
public class EmailServiceUnitTest {

    @Mock
    DeliveryPlatform platform;

    @InjectMocks
    EmailService emailService;

    ...
}
```

我们使用`@Mock`注释来模仿`DeliveryPlatform`，它通过`@InjectMocks`注释被自动注入到我们的`EmailService`中。请参考我们的 [Mockito 注释](/web/20221108185122/https://www.baeldung.com/mockito-annotations)文章了解更多详情。

### 2.2.添加一个`ArgumentCaptor`字段

其次，让我们添加一个新的`Email`类型的`ArgumentCaptor`字段来存储我们捕获的参数:

```java
@Captor
ArgumentCaptor<Email> emailCaptor;
```

### 2.3.抓住论点

第三，让我们用 `Mockito.verify`和`ArgumentCaptor`来捕捉`Email`:

```java
Mockito.verify(platform).deliver(emailCaptor.capture());
```

然后我们可以获取捕获的值，并将其存储为一个新的`Email`对象:

```java
Email emailCaptorValue = emailCaptor.getValue();
```

### 2.4.检查捕获的值

最后，让我们看一下整个测试，用一个断言来检查被捕获的`Email`对象:

```java
@Test
public void whenDoesSupportHtml_expectHTMLEmailFormat() {
    String to = "[[email protected]](/web/20221108185122/https://www.baeldung.com/cdn-cgi/l/email-protection)";
    String subject = "Using ArgumentCaptor";
    String body = "Hey, let'use ArgumentCaptor";

    emailService.send(to, subject, body, true);

    Mockito.verify(platform).deliver(emailCaptor.capture());
    Email value = emailCaptor.getValue();
    assertThat(value.getFormat()).isEqualTo(Format.HTML);
}
```

## 3.避免磕碰

虽然**我们可以在 stubbing 中使用一个`ArgumentCaptor`,但是我们通常应该避免这样做。**澄清一下，在 Mockito 中，这通常意味着避免使用带有`Mockito.when`的`ArgumentCaptor`。对于 stubbing，我们应该使用一个`ArgumentMatcher`来代替。

让我们看看为什么我们应该避免磕碰的几个原因。

### 3.1.降低测试可读性

首先，考虑一个简单的测试:

```java
Credentials credentials = new Credentials("baeldung", "correct_password", "correct_key");
Mockito.when(platform.authenticate(Mockito.eq(credentials)))
  .thenReturn(AuthenticationStatus.AUTHENTICATED);

assertTrue(emailService.authenticatedSuccessfully(credentials));
```

这里我们使用`Mockito.eq(credentials)`来指定 mock 应该何时返回一个对象。

接下来，考虑使用`ArgumentCaptor`的相同测试:

```java
Credentials credentials = new Credentials("baeldung", "correct_password", "correct_key");
Mockito.when(platform.authenticate(credentialsCaptor.capture()))
  .thenReturn(AuthenticationStatus.AUTHENTICATED);

assertTrue(emailService.authenticatedSuccessfully(credentials));
assertEquals(credentials, credentialsCaptor.getValue());
```

与第一个测试相比，注意我们如何在最后一行执行额外的断言来做与`Mockito.eq(credentials)`相同的事情。

最后，注意一下`credentialsCaptor.capture()`指的是什么并不清楚。**这是因为我们必须在使用捕获器的行之外创建捕获器，这降低了可读性。**

### 3.2。减少缺陷定位

另一个原因是，如果`emailService.authenticatedSuccessfully`不调用`platform.authenticate`，我们会得到一个异常:

```java
org.mockito.exceptions.base.MockitoException: 
No argument value was captured!
```

这是因为我们的存根方法没有捕获参数。然而，真正的问题不在于我们的测试本身，而在于我们正在测试的实际方法。

换句话说，它将我们误导到测试中的一个异常，而实际的缺陷存在于我们正在测试的方法中。

## 4.结论

在这篇短文中，我们看了使用`ArgumentCaptor`的一般用例。我们还研究了避免使用带有存根的`ArgumentCaptor`的原因。

像往常一样，我们所有的代码样本都可以在 GitHub 上找到[。](https://web.archive.org/web/20221108185122/https://github.com/eugenp/tutorials/tree/master/testing-modules/mockito-simple)