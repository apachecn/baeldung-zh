# 探索 SpringMVC 的表单标签库

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-mvc-form-tags>

## 1。概述

在本系列的第一篇文章中，我们介绍了表单标签库的使用以及如何将数据绑定到控制器。

在本文中，我们将介绍 Spring MVC 提供的各种标签，以帮助我们**创建和验证表单**。

## 2。`input`标签

我们将从`input`标签开始。默认情况下，该标签使用绑定值和`type='text'`呈现一个 HTML `input`标签:

```
<form:input path="name" />
```

从 Spring 3.1 开始，您可以使用其他特定于 HTML5 的类型，比如 email、date 等等。 例如如果我们想 创建一个邮件 字段我们可以使用 `type='email':`

```
<form:input type="email" path="email" />
```

类似地，创建 日期字段，我们可以使用`type='date'`，这将在许多兼容 HTML5 的浏览器中呈现日期选择器: 

```
<form:input type="date" path="dateOfBirth" />
```

## 3。`password`标签

这个标签使用绑定值呈现一个带有`type='password'`的 HTML `input`标签。该 HTML 输入屏蔽了在字段中键入的值:

```
<form:password path="password" />
```

## 4。`textarea`标签

这个标签呈现了一个 HTML `textarea`:

```
<form:textarea path="notes" rows="3" cols="20"/>
```

我们可以像指定 HTML `textarea`一样指定**行**和**列**的数量。

## 5。`checkbox`和`checkboxes`标签

`checkbox`标签呈现一个带有`type='checkbox'`的 HTML `input`标签。Spring MVC 的表单标签库为`checkbox`标签提供了不同的方法，可以满足我们所有的`checkbox`需求:

```
<form:checkbox path="receiveNewsletter" />
```

上面例子生成一个经典的单个`checkbox`，带有一个`boolean`值。如果我们将界限值设置为`true`，默认情况下该复选框将被选中。

以下示例生成多个复选框**。**在这种情况下，`checkbox`值被硬编码在 JSP 页面中:

```
Bird watching: <form:checkbox path="hobbies" value="Bird watching"/>
Astronomy: <form:checkbox path="hobbies" value="Astronomy"/>
Snowboarding: <form:checkbox path="hobbies" value="Snowboarding"/>
```

这里，绑定值是类型`array`或`java.util.Collection`:

```
String[] hobbies;
```

`checkboxes` 标签的用途是用来呈现多个复选框，其中的复选框值是在运行时生成的:

```
<form:checkboxes items="${favouriteLanguageItem}" path="favouriteLanguage" />
```

为了生成这些值，我们传入一个包含`items`属性中可用选项的`Array`、`List`或`Map`。我们可以在控制器内部初始化我们的值:

```
List<String> favouriteLanguageItem = new ArrayList<String>();
favouriteLanguageItem.add("Java");
favouriteLanguageItem.add("C++");
favouriteLanguageItem.add("Perl");
```

通常，绑定属性是一个集合，因此它可以保存用户选择的多个值:

```
List<String> favouriteLanguage;
```

## 6。`radiobutton`和`radiobuttons`标签

这个标签使用`type='radio':`呈现一个 HTML `input`标签

```
Male: <form:radiobutton path="sex" value="M"/>
Female: <form:radiobutton path="sex" value="F"/>
```

典型的使用模式包括多个标记实例，不同的值绑定到相同的属性:

```
private String sex;
```

就像`checkboxes`标签一样，`radiobuttons`标签使用`type='radio'`呈现多个 HTML `input`标签:

```
<form:radiobuttons items="${jobItem}" path="job" />
```

在这种情况下，我们可能希望将可用选项作为包含`items` 属性中可用选项的`Array`、`List`或`Map`进行传递:

```
List<String> jobItem = new ArrayList<String>();
jobItem.add("Full time");
jobItem.add("Part time");
```

## 7。`select`标签

这个标签呈现了一个 HTML `select`元素:

```
<form:select path="country" items="${countryItems}" />
```

为了生成这些值，我们传入一个包含`items` 属性中可用选项的`Array`、`List`或`Map`。 再来一次 ，我们可以在控制器内部初始化我们的值:

```
Map<String, String> countryItems = new LinkedHashMap<String, String>();
countryItems.put("US", "United States");
countryItems.put("IT", "Italy");
countryItems.put("UK", "United Kingdom");
countryItems.put("FR", "France");
```

select 标签也支持使用嵌套的`option`和`options`标签。

当`option`标签呈现单个 HTML `option`时，`options`标签呈现一系列 HTML `option`标签。

`options`标签接受一个`Array`、`List` 或`Map`，包含`items` 属性中的可用选项，就像`select`标签一样:

```
<form:select path="book">
    <form:option value="-" label="--Please Select--"/>
    <form:options items="${books}" />
</form:select>
```

当 我们有了 需要 来一次选择几个项目，我们就可以创建一个 多个列表框。 要呈现这种类型的列表，只需在`select`标签中添加`multiple=”true”`属性。

```
<form:select path="fruit" items="${fruit}" multiple="true"/>
```

这里绑定的属性是一个`array`或一个`java.util.Collection`:

```
List<String> fruit;
```

## 8。`hidden`标签

该标签使用绑定值呈现带有`type='hidden'`的 HTML `input`标签:

```
<form:hidden path="id" value="12345" />
```

## 9。`Errors`标签

字段错误消息由与控制器相关联的验证器生成。我们可以使用 Errors 标记来呈现这些字段错误消息:

```
<form:errors path="name" cssClass="error" />
```

这将显示在`path`属性中指定的字段的错误。默认情况下，错误消息呈现在一个`span` 标记中，将`.errors` 附加到`path`值作为`id`，并且可以选择来自`cssClass`属性的一个 CSS 类，该类可用于样式化输出:

```
<span id="name.errors" class="error">Name is required!</span>
```

要用不同的元素而不是默认的`span`标签来包含错误消息，我们可以在`element`属性中指定首选元素:

```
<form:errors path="name" cssClass="error" element="div" />
```

这将在一个`div` 元素中呈现错误消息:

```
<div id="name.errors" class="error">Name is required!</div>
```

I n 除了拥有 显示 特定输入 元素 的 错误的能力之外，我们还可以显示给定页面的整个错误列表(不考虑字段)。这是通过使用通配符 `*`实现的:

```
<form:errors path="*" />
```

### 9.1。验证器

要显示给定字段的错误，我们需要定义一个验证器:

```
public class PersonValidator implements Validator {

    @Override
    public boolean supports(Class clazz) {
        return Person.class.isAssignableFrom(clazz);
    }

    @Override
    public void validate(Object obj, Errors errors) {
        ValidationUtils.rejectIfEmptyOrWhitespace(errors, "name", "required.name");
    }
}
```

在这种情况下，如果字段`name`为空，验证器将从资源包中返回由`required.name`标识的错误消息。

资源包在 Spring `XML`配置文件中定义如下:

```
<bean class="org.springframework.context.support.ResourceBundleMessageSource" id="messageSource">
     <property name="basename" value="messages" />
</bean>
```

或者在纯 Java 配置风格中:

```
@Bean
public MessageSource messageSource() {
    ResourceBundleMessageSource messageSource = new ResourceBundleMessageSource();
    messageSource.setBasenames("messages");
    return messageSource;
}
```

错误信息在`messages.properties`文件中定义:

```
required.name = Name is required!
```

为了应用这个验证，我们需要在我们的控制器中包含一个对验证器的引用，并在用户提交表单时调用控制器方法中的方法`validate`:

```
@RequestMapping(value = "/addPerson", method = RequestMethod.POST)
public String submit(
  @ModelAttribute("person") Person person, 
  BindingResult result, 
  ModelMap modelMap) {

    validator.validate(person, result);

    if (result.hasErrors()) {
        return "personForm";
    }

    modelMap.addAttribute("person", person);
    return "personView";
}
```

### 9.2。JSR 303 豆验证

从 Spring 3 开始，我们可以使用 **JSR 303** (通过`@Valid`注释)进行 bean 验证。为此，我们需要在类路径上有一个 **JSR303 验证器框架**。我们将使用 **Hibernate 验证器**(参考实现)。下面是我们需要包含在 POM 中的依赖关系:

```
<dependency>
    <groupId>org.hibernate</groupId>
    <artifactId>hibernate-validator</artifactId>
    <version>5.1.1.Final</version>
</dependency>
```

为了让 Spring MVC 通过`@Valid` 注释支持 JSR 303 验证，我们需要在我们的 Spring 配置文件中启用以下内容:

```
<mvc:annotation-driven/>
```

或者在 Java 配置中使用相应的注释`@EnableWebMvc`:

```
@EnableWebMvc
@Configuration
public class ClientWebConfigJava implements WebMvcConfigurer {
    // All web configuration will go here
}
```

接下来，我们需要对控制器方法 进行注释，我们要用`@Valid`注释 验证 :

```
@RequestMapping(value = "/addPerson", method = RequestMethod.POST)
public String submit(
  @Valid @ModelAttribute("person") Person person, 
  BindingResult result, 
  ModelMap modelMap) {

    if(result.hasErrors()) {
        return "personForm";
    }

    modelMap.addAttribute("person", person);
    return "personView";
}
```

现在，我们可以用 Hibernate validator 注释来注释实体的属性以验证它:

```
@NotEmpty
private String password;
```

默认情况下，如果我们将密码输入字段留空，该注释将显示`“may not be empty”`。

我们可以通过在验证器示例中定义的资源包中创建一个属性来覆盖默认的错误消息。消息的密钥遵循规则`AnnotationName.entity.fieldname`:

```
NotEmpty.person.password = Password is required!
```

## 10。结论

在本教程中，我们探索了 Spring 为处理表单提供的各种标签。

我们还看了显示验证错误的标签，以及显示定制错误消息所需的配置。

以上所有例子都可以在一个 [GitHub 项目](https://web.archive.org/web/20220626102933/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-mvc-xml-2)中找到。这是一个基于 Eclipse 的项目，因此应该很容易导入和运行。

当项目在本地运行时，可以在以下位置访问表单示例:

[http://localhost:8080/spring-MVC-XML/person](https://web.archive.org/web/20220626102933/http://localhost:8080/spring-mvc-xml/person)