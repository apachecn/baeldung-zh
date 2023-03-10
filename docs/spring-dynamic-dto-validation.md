# 从数据库中检索到动态 DTO 验证配置

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-dynamic-dto-validation>

## 1。概述

在本教程中，我们将了解如何使用从数据库中检索到的正则表达式来匹配字段值，从而**创建一个定制的验证注释。**

我们将使用 Hibernate Validator 作为基本实现。

## 2。Maven 依赖关系

对于开发，我们将需要以下依赖项:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
    <version>2.4.0</version>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
    <version>2.4.0</version>
</dependency>
```

最新版本的[spring-boot-starter-百里香叶](https://web.archive.org/web/20220524032545/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22spring-boot-starter-thymeleaf%22)、[spring-boot-starter-data-JPA](https://web.archive.org/web/20220524032545/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22spring-boot-starter-data-jpa%22)可以从 Maven Central 下载。

## 3。自定义验证注释

对于我们的例子，我们将创建一个名为`@ContactInfo`的定制注释，它将根据从数据库中检索到的正则表达式来验证一个值。然后，我们将在名为`Customer`的 POJO 类的`contactInfo`字段上应用这个验证。

为了从数据库中检索正则表达式，我们将把它们建模为一个`ContactInfoExpression`实体类。

### 3.1。数据模型和存储库

让我们用`id`和`contactInfo`字段创建`Customer`类:

```java
@Entity
public class Customer {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private long id;

    private String contactInfo;

    // standard constructor, getters, setters
}
```

接下来，让我们看看`ContactInfoExpression`类——它将在一个名为`pattern`的属性中保存正则表达式值:

```java
@Entity
public class ContactInfoExpression {

    @Id
    @Column(name="expression_type")
    private String type;

    private String pattern;

    //standard constructor, getters, setters
}
```

接下来，让我们添加一个基于 Spring 数据的存储库接口来操作`ContactInfoExpression`实体:

```java
public interface ContactInfoExpressionRepository 
  extends Repository<ContactInfoExpression, String> {

    Optional<ContactInfoExpression> findById(String id);
}
```

### 3.2。数据库设置

为了存储正则表达式，我们将使用一个具有以下持久性配置的`H2`内存数据库:

```java
@EnableJpaRepositories("com.baeldung.dynamicvalidation.dao")
@EntityScan("com.baeldung.dynamicvalidation.model")
@Configuration
public class PersistenceConfig {

    @Bean
    public DataSource dataSource() {
        EmbeddedDatabaseBuilder builder = new EmbeddedDatabaseBuilder();
        EmbeddedDatabase db = builder.setType(EmbeddedDatabaseType.H2)
          .addScript("schema-expressions.sql")
          .addScript("data-expressions.sql")
          .build();
        return db;
    }
}
```

提到的两个脚本用于创建模式并将数据插入到`contact_info_expression`表中:

```java
CREATE TABLE contact_info_expression(
  expression_type varchar(50) not null,
  pattern varchar(500) not null,
  PRIMARY KEY ( expression_type )
);
```

`data-expressions.sql`脚本将添加三条记录来表示类型`email`、`phone,` 和`website`。这些表示用于验证该值是有效电子邮件地址、有效美国电话号码还是有效 URL 的正则表达式:

```java
insert into contact_info_expression values ('email',
  '[a-z0-9!#$%&*+/=?^_`{|}~-]+(?:\.[a-z0-9!#$%&*+/=?^_`{|}~-]+)*@(?:[a-z0-9](?:[a-z0-9-]*[a-z0-9])?\.)+[a-z0-9](?:[a-z0-9-]*[a-z0-9])?')
insert into contact_info_expression values ('phone',
  '^([0-9]( |-)?)?(\(?[0-9]{3}\)?|[0-9]{3})( |-)?([0-9]{3}( |-)?[0-9]{4}|[a-zA-Z0-9]{7})

### 3.3。创建自定义验证器

让我们创建包含实际验证逻辑的`ContactInfoValidator`类。遵循 Java 验证规范指南，类**实现了`ConstraintValidator`接口**并覆盖了`isValid()`方法。

该类将获取当前使用的联系信息类型的值— `email`、`phone,`或`website` —该值在名为`contactInfoType`的属性中设置，然后使用它从数据库中检索正则表达式的值:

```
public class ContactInfoValidator implements ConstraintValidator<ContactInfo, String> {

    private static final Logger LOG = Logger.getLogger(ContactInfoValidator.class);

    @Value("${contactInfoType}")
    private String expressionType;

    private String pattern;

    @Autowired
    private ContactInfoExpressionRepository expressionRepository;

    @Override
    public void initialize(ContactInfo contactInfo) {
        if (StringUtils.isEmptyOrWhitespace(expressionType)) {
            LOG.error("Contact info type missing!");
        } else {
            pattern = expressionRepository.findById(expressionType)
              .map(ContactInfoExpression::getPattern).get();
        }
    }

    @Override
    public boolean isValid(String value, ConstraintValidatorContext context) {
        if (!StringUtils.isEmptyOrWhitespace(pattern)) {
            return Pattern.matches(pattern, value);
        }
        LOG.error("Contact info pattern missing!");
        return false;
    }
}
```java

可以在`application.properties`文件中将`contactInfoType`属性设置为`email`、`phone`或`website`中的一个值:

```
contactInfoType=email
```java

### 3.4。创建自定义约束注释

现在，让我们为自定义约束创建注释接口:

```
@Constraint(validatedBy = { ContactInfoValidator.class })
@Target({ METHOD, FIELD, ANNOTATION_TYPE, CONSTRUCTOR, PARAMETER })
@Retention(RetentionPolicy.RUNTIME)
public @interface ContactInfo {
    String message() default "Invalid value";

    Class<?>[] groups() default {};

    Class<? extends Payload>[] payload() default {};
}
```java

### 3.5。应用自定义约束

最后，让我们向我们的`Customer`类的`contactInfo`字段添加验证注释:

```
public class Customer {

    // ...
    @ContactInfo
    @NotNull
    private String contactInfo;

    // ...
}
```java

## 4。弹簧控制器和 HTML 表单

为了测试我们的验证注释，我们将创建一个 Spring MVC 请求映射，它使用`@Valid`注释来触发对`Customer`对象的验证:

```
@PostMapping("/customer")
public String validateCustomer(@Valid Customer customer, BindingResult result, Model model) {
    if (result.hasErrors()) {
        model.addAttribute("message", "The information is invalid!");
    } else {
        model.addAttribute("message", "The information is valid!");
    }
    return "customer";
}
```java

`Customer`对象从 HTML 表单发送到控制器:

```
<form action="customer" method="POST">
Contact Info: <input type="text" name="contactInfo" /> <br />
<input type="submit" value="Submit" />
</form>
<span th:text="${message}"></span>
```java

总而言之，我们可以将我们的应用程序作为 Spring Boot 应用程序运行:

```
@SpringBootApplication
public class DynamicValidationApp {
    public static void main(String[] args) {
        SpringApplication.run(DynamicValidationApp.class, args);
    }
}
```java

## 5。结论

在这个例子中，我们展示了如何创建一个定制的验证注释，从数据库中动态地检索一个正则表达式，并使用它来验证带注释的字段。

这个例子的完整源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220524032545/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-data-2))
insert into contact_info_expression values ('website',
  '^(http:\/\/www\.|https:\/\/www\.|http:\/\/|https:\/\/)?[a-z0-9]+([\-\.]{1}[a-z0-9]+)*\.[a-z]{2,5}(:[0-9]{1,5})?(\/.*)?

### 3.3。创建自定义验证器

让我们创建包含实际验证逻辑的`ContactInfoValidator`类。遵循 Java 验证规范指南，类**实现了`ConstraintValidator`接口**并覆盖了`isValid()`方法。

该类将获取当前使用的联系信息类型的值— `email`、`phone,`或`website` —该值在名为`contactInfoType`的属性中设置，然后使用它从数据库中检索正则表达式的值:

[PRE7]

可以在`application.properties`文件中将`contactInfoType`属性设置为`email`、`phone`或`website`中的一个值:

[PRE8]

### 3.4。创建自定义约束注释

现在，让我们为自定义约束创建注释接口:

[PRE9]

### 3.5。应用自定义约束

最后，让我们向我们的`Customer`类的`contactInfo`字段添加验证注释:

[PRE10]

## 4。弹簧控制器和 HTML 表单

为了测试我们的验证注释，我们将创建一个 Spring MVC 请求映射，它使用`@Valid`注释来触发对`Customer`对象的验证:

[PRE11]

`Customer`对象从 HTML 表单发送到控制器:

[PRE12]

总而言之，我们可以将我们的应用程序作为 Spring Boot 应用程序运行:

[PRE13]

## 5。结论

在这个例子中，我们展示了如何创建一个定制的验证注释，从数据库中动态地检索一个正则表达式，并使用它来验证带注释的字段。

这个例子的完整源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220524032545/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-data-2))
```

### 3.3。创建自定义验证器

让我们创建包含实际验证逻辑的`ContactInfoValidator`类。遵循 Java 验证规范指南，类**实现了`ConstraintValidator`接口**并覆盖了`isValid()`方法。

该类将获取当前使用的联系信息类型的值— `email`、`phone,`或`website` —该值在名为`contactInfoType`的属性中设置，然后使用它从数据库中检索正则表达式的值:

[PRE7]

可以在`application.properties`文件中将`contactInfoType`属性设置为`email`、`phone`或`website`中的一个值:

[PRE8]

### 3.4。创建自定义约束注释

现在，让我们为自定义约束创建注释接口:

[PRE9]

### 3.5。应用自定义约束

最后，让我们向我们的`Customer`类的`contactInfo`字段添加验证注释:

[PRE10]

## 4。弹簧控制器和 HTML 表单

为了测试我们的验证注释，我们将创建一个 Spring MVC 请求映射，它使用`@Valid`注释来触发对`Customer`对象的验证:

[PRE11]

`Customer`对象从 HTML 表单发送到控制器:

[PRE12]

总而言之，我们可以将我们的应用程序作为 Spring Boot 应用程序运行:

[PRE13]

## 5。结论

在这个例子中，我们展示了如何创建一个定制的验证注释，从数据库中动态地检索一个正则表达式，并使用它来验证带注释的字段。

这个例子的完整源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220524032545/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-data-2)