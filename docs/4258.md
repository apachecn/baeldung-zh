# Java toString()方法

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-tostring>

## 1.概观

Java 中的每个类都直接或间接地是`Object`类的子类。由于`Object`类包含一个`toString() `方法，我们可以在任何实例上调用`toString()`并获得它的字符串表示。

在本教程中，我们将看看`toString()`的**默认行为，并学习如何改变它的行为。**

## 2.默认行为

每当我们打印一个对象引用时，它都会在内部调用`toString()`方法。所以，如果我们没有在类中定义一个`toString()`方法，那么`Object#` `toString()`就会被调用。

`Object's` `toString()`方法相当通用:

```
public String toString() {
    return getClass().getName()+"@"+Integer.toHexString(hashCode());
}
```

为了了解这是如何工作的，让我们创建一个将在整个教程中使用的`Customer `对象:

```
public class Customer {
    private String firstName;
    private String lastName;
    // standard getters and setters. No toString() implementation
}
```

现在，如果我们尝试打印我们的`C` `ustomer`对象，将调用`Object` # `toString()`，输出将类似于:

```
[[email protected]](/web/20221126214903/https://www.baeldung.com/cdn-cgi/l/email-protection)
```

## 3.覆盖默认行为

查看上面的输出，我们可以看到它没有给我们提供太多关于我们的`Customer` 对象的内容的信息。一般来说，我们对知道一个对象的 hashcode 不感兴趣，而是我们对象属性的内容。

通过覆盖`toString() `方法的默认行为，我们可以使方法调用的输出更有意义。

现在，让我们看几个使用对象的不同场景，看看我们如何覆盖这个默认行为。

## 4.原始类型和`Strings`

我们的`Customer`对象同时具有`String`和原始属性。我们需要覆盖`toString()`方法来获得更有意义的输出:

```
public class CustomerPrimitiveToString extends Customer {
    private long balance;

    @Override
    public String toString() {
        return "Customer [balance=" + balance + ", getFirstName()=" + getFirstName()
          + ", getLastName()=" + getLastName() + "]";
    }
} 
```

让我们看看现在调用`toString() `时会得到什么:

```
@Test
public void givenPrimitive_whenToString_thenCustomerDetails() {
    CustomerPrimitiveToString customer = new CustomerPrimitiveToString();
    customer.setFirstName("Rajesh");
    customer.setLastName("Bhojwani");
    customer.setBalance(110);
    assertEquals("Customer [balance=110, getFirstName()=Rajesh, getLastName()=Bhojwani]", 
      customer.toString());
}
```

## 5.复杂的 Java 对象

现在让我们考虑一个场景，其中我们的`Customer`对象也包含一个类型为`Order.` 的`order `属性，我们的`Order `类同时具有`String`和原始数据类型字段。

所以，让我们再次覆盖`toString()`:

```
public class CustomerComplexObjectToString extends Customer {
    private Order order;
    //standard setters and getters

    @Override
    public String toString() {
        return "Customer [order=" + order + ", getFirstName()=" + getFirstName()
          + ", getLastName()=" + getLastName() + "]";
    }      
}
```

**因为`order`是一个复杂的对象`,` 如果我们只是打印我们的`Customer`对象，而没有覆盖我们的`Order` 类中的`toString() `方法，它会将`orders`打印为`[[email protected]](/web/20221126214903/https://www.baeldung.com/cdn-cgi/l/email-protection)<hashcode>.`**

为了解决这个问题，让我们也覆盖`Order`中的`toString()`:

```
public class Order {

    private String orderId;
    private String desc;
    private long value;
    private String status;

    @Override
    public String toString() {
        return "Order [orderId=" + orderId + ", desc=" + desc + ", value=" + value + "]";
    }
} 
```

现在，让我们看看当我们在包含一个`order `属性的`Customer`对象上调用`toString() `方法时会发生什么:

```
@Test
public void givenComplex_whenToString_thenCustomerDetails() {
    CustomerComplexObjectToString customer = new CustomerComplexObjectToString();    
    // .. set up customer as before
    Order order = new Order();
    order.setOrderId("A1111");
    order.setDesc("Game");
    order.setStatus("In-Shiping");
    customer.setOrders(order);

    assertEquals("Customer [order=Order [orderId=A1111, desc=Game, value=0], " +
      "getFirstName()=Rajesh, getLastName()=Bhojwani]", customer.toString());
}
```

## 6.对象数组

接下来，让我们把我们的`Customer `改成有一个`Order` s `.` 的数组，如果我们只是打印我们的`Customer`对象，而没有对我们的`orders`对象进行特殊处理，它会把`orders`打印成`Order;@<hashcode>`。

为了解决这个问题，让我们将`[Arrays.toString()](/web/20221126214903/https://www.baeldung.com/java-array-to-string) `用于`orders`字段:

```
public class CustomerArrayToString  extends Customer {
    private Order[] orders;

    @Override
    public String toString() {
        return "Customer [orders=" + Arrays.toString(orders) 
          + ", getFirstName()=" + getFirstName()
          + ", getLastName()=" + getLastName() + "]";
    }    
} 
```

让我们看看调用上面的`toString()`方法的结果:

```
@Test
public void givenArray_whenToString_thenCustomerDetails() {
    CustomerArrayToString customer = new CustomerArrayToString();
    // .. set up customer as before
    // .. set up order as before
    customer.setOrders(new Order[] { order });         

    assertEquals("Customer [orders=[Order [orderId=A1111, desc=Game, value=0]], " +
      "getFirstName()=Rajesh, getLastName()=Bhojwani]", customer.toString());
}
```

## 7.包装器、集合和`StringBuffers`

当一个对象完全由[包装器](/web/20221126214903/https://www.baeldung.com/java-wrapper-classes)、[集合](/web/20221126214903/https://www.baeldung.com/java-collections)或 [`StringBuffer` s](/web/20221126214903/https://www.baeldung.com/java-collections) 组成时，不需要定制`toString() `实现，因为这些对象已经用有意义的表示覆盖了`toString()`方法:

```
public class CustomerWrapperCollectionToString extends Customer {
    private Integer score; // Wrapper class object
    private List<String> orders; // Collection object
    private StringBuffer fullname; // StringBuffer object

    @Override
    public String toString() {
        return "Customer [score=" + score + ", orders=" + orders + ", fullname=" + fullname
          + ", getFirstName()=" + getFirstName() + ", getLastName()=" + getLastName() + "]";
    }
} 
```

让我们再来看看调用`toString()`的结果:

```
@Test
public void givenWrapperCollectionStrBuffer_whenToString_thenCustomerDetails() {
    CustomerWrapperCollectionToString customer = new CustomerWrapperCollectionToString();
    // .. set up customer as before
    // .. set up orders as before 
    customer.setOrders(new Order[] { order }); 

    StringBuffer fullname = new StringBuffer();
    fullname.append(customer.getLastName()+ ", " + customer.getFirstName());

    assertEquals("Customer [score=8, orders=[Book, Pen], fullname=Bhojwani, Rajesh, getFirstName()=Rajesh, "
      + "getLastName()=Bhojwani]", customer.toString());
}
```

## 8.结论

在本文中，我们研究了如何创建我们自己的`toString() `方法的实现。

本文的所有源代码都可以在 GitHub 上的[处获得。](https://web.archive.org/web/20221126214903/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-string-operations)