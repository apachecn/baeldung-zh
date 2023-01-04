# 使用@RepositoryEventHandler 的 Spring 数据休息事件

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-data-rest-events>

## 1.介绍

在处理实体时，REST exporter 处理创建、保存和删除事件的操作。**我们可以使用一个`ApplicationListener`来监听这些事件，并在特定动作被执行时执行一个函数。**

或者，我们可以**使用基于域类型过滤事件的带注释的处理程序。**

## 2.编写带注释的处理程序

`ApplicationListener`不区分实体类型；但是**使用带注释的处理程序，我们可以基于域类型**过滤事件。

我们可以通过在 POJO 上添加`@RepositoryEventHandler`注释来声明基于注释的事件处理程序。结果，这通知`BeanPostProcessor`需要检查 POJO 的处理程序方法。

在下面的例子中，我们用对应于实体`Author – `的`RepositoryEventHandler` 来注释该类，并在`AuthorEventHandler`类中声明与对应于实体`Author`的不同之前和之后事件相关的方法:

```
@RepositoryEventHandler(Author.class) 
public class AuthorEventHandler {
    Logger logger = Logger.getLogger("Class AuthorEventHandler");

    @HandleBeforeCreate
    public void handleAuthorBeforeCreate(Author author){
        logger.info("Inside Author Before Create....");
        String name = author.getName();
    }

    @HandleAfterCreate
    public void handleAuthorAfterCreate(Author author){
        logger.info("Inside Author After Create ....");
        String name = author.getName();
    }

    @HandleBeforeDelete
    public void handleAuthorBeforeDelete(Author author){
        logger.info("Inside Author Before Delete ....");
        String name = author.getName();
    }

    @HandleAfterDelete
    public void handleAuthorAfterDelete(Author author){
        logger.info("Inside Author After Delete ....");
        String name = author.getName();
    }
}
```

这里，根据对`Author`实体执行的操作，调用`AuthorEventHandler` 类的不同方法。

在找到带有`@RepositoryEventHandler`注释的类时，Spring 遍历该类中的方法，以找到对应于下面提到的 before 和 after 事件的注释:

**`Before*`事件注释—**在事件被调用之前调用与 before 关联的注释。

*   `BeforeCreateEvent`
*   `BeforeDeleteEvent`
*   `BeforeSaveEvent`
*   `BeforeLinkSaveEvent`

**`After*`事件注解——事件被调用后，与事件注解关联的**被调用。

*   `AfterLinkSaveEvent`
*   `AfterSaveEvent`
*   `AfterCreateEvent`
*   `AfterDeleteEvent`

我们也可以在一个类中声明对应于相同事件类型的不同实体类型的方法:

```
@RepositoryEventHandler
public class BookEventHandler {

    @HandleBeforeCreate
    public void handleBookBeforeCreate(Book book){
        // code for before create book event
    }

    @HandleBeforeCreate
    public void handleAuthorBeforeCreate(Author author){
        // code for before create author event
    }
}
```

这里，`BookEventHandler `类处理不止一个实体。在找到带有`@RepositoryEventHandler `注释的类时，它遍历这些方法，并在相应的 create 事件之前调用相应的实体。

此外，我们需要在`@Configuration`类中声明事件处理程序，该类将检查 bean 中的处理程序，并将它们与正确的事件进行匹配:

```
@Configuration
public class RepositoryConfiguration{

    public RepositoryConfiguration(){
        super();
    }

    @Bean
    AuthorEventHandler authorEventHandler() {
        return new AuthorEventHandler();
    }

    @Bean
    BookEventHandler bookEventHandler(){
        return new BookEventHandler();
    }
}
```

## 3.结论

总之，这是对实施和理解`@RepositoryEventHandler.` 的介绍

在这个快速教程中，我们学习了如何实现`@RepositoryEventHandler`注释来处理对应于实体类型的各种事件。

和往常一样，可以在 Github 上找到完整的代码示例[。](https://web.archive.org/web/20220703143656/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-data-rest)