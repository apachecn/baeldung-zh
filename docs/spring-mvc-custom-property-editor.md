# Spring 自定义属性编辑器

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-mvc-custom-property-editor>

## 1。简介

简单地说，Spring 大量使用属性编辑器来管理`String`值和自定义`Object`类型之间的转换；这是基于[的 Java Beans 属性编辑器](https://web.archive.org/web/20220628160207/https://docs.oracle.com/en/java/javase/11/docs/api/java.desktop/java/beans/PropertyEditor.html)。

在本教程中，我们将讨论两个不同的用例，来演示**自动属性编辑器绑定和自定义属性编辑器绑定**。

## 2。自动属性编辑器绑定

标准的`JavaBeans`基础设施将自动发现`PropertyEditor`类，如果它们和它们处理的类在同一个包中的话。此外，这些需要有与该类相同的名称加上`Editor`后缀。

例如，如果我们创建了一个`CreditCard`模型类，那么我们应该将编辑器类命名为`CreditCardEditor.`

现在让我们来看一个实用的属性绑定例子。

在我们的场景中，我们将在请求 URL 中将信用卡号作为路径变量传递，并将该值绑定为`a CreditCard`对象。

让我们首先创建`CreditCard`模型类定义字段`rawCardNumber,`银行识别号(前 6 位)、账号(7 到 15 位)和校验码(最后一位):

```java
public class CreditCard {

    private String rawCardNumber;
    private Integer bankIdNo;
    private Integer accountNo;
    private Integer checkCode;

    // standard constructor, getters, setters
}
```

接下来，我们将创建`CreditCardEditor`类。这实现了将作为`String`给出的信用卡号转换为`CreditCard`对象的业务逻辑。

**属性编辑器类应该扩展`PropertyEditorSupport`并实现`getAsText()`和`setAsText()`方法:**

```java
public class CreditCardEditor extends PropertyEditorSupport {

    @Override
    public String getAsText() {
        CreditCard creditCard = (CreditCard) getValue();

        return creditCard == null ? "" : creditCard.getRawCardNumber();
    }

    @Override
    public void setAsText(String text) throws IllegalArgumentException {
        if (StringUtils.isEmpty(text)) {
            setValue(null);
        } else {
            CreditCard creditCard = new CreditCard();
            creditCard.setRawCardNumber(text);

            String cardNo = text.replaceAll("-", "");
            if (cardNo.length() != 16)
                throw new IllegalArgumentException(
                  "Credit card format should be xxxx-xxxx-xxxx-xxxx");

            try {
                creditCard.setBankIdNo( Integer.valueOf(cardNo.substring(0, 6)) );
                creditCard.setAccountNo( Integer.valueOf(
                  cardNo.substring(6, cardNo.length() - 1)) );
                creditCard.setCheckCode( Integer.valueOf(
                  cardNo.substring(cardNo.length() - 1)) );
            } catch (NumberFormatException nfe) {
                throw new IllegalArgumentException(nfe);
            }

            setValue(creditCard);
        }
    }
}
```

**将一个对象序列化为`String,`时调用`getAsText()`方法，而`setAsText()`用于将`String`转换为另一个对象。**

由于这些类位于同一个包中，我们不需要为绑定类型`CreditCard`的`Editor`做任何其他事情。

我们现在可以将它作为 REST API 中的资源公开；该操作将信用卡号作为请求路径变量，Spring 将该文本值绑定为一个`CrediCard`对象，并将其作为方法参数传递:

```java
@GetMapping(value = "/credit-card/{card-no}", 
  produces = MediaType.APPLICATION_JSON_UTF8_VALUE)
public CreditCard parseCreditCardNumber(
    @PathVariable("card-no") CreditCard creditCard) {
    return creditCard;
}
```

例如，对于一个示例请求 URL `/property-editor/credit-card/1234-1234-1111-0019,`,我们将得到响应:

```java
{
    "rawCardNumber": "1234-1234-1111-0011",
    "bankIdNo": 123412,
    "accountNo": 341111001,
    "checkCode": 9
}
```

## 3。自定义属性编辑器绑定

如果我们在同一个包中没有所需的类型类和属性编辑器类，或者没有预期的命名约定，我们将不得不在所需的类型和属性编辑器之间定义一个自定义绑定。

在我们的自定义属性编辑器绑定场景中，一个`String`值将作为路径变量在 URL 中传递，我们将该值绑定为一个 `ExoticType`对象，该对象仅将该值作为属性保存。

和第 2 节一样，让我们首先创建一个模型类`ExoticType:`

```java
public class ExoticType {
    private String name;

    // standard constructor, getters, setters
}
```

而我们的自定义属性编辑器类`CustomExoticTypeEditor`又扩展了`PropertyEditorSupport` `:`

```java
public class CustomExoticTypeEditor extends PropertyEditorSupport {

    @Override
    public String getAsText() {
        ExoticType exoticType = (ExoticType) getValue();
        return exoticType == null ? "" : exoticType.getName();
    }

    @Override
    public void setAsText(String text) throws IllegalArgumentException {
        ExoticType exoticType = new ExoticType();
        exoticType.setName(text.toUpperCase());

        setValue(exoticType);
    }
}
```

由于 Spring 不能检测属性编辑器，**我们将需要一个在我们的`Controller`类中用`@InitBinder`注释的方法来注册编辑器:**

```java
@InitBinder
public void initBinder(WebDataBinder binder) {
    binder.registerCustomEditor(ExoticType.class, 
        new CustomExoticTypeEditor());
}
```

然后我们可以将用户输入绑定到`ExoticType`对象:

```java
@GetMapping(
  value = "/exotic-type/{value}", 
  produces = MediaType.APPLICATION_JSON_UTF8_VALUE)
public ExoticType parseExoticType(
  @PathVariable("value") ExoticType exoticType) {
    return exoticType;
}
```

对于示例请求 URL `/property-editor/exotic-type/passion-fruit, `,我们将获得示例响应:

```java
{
    "name": "PASSION-FRUIT"
}
```

## 4。结论

在这篇简短的文章中，我们看到了如何使用自动和定制的属性编辑器绑定来将人类可读的`String`值转换为复杂的 Java 类型。

我们这里例子的完整源代码一如既往地在 GitHub 的[上。](https://web.archive.org/web/20220628160207/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-data)