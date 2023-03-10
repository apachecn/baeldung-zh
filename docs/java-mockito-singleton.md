# 用 Mockito 嘲笑一个单身者

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-mockito-singleton>

## 1.概观

在本教程中，我们将探索如何使用 [Mockito](/web/20221108223429/https://www.baeldung.com/mockito-mock-methods) 来模仿单例。

## 2.项目设置

我们将创建一个使用 [singleton](/web/20221108223429/https://www.baeldung.com/java-singleton) 的小项目，然后看看如何为使用该 singleton 的类编写测试。

### 2.1.依赖性–JUnit 和 Mockito

让我们首先将 [JUnit](https://web.archive.org/web/20221108223429/https://search.maven.org/search?q=g:org.junit.jupiter%20AND%20a:junit-jupiter-api) 和 [Mockito](https://web.archive.org/web/20221108223429/https://search.maven.org/search?q=g:org.mockito%20AND%20a:mockito-core) 依赖项添加到我们的`pom.xml`:

```java
<dependencies>
    <dependency>
        <groupId>org.junit.jupiter</groupId>
        <artifactId>junit-jupiter-api</artifactId>
        <version>5.9.1</version>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.mockito</groupId>
        <artifactId>mockito-core</artifactId>
        <version>4.8.1</version>
        <scope>test</scope>
    </dependency>
</dependencies> 
```

### 2.2.代码示例

我们将创建一个管理内存缓存的 singleton `CacheManager`:

```java
public class CacheManager {
    private final HashMap<String, Object> map;
    private static CacheManager instance;

    private CacheManager() {
        map = new HashMap<>();
    }

    public static CacheManager getInstance() {
        if(instance == null) {
            instance = new CacheManager();
        }
        return instance;
    }

    public <T> T getValue(String key, Class<T> clazz) {
        return clazz.cast(map.get(key));
    }

    public Object setValue(String key, Object value) {
        return map.put(key, value);
    }
}
```

为了简单起见，我们使用了更简单的单例实现，没有考虑多线程的情况。

接下来，我们将制作一个`ProduceService`:

```java
public class ProductService {

    private final ProductDAO productDAO;
    private final CacheManager cacheManager;

    public ProductService(ProductDAO productDAO) {
        this.productDAO = productDAO;
        this.cacheManager = CacheManager.getInstance();
    }

    public Product getProduct(String productName) {
        Product product = cacheManager.getValue(productName, Product.class);
        if (product == null) {
            product = productDAO.getProduct(productName);
        }

        return product;
    }
} 
```

`getProduct()`方法首先检查值是否存在于缓存中。如果没有，它调用[道](/web/20221108223429/https://www.baeldung.com/java-dao-pattern)来获取产品。

我们将为`getProduct()`方法编写一个测试。如果产品存在于缓存中，测试将检查是否没有对 DAO 的调用。为此，我们希望让`cacheManager.getValue()`方法在被调用时返回一个产品。

由于单例实例是由`static getInstance()`方法提供的，因此需要对其进行不同的模拟和注入。让我们来看几个方法。

## 3.使用另一个构造函数的解决方法

一种解决方法是向`ProductService`添加另一个构造函数，这使得注入单例`CacheManager`的模拟实例变得容易:

```java
public ProductService(ProductDAO productDAO, CacheManager cacheManager) {
    this.productDAO = productDAO;
    this.cacheManager = cacheManager;
} 
```

让我们编写一个测试，利用这个构造函数并使用 Mockito 模拟`CacheManager`:

```java
@Test
void givenValueExistsInCache_whenGetProduct_thenDAOIsNotCalled() {
    ProductDAO productDAO = mock(ProductDAO.class);
    CacheManager cacheManager = mock(CacheManager.class);
    Product product = new Product("product1", "description");

    when(cacheManager.getValue(any(), any())).thenReturn(product);

    ProductService productService = new ProductService(productDAO, cacheManager);
    productService.getProduct("product1");

    Mockito.verify(productDAO, times(0)).getProduct(any());
} 
```

这里需要注意几个要点:

*   我们模仿缓存管理器，并使用新的构造函数将其注入到`ProductService`中。
*   我们使用了`cacheManager.getValue()`方法来返回被调用的产品。
*   最后，我们验证了在调用`productService.getProduct()`方法的过程中没有调用`productDao.getProduct()`方法。

这很好，但是不推荐这样做。编写测试不应该要求我们在类中创建额外的方法或构造函数。

接下来，让我们看看另一种不需要改变被测试代码的方法。

## 4.用 Mockito-inline 嘲讽

模仿单例缓存管理器的另一种方式是模仿`static`方法`CacheManager.getInstance()`。 **Mockito-core 默认不支持`static`的嘲讽方式。然而，我们可以通过启用 Mockito-inline 扩展来模仿`static`方法。**

### 4.1.启用 Mockito-inline

用 Mockito 启用[模仿`static`方法的一种方式是添加](/web/20221108223429/https://www.baeldung.com/mockito-mock-static-methods) [Mockito-inline](https://web.archive.org/web/20221108223429/https://search.maven.org/search?q=g:org.mockito%20AND%20a:mockito-inline) 依赖项，而不是 Mockito-core:

```java
<dependency>
    <groupId>org.mockito</groupId>
    <artifactId>mockito-inline</artifactId>
    <version>4.8.1</version>
    <scope>test</scope>
</dependency> 
```

我们可以用这个依赖来代替`mockito-core`。

另一种方式是通过[激活内联模拟生成器](/web/20221108223429/https://www.baeldung.com/mockito-mock-static-methods#configuring-mockito-for-static-methods)。

### 4.2.修改测试

让我们对模拟`CacheManager`的测试做一些修改:

```java
@Test
void givenValueExistsInCache_whenGetProduct_thenDAOIsNotCalled_mockingStatic() {
    ProductDAO productDAO = mock(ProductDAO.class);
    CacheManager cacheManager = mock(CacheManager.class);
    Product product = new Product("product1", "description");

    try (MockedStatic<CacheManager> cacheManagerMock = mockStatic(CacheManager.class)) {
        cacheManagerMock.when(CacheManager::getInstance).thenReturn(cacheManager);
        when(cacheManager.getValue(any(), any())).thenReturn(product);

        ProductService productService = new ProductService(productDAO);
        productService.getProduct("product1");

        Mockito.verify(productDAO, times(0)).getProduct(any());
    }
} 
```

在上面的代码中需要注意几个要点:

*   我们使用方法`mockStatic()`创建了类`CacheManager.`的模拟版本
*   接下来，我们模仿`getInstance()`方法来返回我们模仿的`CacheManager`实例。
*   在模仿了`getInstance()`方法之后，我们创建了`ProductService`。当`ProductService` 的构造函数调用`getInstance(),` 时，被嘲笑的`CacheManager` 实例将被返回。

测试按预期执行，因为被模仿的缓存管理器返回了产品。

## 5.结论

在本文中，我们研究了几种使用 Mockito 为单体编写单元测试的方法。我们研究了一种基于构造函数的方法来传递模拟实例。然后我们看了使用 Mockito-inline 模仿`static getInstance()`方法。

和往常一样，本文中使用的代码示例可以在 GitHub 上找到[。](https://web.archive.org/web/20221108223429/https://github.com/eugenp/tutorials/tree/master/testing-modules/mockito-2)