# 春季注入系列

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-injecting-collections>

## 1。简介

在本教程中，我们将展示如何使用 Spring 框架来**注入`Java`集合。**

简单地说，我们将用`List, Map, Set`集合接口来演示例子。

## 2。`List`同`@Autowired`

让我们创建一个示例 bean:

```java
public class CollectionsBean {

    @Autowired
    private List<String> nameList;

    public void printNameList() {
        System.out.println(nameList);
    }
}
```

这里，我们声明了`nameList`属性来保存`String`值的`List`。

**在这个例子中，我们对`nameList`使用场注入。因此，我们把`@Autowired`标注为**。

要了解更多关于依赖注入或实现它的不同方法，请查看本[指南](/web/20221126224914/https://www.baeldung.com/inversion-control-and-dependency-injection-in-spring)。

之后，我们在配置设置类中注册`CollectionsBean `:

```java
@Configuration
public class CollectionConfig {

    @Bean
    public CollectionsBean getCollectionsBean() {
        return new CollectionsBean();
    }

    @Bean
    public List<String> nameList() {
        return Arrays.asList("John", "Adam", "Harry");
    }
}
```

除了注册`CollectionsBean`，我们还通过显式初始化注入一个新的列表，并将其作为一个单独的`@Bean`配置返回。

现在，我们可以测试结果:

```java
ApplicationContext context = new AnnotationConfigApplicationContext(CollectionConfig.class);
CollectionsBean collectionsBean = context.getBean(
  CollectionsBean.class);
collectionsBean.printNameList();
```

printNameList()方法的输出:

```java
[John, Adam, Harry]
```

## 3。`Set`用构造函数注入

要用`Set`集合设置相同的示例，让我们修改`CollectionsBean `类:

```java
public class CollectionsBean {

    private Set<String> nameSet;

    public CollectionsBean(Set<String> strings) {
        this.nameSet = strings;
    }

    public void printNameSet() {
        System.out.println(nameSet);
    }
}
```

**这次我们想使用构造函数注入来初始化`nameSet`属性**。这也需要改变配置类别:

```java
@Bean
public CollectionsBean getCollectionsBean() {
    return new CollectionsBean(new HashSet<>(Arrays.asList("John", "Adam", "Harry")));
}
```

## 4。`Map`用二传手注射

按照同样的逻辑，让我们添加` nameMap`字段来演示 map 注入:

```java
public class CollectionsBean {

    private Map<Integer, String> nameMap;

    @Autowired
    public void setNameMap(Map<Integer, String> nameMap) {
        this.nameMap = nameMap;
    }

    public void printNameMap() {
        System.out.println(nameMap);
    }
}
```

这次**我们有一个 setter 方法，以便使用 setter 依赖注入**。我们还需要在配置类中添加`Map`初始化代码:

```java
@Bean
public Map<Integer, String> nameMap(){
    Map<Integer, String>  nameMap = new HashMap<>();
    nameMap.put(1, "John");
    nameMap.put(2, "Adam");
    nameMap.put(3, "Harry");
    return nameMap;
}
```

调用`printNameMap()`方法后的结果:

```java
{1=John, 2=Adam, 3=Harry}
```

## 5。注入 Bean 引用

让我们看一个例子，在这个例子中，我们注入 bean 引用作为集合的元素。

首先，让我们创建 bean:

```java
public class BaeldungBean {

    private String name;

    // constructor
}
```

并将`BaeldungBean`的一个`List`作为属性添加到`CollectionsBean `类:

```java
public class CollectionsBean {

    @Autowired(required = false)
    private List<BaeldungBean> beanList;

    public void printBeanList() {
        System.out.println(beanList);
    }
}
```

接下来，我们为每个`BaeldungBean`元素添加 Java 配置工厂方法:

```java
@Configuration
public class CollectionConfig {

    @Bean
    public BaeldungBean getElement() {
        return new BaeldungBean("John");
    }

    @Bean
    public BaeldungBean getAnotherElement() {
        return new BaeldungBean("Adam");
    }

    @Bean
    public BaeldungBean getOneMoreElement() {
        return new BaeldungBean("Harry");
    }

    // other factory methods
}
```

**Spring 容器将`BaeldungBean`类型的各个 beans 注入到一个集合中。**

为了测试这一点，我们调用了`collectionsBean.printBeanList()`方法。输出将 bean 名称显示为列表元素:

```java
[John, Harry, Adam]
```

现在，**让我们考虑一个没有`BaeldungBean`** 的场景。如果没有在应用程序上下文中注册的`BaeldungBean`，Spring 将抛出一个异常，因为缺少必需的依赖项。

我们可以使用`@Autowired(required = false)`将依赖关系标记为可选的。不抛出异常，`beanList`不会被初始化，它的值将保持`null`。

如果我们需要一个空列表来代替`null,`，我们可以用一个新的`ArrayList:`来初始化`beanList`

```java
@Autowired(required = false)
private List<BaeldungBean> beanList = new ArrayList<>();
```

### 5.1。使用`@Order`分类豆子

**我们可以在注入集合**时指定 beans 的顺序。

为此，我们使用`@Order`注释并指定索引:

```java
@Configuration
public class CollectionConfig {

    @Bean
    @Order(2)
    public BaeldungBean getElement() {
        return new BaeldungBean("John");
    }

    @Bean
    @Order(3)
    public BaeldungBean getAnotherElement() {
        return new BaeldungBean("Adam");
    }

    @Bean
    @Order(1)
    public BaeldungBean getOneMoreElement() {
        return new BaeldungBean("Harry");
    }
}
```

**Spring 容器首先将注入名为`“Harry”`** 的 bean，因为它具有最低的顺序值。

然后，它将注入`“John”,`，最后是`“Adam”` bean:

```java
[Harry, John, Adam]
```

在[指南](/web/20221126224914/https://www.baeldung.com/spring-order)中了解更多关于`@Order`的信息。

### 5.2。使用`@Qualifier`选择豆子

**我们可以使用`@Qualifier`来选择要注入到与`@Qualifier`名称匹配的特定集合中的 beans。**

下面是我们如何将它用于注入点:

```java
@Autowired
@Qualifier("CollectionsBean")
private List<BaeldungBean> beanList;
```

然后，我们用相同的`@Qualifier`标记我们想要注入到`List`中的 beans:

```java
@Configuration
public class CollectionConfig {

    @Bean
    @Qualifier("CollectionsBean")
    public BaeldungBean getElement() {
        return new BaeldungBean("John");
    }

    @Bean
    public BaeldungBean getAnotherElement() {
        return new BaeldungBean("Adam");
    }

    @Bean
    public BaeldungBean getOneMoreElement() {
        return new BaeldungBean("Harry");
    }

    // other factory methods
}
```

在这个例子中，我们指定名为*“约翰”*的 bean 将被注入到名为`“CollectionsBean”`的`List`中。我们在这里测试的结果是:

```java
ApplicationContext context = new AnnotationConfigApplicationContext(CollectionConfig.class);
CollectionsBean collectionsBean = context.getBean(CollectionsBean.class);
collectionsBean.printBeanList();
```

从输出中，我们看到我们的集合只有一个元素:

```java
[John]
```

## 6。将空列表设置为默认值

我们可以通过使用`Collections.emptyList()`静态方法将注入列表属性的默认值设置为空列表:

```java
public class CollectionsBean {

    @Value("${names.list:}#{T(java.util.Collections).emptyList()}")
    private List<String> nameListWithDefaultValue;

    public void printNameListWithDefaults() {
        System.out.println(nameListWithDefaultValue);
    }
}
```

如果我们使用未通过属性文件初始化的“names.list”键运行此命令:

```java
collectionsBean.printNameListWithDefaults();
```

我们将得到一个空列表作为输出:

```java
[ ]
```

## 7。总结

通过这篇指南，我们学习了如何使用 Spring 框架注入不同类型的 Java 集合。

我们还研究了引用类型的注入，以及如何在集合中选择或排序它们。

像往常一样，完整的代码可以在 [GitHub 项目](https://web.archive.org/web/20221126224914/https://github.com/eugenp/tutorials/tree/master/spring-di-2)中获得。