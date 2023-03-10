# 用 Spring 实现简单的电子商务

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-angular-ecommerce>

## 1。我们的电子商务应用概述

在本教程中，我们将实现一个简单的电子商务应用程序。我们将开发一个使用 [Spring Boot](https://web.archive.org/web/20220627181507/https://spring.io/projects/spring-boot) 的 API 和一个使用[角度](https://web.archive.org/web/20220627181507/https://angular.io/)的客户端应用程序。

基本上，用户将能够从购物车中添加/删除产品列表中的产品，并下订单。

## 2.后端部分

为了开发 API，我们将使用最新版本的 Spring Boot。我们还将 JPA 和 H2 数据库用于事物的持久性方面。

**要了解关于 Spring Boot、** **的更多信息，您可以查看我们的 [Spring Boot 系列文章](/web/20220627181507/https://www.baeldung.com/spring-boot)** ，如果您想要**熟悉构建 REST API，请查看[另一系列文章](/web/20220627181507/https://www.baeldung.com/rest-with-spring-series/)** 。

### 2.1.Maven 依赖性

让我们准备我们的项目并将所需的依赖项导入到我们的`pom.xml`中。

我们需要一些核心的 Spring Boot 依赖关系:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
    <version>2.2.2.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <version>2.2.2.RELEASE</version>
</dependency> 
```

然后， [H2 数据库](https://web.archive.org/web/20220627181507/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22com.h2database%22%20AND%20a%3A%22h2%22):

```java
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <version>1.4.197</version>
    <scope>runtime</scope>
</dependency>
```

最后是杰克逊图书馆:

```java
<dependency>
    <groupId>com.fasterxml.jackson.datatype</groupId>
    <artifactId>jackson-datatype-jsr310</artifactId>
    <version>2.9.6</version>
</dependency>
```

我们已经使用了 [Spring Initializr](https://web.archive.org/web/20220627181507/https://start.spring.io/) 来快速建立具有所需依赖项的项目。

### 2.2.设置数据库

尽管我们可以在 Spring Boot 中使用现成的内存 H2 数据库，但在开始开发我们的 API 之前，我们仍然会做一些调整。

我们将在我们的`application.properties`文件**中**启用 H2 控制台**，这样我们就可以实际检查我们数据库的状态，看看是否一切都如我们所预期的那样**。

此外，在开发时将 SQL 查询记录到控制台也很有用:

```java
spring.datasource.name=ecommercedb
spring.jpa.show-sql=true

#H2 settings
spring.h2.console.enabled=true
spring.h2.console.path=/h2-console
```

添加这些设置后，我们将能够在`[http://localhost:8080/h2-console](https://web.archive.org/web/20220627181507/http://localhost:8080/h2-console)`使用`jdbc:h2:mem:ecommercedb `作为 JDBC URL 和没有密码的用户`sa `访问数据库。

### 2.3.项目结构

该项目将被组织成几个标准包，角度应用程序放在前端文件夹:

```java
├───pom.xml            
├───src
    ├───main
    │   ├───frontend
    │   ├───java
    │   │   └───com
    │   │       └───baeldung
    │   │           └───ecommerce
    │   │               │   EcommerceApplication.java
    │   │               ├───controller 
    │   │               ├───dto  
    │   │               ├───exception
    │   │               ├───model
    │   │               ├───repository
    │   │               └───service
    │   │                       
    │   └───resources
    │       │   application.properties
    │       ├───static
    │       └───templates
    └───test
        └───java
            └───com
                └───baeldung
                    └───ecommerce
                            EcommerceApplicationIntegrationTest.java
```

我们应该注意到，repository 包中的所有接口都很简单，并且扩展了 Spring Data 的 [CrudRepository](https://web.archive.org/web/20220627181507/https://docs.spring.io/spring-data/commons/docs/current/api/org/springframework/data/repository/CrudRepository.html) ，因此我们将省略在此显示它们。

### 2.4.异常处理

为了正确处理最终的异常，我们的 API 需要一个异常处理程序。

你可以在我们的[Spring 的 REST 错误处理](/web/20220627181507/https://www.baeldung.com/exception-handling-for-rest-with-spring)和【REST API 的自定义错误消息处理文章中找到关于这个主题的更多细节。

在这里，我们关注`ConstraintViolationException`和我们的客户`ResourceNotFoundException`:

```java
@RestControllerAdvice
public class ApiExceptionHandler {

    @SuppressWarnings("rawtypes")
    @ExceptionHandler(ConstraintViolationException.class)
    public ResponseEntity<ErrorResponse> handle(ConstraintViolationException e) {
        ErrorResponse errors = new ErrorResponse();
        for (ConstraintViolation violation : e.getConstraintViolations()) {
            ErrorItem error = new ErrorItem();
            error.setCode(violation.getMessageTemplate());
            error.setMessage(violation.getMessage());
            errors.addError(error);
        }
        return new ResponseEntity<>(errors, HttpStatus.BAD_REQUEST);
    }

    @SuppressWarnings("rawtypes")
    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ErrorItem> handle(ResourceNotFoundException e) {
        ErrorItem error = new ErrorItem();
        error.setMessage(e.getMessage());

        return new ResponseEntity<>(error, HttpStatus.NOT_FOUND);
    }
}
```

### 2.5.制品

**如果你需要更多关于春天坚持的知识，[春天坚持系列](/web/20220627181507/https://www.baeldung.com/persistence-with-spring-series/)** 里有很多有用的文章。

我们的应用程序将只支持从数据库中读取产品的**，所以我们需要先添加一些。**

让我们创建一个简单的`Product`类:

```java
@Entity
public class Product {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @NotNull(message = "Product name is required.")
    @Basic(optional = false)
    private String name;

    private Double price;

    private String pictureUrl;

    // all arguments contructor 
    // standard getters and setters
}
```

虽然用户没有机会通过应用程序添加产品，但我们将支持在数据库中保存产品，以便预填充产品列表。

一项简单的服务就足以满足我们的需求:

```java
@Service
@Transactional
public class ProductServiceImpl implements ProductService {

    // productRepository constructor injection

    @Override
    public Iterable<Product> getAllProducts() {
        return productRepository.findAll();
    }

    @Override
    public Product getProduct(long id) {
        return productRepository
          .findById(id)
          .orElseThrow(() -> new ResourceNotFoundException("Product not found"));
    }

    @Override
    public Product save(Product product) {
        return productRepository.save(product);
    }
}
```

一个简单的控制器将处理检索产品列表的请求:

```java
@RestController
@RequestMapping("/api/products")
public class ProductController {

    // productService constructor injection

    @GetMapping(value = { "", "/" })
    public @NotNull Iterable<Product> getProducts() {
        return productService.getAllProducts();
    }
}
```

为了向用户展示产品列表，我们现在需要做的就是将一些产品放入数据库。因此，我们将利用`CommandLineRunner`类在我们的主应用程序类中创建一个`Bean`。

这样，我们将在应用程序启动时将产品插入数据库:

```java
@Bean
CommandLineRunner runner(ProductService productService) {
    return args -> {
        productService.save(...);
        // more products
}
```

如果我们现在启动应用程序，我们可以通过`[http://localhost:8080/api/products](https://web.archive.org/web/20220627181507/http://localhost:8080/api/products).`检索产品列表。同样，如果我们转到`[http://localhost:8080/h2-console](https://web.archive.org/web/20220627181507/http://localhost:8080/h2-console)`并登录，我们会看到有一个名为`PRODUCT`的表，其中包含我们刚刚添加的产品。

### 2.6.命令

在 API 方面，我们需要启用 POST 请求来保存最终用户将要下的订单。

让我们首先创建模型:

```java
@Entity
@Table(name = "orders")
public class Order {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @JsonFormat(pattern = "dd/MM/yyyy")
    private LocalDate dateCreated;

    private String status;

    @JsonManagedReference
    @OneToMany(mappedBy = "pk.order")
    @Valid
    private List<OrderProduct> orderProducts = new ArrayList<>();

    @Transient
    public Double getTotalOrderPrice() {
        double sum = 0D;
        List<OrderProduct> orderProducts = getOrderProducts();
        for (OrderProduct op : orderProducts) {
            sum += op.getTotalPrice();
        }
        return sum;
    }

    @Transient
    public int getNumberOfProducts() {
        return this.orderProducts.size();
    }

    // standard getters and setters
}
```

这里我们应该注意一些事情。当然，最值得注意的事情之一是**记得更改我们的表的默认名称**。由于我们将类命名为`Order`，默认情况下应该创建名为`ORDER`的表。但是因为这是一个保留的 SQL 单词，我们添加了`@Table(name = “orders”)`以避免冲突。

此外，我们有两个 **`@Transient`方法将返回订单的总金额和订单中的产品数量**。两者都表示计算出的数据，所以不需要将其存储在数据库中。

最后，我们有一个代表订单详细信息的 **`@OneToMany`关系。为此，我们需要另一个实体类:**

```java
@Entity
public class OrderProduct {

    @EmbeddedId
    @JsonIgnore
    private OrderProductPK pk;

    @Column(nullable = false)
	private Integer quantity;

    // default constructor

    public OrderProduct(Order order, Product product, Integer quantity) {
        pk = new OrderProductPK();
        pk.setOrder(order);
        pk.setProduct(product);
        this.quantity = quantity;
    }

    @Transient
    public Product getProduct() {
        return this.pk.getProduct();
    }

    @Transient
    public Double getTotalPrice() {
        return getProduct().getPrice() * getQuantity();
    }

    // standard getters and setters

    // hashcode() and equals() methods
}
```

**我们这里有一个复合主键****:**

```java
@Embeddable
public class OrderProductPK implements Serializable {

    @JsonBackReference
    @ManyToOne(optional = false, fetch = FetchType.LAZY)
    @JoinColumn(name = "order_id")
    private Order order;

    @ManyToOne(optional = false, fetch = FetchType.LAZY)
    @JoinColumn(name = "product_id")
    private Product product;

    // standard getters and setters

    // hashcode() and equals() methods
}
```

这些类并不复杂，但是我们应该注意到在`OrderProduct`类中，我们将`@JsonIgnore`放在主键上。这是因为我们不想序列化主键的`Order`部分，因为这是多余的。

我们只需要将`Product`显示给用户，这就是为什么我们有 transient `getProduct()`方法。

接下来，我们需要一个简单的服务实现:

```java
@Service
@Transactional
public class OrderServiceImpl implements OrderService {

    // orderRepository constructor injection

    @Override
    public Iterable<Order> getAllOrders() {
        return this.orderRepository.findAll();
    }

    @Override
    public Order create(Order order) {
        order.setDateCreated(LocalDate.now());
        return this.orderRepository.save(order);
    }

    @Override
    public void update(Order order) {
        this.orderRepository.save(order);
    }
}
```

以及映射到`/api/orders`以处理`Order`请求的控制器。

最重要的是`create`()方法:

```java
@PostMapping
public ResponseEntity<Order> create(@RequestBody OrderForm form) {
    List<OrderProductDto> formDtos = form.getProductOrders();
    validateProductsExistence(formDtos);
    // create order logic
    // populate order with products

    order.setOrderProducts(orderProducts);
    this.orderService.update(order);

    String uri = ServletUriComponentsBuilder
      .fromCurrentServletMapping()
      .path("/orders/{id}")
      .buildAndExpand(order.getId())
      .toString();
    HttpHeaders headers = new HttpHeaders();
    headers.add("Location", uri);

    return new ResponseEntity<>(order, headers, HttpStatus.CREATED);
}
```

首先，**我们接受一个产品列表及其相应的数量**。之后，**我们检查数据库中是否存在所有产品**和**，然后创建并保存一个新订单**。我们保留了对新创建的对象的引用，这样我们可以向它添加订单细节。

最后，**我们创建一个“位置”头**。

详细的实现在存储库中——本文末尾提到了到它的链接。

## 3.前端

现在我们已经构建好了 Spring Boot 应用程序，是时候移动项目的棱角部分了。要做到这一点，我们首先要安装带有 NPM 的 [Node.js](https://web.archive.org/web/20220627181507/https://nodejs.org/en/) ，然后是一个 [Angular CLI](https://web.archive.org/web/20220627181507/https://cli.angular.io/) ，Angular 的命令行界面。

正如我们在官方文档中看到的那样，安装这两个软件真的很容易。

### 3.1.设置角度项目

正如我们提到的，我们将使用`Angular CLI`来创建我们的应用程序。为了简单起见，我们将 Angular 应用程序放在`/src/main/frontend`文件夹中。

要创建它，我们需要在`/src/main`文件夹中打开一个终端(或命令提示符)并运行:

```java
ng new frontend
```

这将创建我们的 Angular 应用程序所需的所有文件和文件夹。在文件`pakage.json`中，我们可以检查安装了哪些版本的依赖项。本教程基于 Angular v6.0.3，但旧版本应该可以完成这项工作，至少是版本 4.3 和更新版本(我们这里使用的`HttpClient`是在 Angular 4.3 中引入的)。

我们应该注意到**我们将从`/frontend`文件夹**中运行所有命令，除非另有说明。

该设置足以通过运行`ng serve`命令启动角度应用。默认情况下，它运行在`[http://localhost:4200](https://web.archive.org/web/20220627181507/http://localhost:4200/)`上，如果我们现在去那里，我们会看到加载了基本角度应用程序。

### 3.2.添加引导

在我们继续创建我们自己的组件之前，让我们先把`Bootstrap`添加到我们的项目中，这样我们可以让我们的页面看起来更好。

我们只需要几样东西就能实现这一点。**首先，我们需要** **运行一个命令来安装它**:

```java
npm install --save bootstrap
```

还有**然后要说到棱角实际使用它的**。为此，我们需要打开一个文件`src/main/frontend/angular.json`，在`“styles”`属性下添加`node_modules/bootstrap/dist/css/bootstrap.min.css `。仅此而已。

### 3.3.组件和型号

在我们开始为我们的应用程序创建组件之前，让我们先看看我们的应用程序实际上会是什么样子:

[![ecommerce](img/d3c7fbf9edd260008712c54620804f22.png)](/web/20220627181507/https://www.baeldung.com/wp-content/uploads/2018/08/ecommerce.png)

现在，我们将创建一个基础组件，名为`ecommerce`:

```java
ng g c ecommerce
```

这将在`/frontend/src/app`文件夹中创建我们的组件。**为了在应用程序启动时加载它，我们将** **将其** **包含到`app.component.html`** 中:

```java
<div class="container">
    <app-ecommerce></app-ecommerce>
</div>
```

接下来，我们将在这个基本组件中创建其他组件:

```java
ng g c /ecommerce/products
ng g c /ecommerce/orders
ng g c /ecommerce/shopping-cart
```

当然，如果愿意，我们可以手动创建所有这些文件夹和文件，但是在这种情况下，我们需要**记住在我们的`AppModule`** 中注册那些组件。

我们还需要一些模型来轻松处理我们的数据:

```java
export class Product {
    id: number;
    name: string;
    price: number;
    pictureUrl: string;

    // all arguments constructor
}
```

```java
export class ProductOrder {
    product: Product;
    quantity: number;

    // all arguments constructor
}
```

```java
export class ProductOrders {
    productOrders: ProductOrder[] = [];
}
```

最后提到的模型在后端与我们的`OrderForm`匹配。

### 3.4.基础组件

在我们的`ecommerce`组件的顶部，我们将在右边放置一个带有 Home 链接的导航栏:

```java
<nav class="navbar navbar-expand-lg navbar-dark bg-dark fixed-top">
    <div class="container">
        <a class="navbar-brand" href="#">Baeldung Ecommerce</a>
        <button class="navbar-toggler" type="button" data-toggle="collapse" 
          data-target="#navbarResponsive" aria-controls="navbarResponsive" 
          aria-expanded="false" aria-label="Toggle navigation" 
          (click)="toggleCollapsed()">
            <span class="navbar-toggler-icon"></span>
        </button>
        <div id="navbarResponsive" 
            [ngClass]="{'collapse': collapsed, 'navbar-collapse': true}">
            <ul class="navbar-nav ml-auto">
                <li class="nav-item active">
                    <a class="nav-link" href="#" (click)="reset()">Home
                        <span class="sr-only">(current)</span>
                    </a>
                </li>
            </ul>
        </div>
    </div>
</nav>
```

我们还将从这里加载其他组件:

```java
<div class="row">
    <div class="col-md-9">
        <app-products #productsC [hidden]="orderFinished"></app-products>
    </div>
    <div class="col-md-3">
        <app-shopping-cart (onOrderFinished)=finishOrder($event) #shoppingCartC 
          [hidden]="orderFinished"></app-shopping-cart>
    </div>
    <div class="col-md-6 offset-3">
        <app-orders #ordersC [hidden]="!orderFinished"></app-orders>
    </div>
</div>
```

我们应该记住，为了从我们的组件中看到内容，因为我们使用了`navbar`类，我们需要向`app.component.css`添加一些 CSS:

```java
.container {
    padding-top: 65px;
}
```

在我们评论最重要的部分之前，让我们检查一下`.ts`文件:

```java
@Component({
    selector: 'app-ecommerce',
    templateUrl: './ecommerce.component.html',
    styleUrls: ['./ecommerce.component.css']
})
export class EcommerceComponent implements OnInit {
    private collapsed = true;
    orderFinished = false;

    @ViewChild('productsC')
    productsC: ProductsComponent;

    @ViewChild('shoppingCartC')
    shoppingCartC: ShoppingCartComponent;

    @ViewChild('ordersC')
    ordersC: OrdersComponent;

    toggleCollapsed(): void {
        this.collapsed = !this.collapsed;
    }

    finishOrder(orderFinished: boolean) {
        this.orderFinished = orderFinished;
    }

    reset() {
        this.orderFinished = false;
        this.productsC.reset();
        this.shoppingCartC.reset();
        this.ordersC.paid = false;
    }
}
```

正如我们所见，点击`Home`链接将重置子组件。我们需要从父组件访问子组件中的方法和字段，所以这就是为什么我们保留对子组件的引用并在`reset()`方法中使用它们。

### 3.5.服务

为了让**兄弟组件相互通信** **以及从/向我们的 API** 检索/发送数据，我们需要创建一个服务:

```java
@Injectable()
export class EcommerceService {
    private productsUrl = "/api/products";
    private ordersUrl = "/api/orders";

    private productOrder: ProductOrder;
    private orders: ProductOrders = new ProductOrders();

    private productOrderSubject = new Subject();
    private ordersSubject = new Subject();
    private totalSubject = new Subject();

    private total: number;

    ProductOrderChanged = this.productOrderSubject.asObservable();
    OrdersChanged = this.ordersSubject.asObservable();
    TotalChanged = this.totalSubject.asObservable();

    constructor(private http: HttpClient) {
    }

    getAllProducts() {
        return this.http.get(this.productsUrl);
    }

    saveOrder(order: ProductOrders) {
        return this.http.post(this.ordersUrl, order);
    }

    // getters and setters for shared fields
}
```

我们可以注意到，这里有一些相对简单的东西。我们发出 GET 和 POST 请求来与 API 通信。此外，我们使我们需要在组件之间共享的数据可见，以便我们可以在以后订阅它。

然而，关于与 API 的通信，我们需要指出一点。如果我们现在运行这个应用程序，我们将会收到 404，并且检索不到任何数据。这样做的原因是，由于我们使用的是相对 URL，Angular 默认会尝试调用`[http://localhost:4200/api/products](https://web.archive.org/web/20220627181507/http://localhost:4200/api/products)`，而我们的后端应用程序运行在`localhost:8080`上。

当然，我们可以将 URL 硬编码到`localhost:8080`，但这不是我们想要做的。相反，**当处理不同的域时，我们应该在我们的`/frontend`文件夹**中创建一个名为`proxy-conf.json`的文件:

```java
{
    "/api": {
        "target": "http://localhost:8080",
        "secure": false
    }
}
```

然后我们需要**打开`package.json`并改变`scripts.start`属性**来匹配:

```java
"scripts": {
    ...
    "start": "ng serve --proxy-config proxy-conf.json",
    ...
  }
```

现在我们只需要**记住用`npm start`而不是`ng serve`** 启动应用程序。

### 3.6.制品

在我们的`ProductsComponent`中，我们将注入我们之前创建的服务，从 API 加载产品列表，并将其转换为`ProductOrders`的列表，因为我们希望为每个产品添加一个数量字段:

```java
export class ProductsComponent implements OnInit {
    productOrders: ProductOrder[] = [];
    products: Product[] = [];
    selectedProductOrder: ProductOrder;
    private shoppingCartOrders: ProductOrders;
    sub: Subscription;
    productSelected: boolean = false;

    constructor(private ecommerceService: EcommerceService) {}

    ngOnInit() {
        this.productOrders = [];
        this.loadProducts();
        this.loadOrders();
    }

    loadProducts() {
        this.ecommerceService.getAllProducts()
            .subscribe(
                (products: any[]) => {
                    this.products = products;
                    this.products.forEach(product => {
                        this.productOrders.push(new ProductOrder(product, 0));
                    })
                },
                (error) => console.log(error)
            );
    }

    loadOrders() {
        this.sub = this.ecommerceService.OrdersChanged.subscribe(() => {
            this.shoppingCartOrders = this.ecommerceService.ProductOrders;
        });
    }
}
```

我们还需要一个将产品添加到购物车或从购物车中删除产品的选项:

```java
addToCart(order: ProductOrder) {
    this.ecommerceService.SelectedProductOrder = order;
    this.selectedProductOrder = this.ecommerceService.SelectedProductOrder;
    this.productSelected = true;
}

removeFromCart(productOrder: ProductOrder) {
    let index = this.getProductIndex(productOrder.product);
    if (index > -1) {
        this.shoppingCartOrders.productOrders.splice(
            this.getProductIndex(productOrder.product), 1);
    }
    this.ecommerceService.ProductOrders = this.shoppingCartOrders;
    this.shoppingCartOrders = this.ecommerceService.ProductOrders;
    this.productSelected = false;
}
```

最后，我们将创建一个我们在 3.4 节提到的`reset`()方法:

```java
reset() {
    this.productOrders = [];
    this.loadProducts();
    this.ecommerceService.ProductOrders.productOrders = [];
    this.loadOrders();
    this.productSelected = false;
}
```

我们将遍历 HTML 文件中的产品列表，并将其显示给用户:

```java
<div class="row card-deck">
    <div class="col-lg-4 col-md-6 mb-4" *ngFor="let order of productOrders">
        <div class="card text-center">
            <div class="card-header">
                <h4>{{order.product.name}}</h4>
            </div>
            <div class="card-body">
                <a href="#"><img class="card-img-top" src={{order.product.pictureUrl}} 
                    alt=""></a>
                <h5 class="card-title">${{order.product.price}}</h5>
                <div class="row">
                    <div class="col-4 padding-0" *ngIf="!isProductSelected(order.product)">
                        <input type="number" min="0" class="form-control" 
                            [(ngModel)]=order.quantity>
                    </div>
                    <div class="col-4 padding-0" *ngIf="!isProductSelected(order.product)">
                        <button class="btn btn-primary" (click)="addToCart(order)"
                                [disabled]="order.quantity <= 0">Add To Cart
                        </button>
                    </div>
                    <div class="col-12" *ngIf="isProductSelected(order.product)">
                        <button class="btn btn-primary btn-block"
                                (click)="removeFromCart(order)">Remove From Cart
                        </button>
                    </div>
                </div>
            </div>
        </div>
    </div>
</div>
```

我们还将在相应的 CSS 文件中添加一个简单的类，这样一切都可以很好地适应:

```java
.padding-0 {
    padding-right: 0;
    padding-left: 1;
}
```

### 3.7.购物车

在`ShoppingCart`组件中，我们还将注入服务。我们将使用它来订阅`ProductsComponent`中的更改(注意产品何时被选择放入购物车)，然后更新购物车的内容并相应地重新计算总成本:

```java
export class ShoppingCartComponent implements OnInit, OnDestroy {
    orderFinished: boolean;
    orders: ProductOrders;
    total: number;
    sub: Subscription;

    @Output() onOrderFinished: EventEmitter<boolean>;

    constructor(private ecommerceService: EcommerceService) {
        this.total = 0;
        this.orderFinished = false;
        this.onOrderFinished = new EventEmitter<boolean>();
    }

    ngOnInit() {
        this.orders = new ProductOrders();
        this.loadCart();
        this.loadTotal();
    }

    loadTotal() {
        this.sub = this.ecommerceService.OrdersChanged.subscribe(() => {
            this.total = this.calculateTotal(this.orders.productOrders);
        });
    }

    loadCart() {
        this.sub = this.ecommerceService.ProductOrderChanged.subscribe(() => {
            let productOrder = this.ecommerceService.SelectedProductOrder;
            if (productOrder) {
                this.orders.productOrders.push(new ProductOrder(
                    productOrder.product, productOrder.quantity));
            }
            this.ecommerceService.ProductOrders = this.orders;
            this.orders = this.ecommerceService.ProductOrders;
            this.total = this.calculateTotal(this.orders.productOrders);
        });
    }

    ngOnDestroy() {
        this.sub.unsubscribe();
    }
}
```

当订单完成时，我们从这里向父组件发送一个事件，并且我们需要去结帐。这里还有一个`reset`()方法:

```java
finishOrder() {
    this.orderFinished = true;
    this.ecommerceService.Total = this.total;
    this.onOrderFinished.emit(this.orderFinished);
}

reset() {
    this.orderFinished = false;
    this.orders = new ProductOrders();
    this.orders.productOrders = []
    this.loadTotal();
    this.total = 0;
}
```

HTML 文件很简单:

```java
<div class="card text-white bg-danger mb-3" style="max-width: 18rem;">
    <div class="card-header text-center">Shopping Cart</div>
    <div class="card-body">
        <h5 class="card-title">Total: ${{total}}</h5>
        <hr>
        <h6 class="card-title">Items bought:</h6>

        <ul>
            <li *ngFor="let order of orders.productOrders">
                {{ order.product.name }} - {{ order.quantity}} pcs.
            </li>
        </ul>

        <button class="btn btn-light btn-block" (click)="finishOrder()"
             [disabled]="orders.productOrders.length == 0">Checkout
        </button>
    </div>
</div>
```

### 3.8.命令

我们将尽可能使事情简单，并在`OrdersComponent`中通过将属性设置为 true 并将订单保存在数据库中来模拟支付。我们可以通过`h2-console`或点击`[http://localhost:8080/api/orders](https://web.archive.org/web/20220627181507/http://localhost:8080/api/orders).`来检查订单是否已保存

我们还需要这里的`EcommerceService`,以便从购物车中检索产品列表和我们订单的总金额:

```java
export class OrdersComponent implements OnInit {
    orders: ProductOrders;
    total: number;
    paid: boolean;
    sub: Subscription;

    constructor(private ecommerceService: EcommerceService) {
        this.orders = this.ecommerceService.ProductOrders;
    }

    ngOnInit() {
        this.paid = false;
        this.sub = this.ecommerceService.OrdersChanged.subscribe(() => {
            this.orders = this.ecommerceService.ProductOrders;
        });
        this.loadTotal();
    }

    pay() {
        this.paid = true;
        this.ecommerceService.saveOrder(this.orders).subscribe();
    }
}
```

最后，我们需要向用户显示信息:

```java
<h2 class="text-center">ORDER</h2>
<ul>
    <li *ngFor="let order of orders.productOrders">
        {{ order.product.name }} - ${{ order.product.price }} x {{ order.quantity}} pcs.
    </li>
</ul>
<h3 class="text-right">Total amount: ${{ total }}</h3>

<button class="btn btn-primary btn-block" (click)="pay()" *ngIf="!paid">Pay</button>
<div class="alert alert-success" role="alert" *ngIf="paid">
    <strong>Congratulation!</strong> You successfully made the order.
</div>
```

## 4.融合 Spring Boot 和角度应用

我们已经完成了两个应用程序的开发，像我们这样单独开发可能更容易。但是，在生产中，拥有一个应用程序会方便得多，所以现在让我们将这两个应用程序合并起来。

这里我们要做的是**构建 Angular 应用程序，它调用 Webpack 来捆绑所有资产，并将它们推送到 Spring Boot 应用程序**的`/resources/static`目录中。这样，我们就可以运行 Spring Boot 应用程序，测试我们的应用程序，将所有这些打包并作为一个应用程序进行部署。

要做到这一点，我们需要**再次打开`package.json`，在`scripts`之后添加一些新的脚本。`build`** :

```java
"postbuild": "npm run deploy",
"predeploy": "rimraf ../resources/static/ && mkdirp ../resources/static",
"deploy": "copyfiles -f dist/** ../resources/static",
```

我们正在使用一些尚未安装的软件包，所以让我们安装它们:

```java
npm install --save-dev rimraf
npm install --save-dev mkdirp
npm install --save-dev copyfiles
```

`rimraf`命令将查看目录并创建一个新目录(实际上是清理目录)，而`copyfiles`将文件从分发文件夹(Angular 存放所有内容的地方)复制到我们的`static`文件夹中。

现在我们只需要**运行`npm run build`命令，这将运行所有这些命令，最终输出将是静态文件夹**中的打包应用程序。

然后，我们在端口 8080 运行我们的 Spring Boot 应用程序，在那里访问它并使用 Angular 应用程序。

## 5.结论

在本文中，我们创建了一个简单的电子商务应用程序。我们使用 Spring Boot 在后端创建了一个 API，然后在 Angular 开发的前端应用程序中使用它。我们演示了如何制作我们需要的组件，让它们相互通信，以及从 API 检索数据和向 API 发送数据。

最后，我们展示了如何将两个应用程序合并到一个静态文件夹中的打包 web 应用程序中。

和往常一样，我们在本文中描述的完整项目可以在 [GitHub 项目中找到。](https://web.archive.org/web/20220627181507/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-angular)**