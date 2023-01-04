# 使用百里香叶在春季格式化货币

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-thymeleaf-currencies>

## 1.介绍

在本教程中，我们将学习如何使用[百里香叶](https://web.archive.org/web/20220728105348/https://www.thymeleaf.org/)按地区格式化货币。

## 2.Maven 依赖性

让我们从导入 [Spring Boot 百里香依赖](https://web.archive.org/web/20220728105348/https://search.maven.org/artifact/org.springframework.boot/spring-boot-starter-thymeleaf)开始:

```java
<dependency>
    <groupId>org.springframework.boot</groupId> 
    <artifactId>spring-boot-starter-thymeleaf</artifactId> 
    <version>2.2.7.RELEASE</version>
</dependency>
```

## 3.项目设置

我们的项目将是一个简单的 Spring web 应用程序，**根据用户的地区显示货币。**让我们在`resources/templates/currencies`中创建百里香模板 `currencies.html`:

```java
<!DOCTYPE html>
<html 
  xmlns:th="http://www.thymeleaf.org">
    <head>
        <meta charset="UTF-8">
        <title>Currency table</title>
    </head>
</html>
```

我们还可以创建一个控制器类来处理我们的请求:

```java
@Controller
public class CurrenciesController {
    @GetMapping(value = "/currency")
    public String exchange(
      @RequestParam(value = "amount") String amount, Locale locale) {
        return "currencies/currencies";
    }
}
```

## 4.格式化

谈到货币，我们需要根据请求者的地区对它们进行格式化。

在这种情况下，我们将在每个请求中发送`Accept-Language`头来表示用户的地区。

### 4.1.货币

由百里叶提供的 [`Numbers`](https://web.archive.org/web/20220728105348/https://www.thymeleaf.org/apidocs/thymeleaf/3.0.11.RELEASE/org/thymeleaf/expression/Numbers.html) 类支持格式化货币。因此，让我们通过调用`formatCurrency`方法来更新视图

```java
<p th:text="${#numbers.formatCurrency(param.amount)}"></p>
```

当我们运行我们的示例时，我们将看到货币被正确格式化:

```java
@Test
public void whenCallCurrencyWithUSALocale_ThenReturnProperCurrency() throws Exception {
    mockMvc.perform(MockMvcRequestBuilders.get("/currency")
      .header("Accept-Language", "en-US")
      .param("amount", "10032.5"))
      .andExpect(status().isOk())
      .andExpect(content().string(containsString("$10,032.50")));
}
```

**因为我们将`Accept-Language`头设置为美国，所以货币被格式化为一个小数点和一个美元符号。**

### 4.2.货币数组

我们还可以使用`Numbers`类来格式化数组。因此，我们将向控制器添加另一个请求参数:

```java
@GetMapping(value = "/currency")
public String exchange(
  @RequestParam(value = "amount") String amount,
  @RequestParam(value = "amountList") List amountList, Locale locale) {
    return "currencies/currencies";
}
```

接下来，我们可以更新我们的视图，以包含对`listFormatCurrency`方法的调用:

```java
<p th:text="${#numbers.listFormatCurrency(param.amountList)}"></p> 
```

现在让我们看看结果是什么样的:

```java
@Test
public void whenCallCurrencyWithUkLocaleWithArrays_ThenReturnLocaleCurrencies() throws Exception {
    mockMvc.perform(MockMvcRequestBuilders.get("/currency")
      .header("Accept-Language", "en-GB")
      .param("amountList", "10", "20", "30"))
      .andExpect(status().isOk())
      .andExpect(content().string(containsString("£10.00, £20.00, £30.00")));
}
```

结果显示了添加了正确的英国格式的货币列表。

### 4.3.尾随零

使用 [`Strings#replace`](https://web.archive.org/web/20220728105348/https://www.thymeleaf.org/apidocs/thymeleaf/3.0.11.RELEASE/org/thymeleaf/expression/Strings.html) ，我们可以**去掉后面的零**。

```java
<p th:text="${#strings.replace(#numbers.formatCurrency(param.amount), '.00', '')}"></p>
```

现在我们可以看到没有尾随双零的全部金额:

```java
@Test
public void whenCallCurrencyWithUSALocaleWithoutDecimal_ThenReturnCurrencyWithoutTrailingZeros()
  throws Exception {
    mockMvc.perform(MockMvcRequestBuilders.get("/currency")
      .header("Accept-Language", "en-US")
      .param("amount", "10032"))
      .andExpect(status().isOk())
      .andExpect(content().string(containsString("$10,032")));
}
```

### 4.4.小数

根据语言环境的不同，小数的格式可能会有所不同。因此，如果我们想用逗号替换小数点，可以使用`Numbers`类提供的`formatDecimal`方法:

```java
<p th:text="${#numbers.formatDecimal(param.amount, 1, 2, 'COMMA')}"></p>
```

让我们看看测试的结果:

```java
@Test
public void whenCallCurrencyWithUSALocale_ThenReturnReplacedDecimalPoint() throws Exception {
    mockMvc.perform(MockMvcRequestBuilders.get("/currency")
      .header("Accept-Language", "en-US")
      .param("amount", "1.5"))
      .andExpect(status().isOk())
      .andExpect(content().string(containsString("1,5")));
}
```

该值将被格式化为“1，5”。

## 5.结论

在这个简短的教程中，我们展示了如何将百里香与 Spring Web 结合使用，根据用户的地区来处理货币。

和往常一样，代码可以在 GitHub 的[上获得。](https://web.archive.org/web/20220728105348/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-thymeleaf-3)