# 一个更好的刀的 Spring 自定义注释

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-annotation-bean-pre-processor>

## 1。概述

在本教程中，我们将使用 bean 后处理器实现一个定制的 Spring 注释**。**

那么这有什么帮助呢？简而言之——我们可以重用同一个 bean，而不必创建多个相同类型的相似 bean。

我们将在一个简单的项目中为 DAO 实现这样做——用一个单一的、灵活的`GenericDao`来替换它们。

## 2。肚子

我们需要`spring-core`、`spring-aop`和`spring-context-support`罐子来完成这项工作。我们可以在我们的`pom.xml`中声明`spring-context-support`。

```java
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context-support</artifactId>
    <version>5.2.2.RELEASE</version>
</dependency> 
```

如果你想获得 Spring 依赖的新版本——查看一下 maven 知识库。

## 3。新通用刀

大多数 Spring / JPA / Hibernate 实现使用标准的 DAO——通常每个实体一个。

我们将用一个`GenericDao`来代替那个解决方案；我们将编写一个定制的注释处理器，并使用那个`GenericDao`实现:

### 3.1。通用刀

```java
public class GenericDao<E> {

    private Class<E> entityClass;

    public GenericDao(Class<E> entityClass) {
        this.entityClass = entityClass;
    }

    public List<E> findAll() {
        // ...
    }

    public Optional<E> persist(E toPersist) {
        // ...
    }
} 
```

在真实的场景中，您当然需要连接一个 [PersistenceContext](https://web.archive.org/web/20220815044713/https://docs.oracle.com/javaee/7/api/javax/persistence/PersistenceContext.html) 并实际提供这些方法的实现。目前，我们将尽可能简单地解决这个问题。

现在，让我们为定制注入创建注释。

### 3.2。数据访问

```java
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.FIELD, ElementType.PARAMETER, ElementType.METHOD})
@Documented
public @interface DataAccess {
    Class<?> entity();
}
```

我们将使用上面的注释注入一个`GenericDao`,如下所示:

```java
@DataAccess(entity=Person.class)
private GenericDao<Person> personDao;
```

也许你们中的一些人会问，“Spring 如何识别我们的`DataAccess`注释？”。不会——默认情况下不会。

但是我们可以告诉 Spring 通过一个定制的`[BeanPostProcessor](https://web.archive.org/web/20220815044713/https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/beans/factory/config/BeanPostProcessor.html)`来识别注释——接下来让我们来实现它。

### 3.3。`DataAccessAnnotationProcessor`

```java
@Component
public class DataAccessAnnotationProcessor implements BeanPostProcessor {

    private ConfigurableListableBeanFactory configurableBeanFactory;

    @Autowired
    public DataAccessAnnotationProcessor(ConfigurableListableBeanFactory beanFactory) {
        this.configurableBeanFactory = beanFactory;
    }

    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) 
      throws BeansException {
        this.scanDataAccessAnnotation(bean, beanName);
        return bean;
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) 
      throws BeansException {
        return bean;
    }

    protected void scanDataAccessAnnotation(Object bean, String beanName) {
        this.configureFieldInjection(bean);
    }

    private void configureFieldInjection(Object bean) {
        Class<?> managedBeanClass = bean.getClass();
        FieldCallback fieldCallback = 
          new DataAccessFieldCallback(configurableBeanFactory, bean);
        ReflectionUtils.doWithFields(managedBeanClass, fieldCallback);
    }
} 
```

接下来——这是我们刚刚使用的`DataAccessFieldCallback`的实现:

### 3.4。`DataAccessFieldCallback`

```java
public class DataAccessFieldCallback implements FieldCallback {
    private static Logger logger = LoggerFactory.getLogger(DataAccessFieldCallback.class);

    private static int AUTOWIRE_MODE = AutowireCapableBeanFactory.AUTOWIRE_BY_NAME;

    private static String ERROR_ENTITY_VALUE_NOT_SAME = "@DataAccess(entity) "
            + "value should have same type with injected generic type.";
    private static String WARN_NON_GENERIC_VALUE = "@DataAccess annotation assigned "
            + "to raw (non-generic) declaration. This will make your code less type-safe.";
    private static String ERROR_CREATE_INSTANCE = "Cannot create instance of "
            + "type '{}' or instance creation is failed because: {}";

    private ConfigurableListableBeanFactory configurableBeanFactory;
    private Object bean;

    public DataAccessFieldCallback(ConfigurableListableBeanFactory bf, Object bean) {
        configurableBeanFactory = bf;
        this.bean = bean;
    }

    @Override
    public void doWith(Field field) 
    throws IllegalArgumentException, IllegalAccessException {
        if (!field.isAnnotationPresent(DataAccess.class)) {
            return;
        }
        ReflectionUtils.makeAccessible(field);
        Type fieldGenericType = field.getGenericType();
        // In this example, get actual "GenericDAO' type.
        Class<?> generic = field.getType(); 
        Class<?> classValue = field.getDeclaredAnnotation(DataAccess.class).entity();

        if (genericTypeIsValid(classValue, fieldGenericType)) {
            String beanName = classValue.getSimpleName() + generic.getSimpleName();
            Object beanInstance = getBeanInstance(beanName, generic, classValue);
            field.set(bean, beanInstance);
        } else {
            throw new IllegalArgumentException(ERROR_ENTITY_VALUE_NOT_SAME);
        }
    }

    public boolean genericTypeIsValid(Class<?> clazz, Type field) {
        if (field instanceof ParameterizedType) {
            ParameterizedType parameterizedType = (ParameterizedType) field;
            Type type = parameterizedType.getActualTypeArguments()[0];

            return type.equals(clazz);
        } else {
            logger.warn(WARN_NON_GENERIC_VALUE);
            return true;
        }
    }

    public Object getBeanInstance(
      String beanName, Class<?> genericClass, Class<?> paramClass) {
        Object daoInstance = null;
        if (!configurableBeanFactory.containsBean(beanName)) {
            logger.info("Creating new DataAccess bean named '{}'.", beanName);

            Object toRegister = null;
            try {
                Constructor<?> ctr = genericClass.getConstructor(Class.class);
                toRegister = ctr.newInstance(paramClass);
            } catch (Exception e) {
                logger.error(ERROR_CREATE_INSTANCE, genericClass.getTypeName(), e);
                throw new RuntimeException(e);
            }

            daoInstance = configurableBeanFactory.initializeBean(toRegister, beanName);
            configurableBeanFactory.autowireBeanProperties(daoInstance, AUTOWIRE_MODE, true);
            configurableBeanFactory.registerSingleton(beanName, daoInstance);
            logger.info("Bean named '{}' created successfully.", beanName);
        } else {
            daoInstance = configurableBeanFactory.getBean(beanName);
            logger.info(
              "Bean named '{}' already exists used as current bean reference.", beanName);
        }
        return daoInstance;
    }
} 
```

这是一个相当好的实现，但其中最重要的部分是`doWith()`方法:

```java
genericDaoInstance = configurableBeanFactory.initializeBean(beanToRegister, beanName);
configurableBeanFactory.autowireBeanProperties(genericDaoInstance, autowireMode, true);
configurableBeanFactory.registerSingleton(beanName, genericDaoInstance); 
```

这将告诉 Spring 基于运行时通过`@DataAccess`注释注入的对象来初始化 bean。

`beanName`将确保我们将获得 bean 的唯一实例，因为——在这种情况下——我们确实希望根据通过`@DataAccess`注释注入的实体创建单个对象`GenericDao`。

最后，接下来让我们在 Spring 配置中使用这个新的 bean 处理器。

### 3.5。`CustomAnnotationConfiguration`

```java
@Configuration
@ComponentScan("com.baeldung.springcustomannotation")
public class CustomAnnotationConfiguration {} 
```

这里重要的一点是,`[@ComponentScan](https://web.archive.org/web/20220815044713/https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/context/annotation/ComponentScan.html)`注释的值需要指向我们的定制 bean post 处理器所在的包，并确保它在运行时被 Spring 扫描和自动连接。

## 4。测试新刀

让我们从一个支持 Spring 的测试和两个简单的示例实体类开始吧—`Person`和`Account`。

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes={CustomAnnotationConfiguration.class})
public class DataAccessAnnotationTest {

    @DataAccess(entity=Person.class) 
    private GenericDao<Person> personGenericDao;
    @DataAccess(entity=Account.class) 
    private GenericDao<Account> accountGenericDao;
    @DataAccess(entity=Person.class) 
    private GenericDao<Person> anotherPersonGenericDao;

    ...
}
```

我们在`DataAccess`注释的帮助下注入了一些`GenericDao`的实例。为了测试新的 beans 是否被正确注入，我们需要覆盖:

1.  如果注射成功
2.  如果具有相同实体的 bean 实例是相同的
3.  如果`GenericDao`中的方法实际上如预期的那样工作

第一点实际上被 Spring 本身覆盖了——因为如果 bean 不能被连接进来，框架会很早抛出一个异常。

为了测试第 2 点，我们需要查看两个都使用了`Person`类的`GenericDao`实例:

```java
@Test
public void whenGenericDaoInjected_thenItIsSingleton() {
    assertThat(personGenericDao, not(sameInstance(accountGenericDao)));
    assertThat(personGenericDao, not(equalTo(accountGenericDao)));
    assertThat(personGenericDao, sameInstance(anotherPersonGenericDao));
}
```

我们不希望`personGenericDao`等于`accountGenericDao`。

但是我们确实希望`personGenericDao`和`anotherPersonGenericDao`是完全相同的实例。

为了测试第 3 点，我们在这里只测试一些简单的与持久性相关的逻辑:

```java
@Test
public void whenFindAll_thenMessagesIsCorrect() {
    personGenericDao.findAll();
    assertThat(personGenericDao.getMessage(), 
      is("Would create findAll query from Person"));

    accountGenericDao.findAll();
    assertThat(accountGenericDao.getMessage(), 
      is("Would create findAll query from Account"));
}

@Test
public void whenPersist_thenMessagesIsCorrect() {
    personGenericDao.persist(new Person());
    assertThat(personGenericDao.getMessage(), 
      is("Would create persist query from Person"));

    accountGenericDao.persist(new Account());
    assertThat(accountGenericDao.getMessage(), 
      is("Would create persist query from Account"));
} 
```

## 5。结论

在本文中，我们在 Spring 中实现了一个非常酷的自定义注释——还有一个`BeanPostProcessor`。总的目标是摆脱我们通常在持久层中拥有的多个 DAO 实现，并使用一个好的、简单的通用实现，而不会在这个过程中丢失任何东西。

所有这些示例和代码片段的实现**可以在** [**我的 GitHub 项目**](https://web.archive.org/web/20220815044713/https://github.com/eugenp/tutorials/tree/master/spring-core-3) 中找到——这是一个基于 Eclipse 的项目，所以它应该很容易导入并按原样运行。