# 事务性注释:Spring vs. JTA

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-vs-jta-transactional>

## 1.概观

在本教程中，我们将讨论**`org.springframework.transaction.annotation.Transactional`和`javax.transaction.Transactional`注释**的区别。

我们将从概述它们的配置属性开始。然后，我们将讨论每种组件可以应用于什么类型，以及在什么情况下我们可以使用其中的一种。

## 2.配置差异

与 JTA 的版本相比，Spring 的`Transactional`注解增加了额外的配置:

*   隔离——Spring 通过`isolation `属性提供事务范围的隔离；但是，在 JTA，该功能仅在连接级别可用
*   传播——在两个库中都可用，通过 Spring 中的`propagation`属性和 Java EE 中的`value`属性；Spring 提供了`Nested` 作为一种额外的传播类型
*   只读–仅在 Spring 中通过`readOnly`属性可用
*   超时–仅在 Spring 中通过`timeout`属性可用
*   回滚——两种注释都提供了回滚管理；JTA 提供了`rollbackOn`和`dontRollbackOn `属性，而 Spring 有`rollbackFor`和`noRollbackFor`，外加两个附加属性:`rollbackForClassName`和`noRollbackForClassName`

### 2.1.弹簧`Transactional`注释配置

例如，让我们在一个简单的汽车服务上使用和配置 Spring `Transactional`注释:

```java
import org.springframework.transaction.annotation.Transactional;

@Service
@Transactional(
  isolation = Isolation.READ_COMMITTED, 
  propagation = Propagation.SUPPORTS, 
  readOnly = false, 
  timeout = 30)
public class CarService {

    @Autowired
    private CarRepository carRepository;

    @Transactional(
      rollbackFor = IllegalArgumentException.class, 
      noRollbackFor = EntityExistsException.class,
      rollbackForClassName = "IllegalArgumentException", 
      noRollbackForClassName = "EntityExistsException")
    public Car save(Car car) {
        return carRepository.save(car);
    }
}
```

### 2.3.JTA `Transactional`注释配置

让我们使用 JTA `Transactional`注释对一个简单的租赁服务做同样的事情:

```java
import javax.transaction.Transactional;

@Service
@Transactional(Transactional.TxType.SUPPORTS)
public class RentalService {

    @Autowired
    private CarRepository carRepository;

    @Transactional(
      rollbackOn = IllegalArgumentException.class, 
      dontRollbackOn = EntityExistsException.class)
    public Car rent(Car car) {
        return carRepository.save(car);
    }
}
```

## 3.适用性和互换性

JTA `Transactional`注释适用于 CDI 管理的 bean 和被 Java EE 规范定义为受管 bean 的类，而 Spring 的`Transactional`注释仅适用于 Spring beans。

同样值得注意的是，对 JTA 1.2 的支持是在 Spring Framework 4.0 中引入的。因此，**我们可以在 Spring 应用程序**中使用 JTA `Transactional`注释。然而，反过来是不可能的，因为我们不能在 Spring 上下文之外使用 Spring 注释。

## 4.结论

在本教程中，我们讨论了 Spring 和 JTA 的`Transactional` 注释之间的差异，以及何时可以使用其中的一个。

和往常一样，本教程的代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220628235647/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-persistence-simple)