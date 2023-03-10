# @春季订单

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-order>

## 1。概述

在本教程中，我们将学习关于春天的 [`@Order`](https://web.archive.org/web/20220626090103/https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/core/annotation/Order.html) 注释。**`@Order`注释定义了被注释组件或 bean 的排序顺序。**

它有一个可选的值参数，用于确定组件的顺序；默认值为`Ordered.LOWEST_PRECEDENCE`。这标志着该组件在所有其他有序组件中具有最低的优先级。

类似地，值`Ordered.HIGHEST_PRECEDENCE`可用于覆盖组件中的最高优先级。

## 2。何时使用`@Order`

在 Spring 4.0 之前，`@Order`注释只用于 AspectJ 执行顺序。这意味着最高级的建议将首先运行。

从 Spring 4.0 开始，它支持将注入组件排序到一个集合中。因此，Spring 将根据订单值注入相同类型的自动连接 beans。

让我们用一个简单的例子来探究一下。

## 3。如何使用`@Order`

首先，让我们用相关的接口和类来设置我们的项目。

### 3.1。界面创建

让我们创建决定产品评级的`Rating`界面:

```java
public interface Rating {
    int getRating();
}
```

### 3.2。组件创建

最后，让我们创建三个组件来定义一些产品的评级:

```java
@Component
@Order(1)
public class Excellent implements Rating {

    @Override
    public int getRating() {
        return 1;
    }
}

@Component
@Order(2)
public class Good implements Rating {

    @Override
    public int getRating() {
        return 2;
    }
}

@Component
@Order(Ordered.LOWEST_PRECEDENCE)
public class Average implements Rating {

    @Override
    public int getRating() {
        return 3;
    }
}
```

注意，`Average `类的优先级最低，因为它的值被覆盖了。

## 4。测试我们的示例

到目前为止，我们已经创建了测试`@Order`注释所需的所有组件和接口。现在，让我们测试一下，以确认它是否如预期的那样工作:

```java
public class RatingRetrieverUnitTest { 

    @Autowired
    private List<Rating> ratings;

    @Test
    public void givenOrder_whenInjected_thenByOrderValue() {
        assertThat(ratings.get(0).getRating(), is(equalTo(1)));
        assertThat(ratings.get(1).getRating(), is(equalTo(2)));
        assertThat(ratings.get(2).getRating(), is(equalTo(3)));
    }
}
```

## 5。结论

在这篇简短的文章中，我们已经了解了`@Order`注释。我们可以在各种用例中找到`@Order`的应用——自动连线组件的顺序很重要。一个例子是 Spring 的请求过滤器。

由于它对注入优先级的影响，看起来它也可能影响单例启动顺序。但是相比之下，依赖关系和`@DependsOn `声明决定了单例启动顺序。

本教程中提到的所有例子都可以在 Github 上找到[。](https://web.archive.org/web/20220626090103/https://github.com/eugenp/tutorials/tree/master/spring-core-2)