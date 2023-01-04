# Spring 数据 MongoDB 中的自定义级联

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/cascading-with-dbref-and-lifecycle-events-in-spring-data-mongodb>

## 1。概述

本教程将继续探索 Spring Data MongoDB 的一些核心特性——`@DBRef`注释和生命周期事件。

## 2。`@DBRef`

映射框架不支持**存储父子关系**和其他文档中的嵌入文档。但是我们能做的是——我们可以分别存储它们，并使用一个`DBRef`来引用文档。

当从 MongoDB 加载对象时，这些引用将被急切地解析，我们将得到一个映射的对象，它看起来就像嵌入在我们的主文档中一样。

让我们看一些代码:

```java
@DBRef
private EmailAddress emailAddress; 
```

`EmailAddress`看起来像:

```java
@Document
public class EmailAddress {
    @Id
    private String id;

    private String value;

    // standard getters and setters
}
```

注意映射框架**不处理级联操作**。因此——例如——如果我们在父节点上触发一个`save`,子节点不会被自动保存——如果我们也想保存它，我们需要在子节点上显式触发保存。

这正是**生命周期事件派上用场的地方**。

## 3。生命周期事件

Spring Data MongoDB 发布了一些非常有用的生命周期事件——比如`onBeforeConvert, onBeforeSave, onAfterSave, onAfterLoad`和 `onAfterConvert.`

为了截取其中一个事件，我们需要注册一个`AbstractMappingEventListener`的子类，并覆盖这里的一个方法。当事件被调度时，我们的监听器将被调用，域对象被传入。

### 3.1。基本级联保存

让我们看看我们之前的例子——用`emailAddress`保存`user`。我们现在可以监听在一个域对象进入转换器之前被调用的`onBeforeConvert`事件:

```java
public class UserCascadeSaveMongoEventListener extends AbstractMongoEventListener<Object> {
    @Autowired
    private MongoOperations mongoOperations;

    @Override
    public void onBeforeConvert(BeforeConvertEvent<Object> event) { 
        Object source = event.getSource(); 
        if ((source instanceof User) && (((User) source).getEmailAddress() != null)) { 
            mongoOperations.save(((User) source).getEmailAddress());
        }
    }
}
```

现在我们只需要将监听器注册到`MongoConfig`:

```java
@Bean
public UserCascadeSaveMongoEventListener userCascadingMongoEventListener() {
    return new UserCascadeSaveMongoEventListener();
}
```

或者作为 XML:

```java
<bean class="org.baeldung.event.UserCascadeSaveMongoEventListener" />
```

我们已经完成了级联语义——尽管只是针对用户。

### 3.2。通用级联实现

现在让我们通过**使级联功能通用化来改进前面的解决方案。**让我们从定义一个自定义注释开始:

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.FIELD)
public @interface CascadeSave {
    //
}
```

现在**让我们使用自定义监听器**来处理这些字段，而不必强制转换为任何特定的实体:

```java
public class CascadeSaveMongoEventListener extends AbstractMongoEventListener<Object> {

    @Autowired
    private MongoOperations mongoOperations;

    @Override
    public void onBeforeConvert(BeforeConvertEvent<Object> event) { 
        Object source = event.getSource(); 
        ReflectionUtils.doWithFields(source.getClass(), 
          new CascadeCallback(source, mongoOperations));
    }
}
```

因此，我们使用 Spring 的反射实用程序，并对符合我们标准的所有字段运行回调:

```java
@Override
public void doWith(Field field) throws IllegalArgumentException, IllegalAccessException {
    ReflectionUtils.makeAccessible(field);

    if (field.isAnnotationPresent(DBRef.class) && 
      field.isAnnotationPresent(CascadeSave.class)) {

        Object fieldValue = field.get(getSource());
        if (fieldValue != null) {
            FieldCallback callback = new FieldCallback();
            ReflectionUtils.doWithFields(fieldValue.getClass(), callback);

            getMongoOperations().save(fieldValue);
        }
    }
}
```

如您所见，我们正在寻找既有`DBRef`注释又有`CascadeSave`注释的字段。一旦我们找到这些字段，我们保存子实体。

让我们看看`FieldCallback`类，我们用它来检查孩子是否有一个`@Id`注释:

```java
public class FieldCallback implements ReflectionUtils.FieldCallback {
    private boolean idFound;

    public void doWith(Field field) throws IllegalArgumentException, IllegalAccessException {
        ReflectionUtils.makeAccessible(field);

        if (field.isAnnotationPresent(Id.class)) {
            idFound = true;
        }
    }

    public boolean isIdFound() {
        return idFound;
    }
}
```

最后，为了让它们一起工作，我们当然需要对`emailAddress`字段进行正确的注释:

```java
@DBRef
@CascadeSave
private EmailAddress emailAddress;
```

### 3.3。级联测试

现在让我们来看一个场景——我们用`emailAddress`保存一个`User`,保存操作自动级联到这个嵌入的实体:

```java
User user = new User();
user.setName("Brendan");
EmailAddress emailAddress = new EmailAddress();
emailAddress.setValue("[[email protected]](/web/20220625230847/https://www.baeldung.com/cdn-cgi/l/email-protection)");
user.setEmailAddress(emailAddress);
mongoTemplate.insert(user); 
```

让我们检查一下我们的数据库:

```java
{
    "_id" : ObjectId("55cee9cc0badb9271768c8b9"),
    "name" : "Brendan",
    "age" : null,
    "email" : {
        "value" : "[[email protected]](/web/20220625230847/https://www.baeldung.com/cdn-cgi/l/email-protection)"
    }
}
```

## 4。结论

在本文中，我们展示了 Spring Data MongoDB 的一些很酷的特性——`@DBRef`注释、生命周期事件以及我们如何智能地处理级联。

所有这些例子和代码片段的实现**可以在 GitHub** 上找到[——这是一个基于 Maven 的项目，所以它应该很容易导入和运行。](https://web.archive.org/web/20220625230847/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-data-mongodb)