# 使用 libphonenumber 验证电话号码

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-libphonenumber>

## 1。概述

在这个快速教程中，我们将看到如何使用**谷歌的开源库 [`libphonenumber`](https://web.archive.org/web/20220628130506/https://github.com/google/libphonenumber) 在 Java** 中验证电话号码。

## 2。Maven 依赖关系

首先，我们需要在我们的`pom.xml`中添加这个库的依赖项:

```java
<dependency>
    <groupId>com.googlecode.libphonenumber</groupId>
    <artifactId>libphonenumber</artifactId>
    <version>8.12.10</version>
</dependency>
```

最新版本信息可以在 [Maven Central](https://web.archive.org/web/20220628130506/https://search.maven.org/artifact/com.googlecode.libphonenumber/libphonenumber) 上找到。

现在，我们已经准备好使用这个库提供的所有功能。

## 3。`PhoneNumberUtil`

该库提供了一个实用程序类， [`PhoneNumberUtil`](https://web.archive.org/web/20220628130506/https://www.javadoc.io/doc/com.googlecode.libphonenumber/libphonenumber/8.12.9/com/google/i18n/phonenumbers/PhoneNumberUtil.html) ，它提供了几个处理电话号码的方法。

让我们看几个例子，看看如何使用它的各种 API 进行验证。

重要的是，在所有的例子中，我们将使用这个类的 singleton 对象进行方法调用:

```java
PhoneNumberUtil phoneNumberUtil = PhoneNumberUtil.getInstance();
```

### 3.1.`isPossibleNumber`

使用`P`*honeNumberUtil # isPossibleNumber*，我们可以检查一个给定的号码对于一个特定的[国家代码](https://web.archive.org/web/20220628130506/https://countrycode.org/)或地区是否可能。

以美国为例，它的国家代码是 1。我们可以用这种方式检查给定的电话号码是否可能是美国号码:

```java
@Test
public void givenPhoneNumber_whenPossible_thenValid() {
    PhoneNumber number = new PhoneNumber();
    number.setCountryCode(1).setNationalNumber(123000L);
    assertFalse(phoneNumberUtil.isPossibleNumber(number));
    assertFalse(phoneNumberUtil.isPossibleNumber("+1 343 253 00000", "US"));
    assertFalse(phoneNumberUtil.isPossibleNumber("(343) 253-00000", "US"));
    assertFalse(phoneNumberUtil.isPossibleNumber("dial p for pizza", "US"));
    assertFalse(phoneNumberUtil.isPossibleNumber("123-000", "US"));
}
```

在这里，**我们也使用了这个函数的另一个变体，将我们期望拨打的号码所在的地区**作为一个`String`传入。

### 3.2.`isPossibleNumberForType`

该库可以识别不同类型的电话号码，如固定电话、移动电话、免费电话、语音邮件、网络电话、寻呼机以及更多的电话号码。

它的实用方法`isPossibleNumberForType`检查给定的数字对于特定区域的给定类型是否可能。

例如，我们以阿根廷为例，因为它允许不同类型的号码有不同的长度。

因此，我们可以用它来展示这个 API 的能力:

```java
@Test
public void givenPhoneNumber_whenPossibleForType_thenValid() {
    PhoneNumber number = new PhoneNumber();
    number.setCountryCode(54);

    number.setNationalNumber(123456);
    assertTrue(phoneNumberUtil.isPossibleNumberForType(number, PhoneNumberType.FIXED_LINE));
    assertFalse(phoneNumberUtil.isPossibleNumberForType(number, PhoneNumberType.TOLL_FREE));

    number.setNationalNumber(12345678901L);
    assertFalse(phoneNumberUtil.isPossibleNumberForType(number, PhoneNumberType.FIXED_LINE));
    assertTrue(phoneNumberUtil.isPossibleNumberForType(number, PhoneNumberType.MOBILE));
    assertFalse(phoneNumberUtil.isPossibleNumberForType(number, PhoneNumberType.TOLL_FREE));
}
```

正如我们所看到的，上面的代码验证了阿根廷允许 6 位数的固定电话号码和 11 位数的移动电话号码。

### 3.3.`isAlphaNumber`

该方法用于验证给定的电话号码是否是有效的字母数字号码，例如`325-CARS`:

```java
@Test
public void givenPhoneNumber_whenAlphaNumber_thenValid() {
    assertTrue(phoneNumberUtil.isAlphaNumber("325-CARS"));
    assertTrue(phoneNumberUtil.isAlphaNumber("0800 REPAIR"));
    assertTrue(phoneNumberUtil.isAlphaNumber("1-800-MY-APPLE"));
    assertTrue(phoneNumberUtil.isAlphaNumber("1-800-MY-APPLE.."));
    assertFalse(phoneNumberUtil.isAlphaNumber("+876 1234-1234"));
}
```

为了澄清，有效的字母数字开头至少包含三个数字，后面是三个或更多的字母。上面的实用程序方法首先去除给定输入的任何格式，然后检查这个条件。

### 3.4.`isValidNumber`

我们之前讨论的 API 只根据电话号码的长度来快速检查电话号码。另一方面， **`isValidNumber`使用前缀和长度信息**进行完整的验证:

```java
@Test
public void givenPhoneNumber_whenValid_thenOK() throws Exception {

    PhoneNumber phone = phoneNumberUtil.parse("+911234567890", 
      CountryCodeSource.UNSPECIFIED.name());

    assertTrue(phoneNumberUtil.isValidNumber(phone));
    assertTrue(phoneNumberUtil.isValidNumberForRegion(phone, "IN"));
    assertFalse(phoneNumberUtil.isValidNumberForRegion(phone, "US"));
    assertTrue(phoneNumberUtil.isValidNumber(phoneNumberUtil.getExampleNumber("IN")));
}
```

这里，当我们没有指定一个区域时，以及当我们指定了一个区域时，数字都被验证。

### 3.5.`isNumberGeographical​`

此方法检查给定的数字是否有与之关联的地理或区域:

```java
@Test
public void givenPhoneNumber_whenNumberGeographical_thenValid() throws NumberParseException {

    PhoneNumber phone = phoneNumberUtil.parse("+911234567890", "IN");
    assertTrue(phoneNumberUtil.isNumberGeographical(phone));

    phone = new PhoneNumber().setCountryCode(1).setNationalNumber(2530000L);
    assertFalse(phoneNumberUtil.isNumberGeographical(phone));

    phone = new PhoneNumber().setCountryCode(800).setNationalNumber(12345678L);
    assertFalse(phoneNumberUtil.isNumberGeographical(phone));
}
```

这里，在上面的第一个断言中，我们给出了带有区号的国际格式的电话号码，该方法返回 true。第二个断言使用美国的本地号码，第三个断言使用免费号码。所以 API 为这两个函数返回了 false。

## 4。结论

在本教程中，我们看到了由`libphonenumber`提供的一些使用代码样本格式化和验证电话号码的功能。

这是一个丰富的库，提供了更多的实用函数，并满足了我们应用程序对格式化、解析和验证电话号码的大部分需求。

和往常一样，源代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20220628130506/https://github.com/eugenp/tutorials/tree/master/libraries-6)