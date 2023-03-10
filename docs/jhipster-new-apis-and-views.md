# 在 JHipster 中创建新的 API 和视图

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jhipster-new-apis-and-views>

## 1.介绍

在本教程中，我们将看到如何在一个 [JHipster](/web/20220627073805/https://www.baeldung.com/jhipster) 应用程序中创建一个新的 API。然后，我们将该 API 集成到前端显示中。

## 2.示例应用程序

在本教程中，我们将使用一个简单的书店应用程序。

书店建得像一个庞然大物。它使用[角度](/web/20220627073805/https://www.baeldung.com/spring-boot-angular-web)作为前端，并有一个名为`book`的实体，包含以下字段:

*   标题
*   作者
*   出版日期
*   价格
*   量

**JHipster 自动生成 API 和前端视图，为一本书提供简单的操作**:查看、创建、编辑、删除。

对于本教程，**我们将添加一个 API，让用户购买一本书**，以及前端的一个按钮，调用新的 API。

我们将只关注采购的 API 和前端方面。我们不会执行任何支付处理，只有最低限度的验证。

## 3.Spring Boot 的变化

JHipster 为创建新的[控制器](/web/20220627073805/https://www.baeldung.com/spring-controllers)提供了一个[生成器](https://web.archive.org/web/20220627073805/https://www.jhipster.tech/creating-a-spring-controller/)。然而，对于本教程，**我们将手动创建 API 和相关代码**。

### 3.1.资源类

第一步是更新生成的`BookResource`类。**我们添加前端代码**将调用的新端点:

```java
@GetMapping("/books/purchase/{id}")
public ResponseEntity<BookDTO> purchase(@PathVariable Long id) {
    Optional<BookDTO> bookDTO = bookService.purchase(id);
    return ResponseUtil.wrapOrNotFound(bookDTO);
}
```

这在`/books/purchase/{id}`创建了一个新的 API 端点。唯一的输入是图书的`id`，我们返回一个`BookDTO`，它将反映购买后的新库存水平。

### 3.2.服务接口和类

现在，**我们需要更新`BookService`接口来包含一个新的`purchase`方法:**

```java
Optional<BookDTO> purchase(Long id);
```

然后，我们需要在`BookServiceImpl`类中实现新方法:

```java
@Override
public Optional<BookDTO> purchase(Long id) {
    Optional<BookDTO> bookDTO = findOne(id);
    if (bookDTO.isPresent()) {
        int quantity = bookDTO.get().getQuantity();
        if (quantity > 0) {
            bookDTO.get().setQuantity(quantity - 1);
            Book book = bookMapper.toEntity(bookDTO.get());
            book = bookRepository.save(book);
            return bookDTO;
        }
        else {
            throw new BadRequestAlertException("Book is not in stock", "book", "notinstock");
        }
    }
    return Optional.empty();
}
```

让我们看看这段代码中发生了什么。**首先，我们通过书的`id`来查找这本书，以确认它的存在。如果没有，我们返回一个空的 [`Optional`](/web/20220627073805/https://www.baeldung.com/java-optional) 。**

如果它确实存在，那么我们确保它的库存水平大于零。否则，我们抛出一个`BadRequestAlertException.`虽然这个异常通常只在 JHipster 的 REST 层使用，但是我们在这里使用它来演示如何向前端返回有用的错误消息。

否则，如果股票大于零，那么我们将其减一，保存到存储库中，并返回更新后的 DTO。

### 3.3.安全配置

所需的最后一项更改在`SecurityConfiguration`类中:

```java
.antMatchers("/api/books/purchase/**").authenticated()
```

这确保了只有经过身份验证的用户才能调用我们的新 API。

## 4.前端变化

现在我们来关注一下前端的变化。JHipster 创建了一个视图来显示一本书，我们将在那里添加新的购买按钮。

### 4.1.服务级别

首先，**我们需要向现有的`book.service.ts`文件添加一个新方法。**这个文件已经包含了操作图书对象的方法，所以这是为我们的新 API 添加逻辑的好地方:

```java
purchase(id: number): Observable<EntityResponseType> {
    return this.http
        .get<IBook>(`${this.resourceUrl}/purchase/${id}`, { observe: 'response' })
        .pipe(map((res: EntityResponseType) => this.convertDateFromServer(res)));
}
```

### 4.2.成分

然后，**我们需要更新`book.component.ts`中的组件代码。**我们将创建一个函数来调用 Angular book 服务中的新方法，然后监听来自服务器的响应:

```java
purchase(id: number) {
    this.bookService.purchase(id).subscribe(
        (res: HttpResponse<IBook>) => {
            this.book = res.body;
        },
        (res: HttpErrorResponse) => console.log(res.message)
    );
}
```

### 4.3.视角

最后，**我们可以向图书视图**添加一个按钮，调用组件中的新购买方法:

```java
<button type="button"
             class="btn btn-primary"
             (click)="purchase(book.id)">
    <span>Purchase</span>
</button>
```

下图显示了前端的更新视图:

[![jhipster custom api frontend](img/79ed4805fcfa3806f317eda9036dcad4.png)](/web/20220627073805/https://www.baeldung.com/wp-content/uploads/2019/03/jhipster-custom-api-frontend-257x300-1.jpg)

单击 new `Purchase`按钮将导致调用我们的新 API，前端将自动更新新的股票状态(或者如果出错，显示一个错误)。

## 5.结论

在本教程中，我们看到了如何在 JHipster 中创建自定义 API，并将它们集成到前端。

我们首先将 API 和业务逻辑添加到 Spring Boot 中。然后，我们修改了前端代码，以利用新的 API 并显示结果。只需一点努力，我们就能够在 JHipster 自动生成的现有`CRUD`操作之上添加新的功能。

和往常一样，代码可以在 GitHub 的[上获得。](https://web.archive.org/web/20220627073805/https://github.com/eugenp/tutorials/tree/master/jhipster-5/bookstore-monolith)