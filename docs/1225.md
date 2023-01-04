# 使用 Spring 数据 JPA 进行分页和排序

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-data-jpa-pagination-sorting>

## 1.概观

当我们有一个大的数据集，并且我们想把它以较小的块呈现给用户时，分页通常是很有帮助的。

此外，在分页时，我们经常需要根据一些标准对数据进行排序。

在本教程中，我们将学习如何使用 Spring 数据 JPA 轻松地分页和排序。

## 延伸阅读:

## [春季数据 JPA @Query](/web/20221129015157/https://www.baeldung.com/spring-data-jpa-query)

Learn how to use the @Query annotation in Spring Data JPA to define custom queries using JPQL and native SQL.[Read more](/web/20221129015157/https://www.baeldung.com/spring-data-jpa-query) →

## [Spring 数据 JPA 库中派生的查询方法](/web/20221129015157/https://www.baeldung.com/spring-data-derived-queries)

Explore the query derivation mechanism in Spring Data JPA.[Read more](/web/20221129015157/https://www.baeldung.com/spring-data-derived-queries) →

## 2.初始设置

首先，假设我们有一个`Product`实体作为我们的域类:

```
@Entity 
public class Product {

    @Id
    private long id;
    private String name;
    private double price; 

    // constructors, getters and setters 

}
```

我们的每个`Product`实例都有一个惟一的标识符:`id`，它的`name`和与之相关联的`price`。

## 3.创建存储库

要访问我们的`Product`，我们需要一个`ProductRepository`:

```
public interface ProductRepository extends PagingAndSortingRepository<Product, Integer> {

    List<Product> findAllByPrice(double price, Pageable pageable);
}
```

通过扩展 **[`PagingAndSortingRepository`](https://web.archive.org/web/20221129015157/https://docs.spring.io/spring-data/data-commons/docs/current/api/org/springframework/data/repository/PagingAndSortingRepository.html) ，我们得到了用于分页和排序的`findAll(Pageable pageable)` 和 `findAll(Sort sort)` 方法。**

相反，我们可以选择扩展 [`JpaRepository`](/web/20221129015157/https://www.baeldung.com/spring-data-repositories) ，因为它也扩展了`PagingAndSortingRepository`。

一旦我们扩展了`PagingAndSortingRepository`、**，我们就可以添加自己的方法，将`Pageable`和`Sort` 作为参数**，就像我们在这里对`findAllByPrice`所做的那样。

让我们看看如何使用我们的新方法对我们的`Product`进行分页。

## 4.页码

一旦我们的存储库从`PagingAndSortingRepository`开始扩展，我们只需要:

1.  创建或获取一个`PageRequest`对象，它是`Pageable`接口的一个实现
2.  将`PageRequest`对象作为参数传递给我们打算使用的存储库方法

我们可以通过传入请求的页码和页面大小来创建一个`PageRequest`对象。

这里**页面计数从零开始:**

```
Pageable firstPageWithTwoElements = PageRequest.of(0, 2);

Pageable secondPageWithFiveElements = PageRequest.of(1, 5);
```

在 Spring MVC 中，我们也可以选择使用 [Spring Data Web Support](https://web.archive.org/web/20221129015157/https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#core.web) 来获取控制器中的`Pageable`实例。

一旦我们有了我们的`PageRequest`对象，我们就可以在调用我们的存储库的方法时传递它:

```
Page<Product> allProducts = productRepository.findAll(firstPageWithTwoElements);

List<Product> allTenDollarProducts = 
  productRepository.findAllByPrice(10, secondPageWithFiveElements);
```

默认情况下，`findAll(Pageable pageable)`方法返回一个`Page<T>`对象。

然而，**我们可以选择从任何返回分页数据**的自定义方法中返回一个`Page<T>,` 、一个 `Slice<T>,`或一个`List<T>`。

一个`Page<T>`实例，除了拥有一个`Product`列表之外，还知道可用页面的总数。**它触发一个额外的计数查询来实现它。为了避免这样的开销，我们可以返回一个`Slice<T>`或者一个`List<T>`。**

A `Slice`只知道下一个切片是否可用。

## 5.分页和排序

类似地，为了对查询结果进行排序，我们可以简单地通过[将`Sort`](/web/20221129015157/https://www.baeldung.com/spring-data-sorting) 的实例传递给方法:

```
Page<Product> allProductsSortedByName = productRepository.findAll(Sort.by("name"));
```

然而，如果我们想对数据进行排序和分页，该怎么办呢？

我们可以通过将排序细节传递给我们的`PageRequest`对象本身来做到这一点:

```
Pageable sortedByName = 
  PageRequest.of(0, 3, Sort.by("name"));

Pageable sortedByPriceDesc = 
  PageRequest.of(0, 3, Sort.by("price").descending());

Pageable sortedByPriceDescNameAsc = 
  PageRequest.of(0, 5, Sort.by("price").descending().and(Sort.by("name")));
```

基于我们的排序需求，**我们可以在创建`PageRequest`实例时指定排序字段和排序方向**。

像往常一样，我们可以将这个`Pageable`类型的实例传递给存储库的方法。

## 6.结论

在本文中，我们学习了如何在 Spring Data JPA 中对查询结果进行分页和排序。

与往常一样，本文中使用的完整代码示例可以在 Github 上的[处获得。](https://web.archive.org/web/20221129015157/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-data-jpa-enterprise-2)