# Spring 中的切入点表达式介绍

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-aop-pointcut-tutorial>

## 1。概述

在本教程中，我们将讨论 Spring AOP 切入点表达式语言。

首先，我们将介绍一些在面向方面编程中使用的术语。`join point`是程序执行的一个步骤，比如一个方法的执行或者一个异常的处理。在 Spring AOP 中，连接点总是代表一个方法的执行。`pointcut`是一个匹配连接点的谓词，` pointcut expression language`是一种以编程方式描述切入点的方式。

## 2。用途

切入点表达式可以作为`@Pointcut`注释的值出现:

```
@Pointcut("within(@org.springframework.stereotype.Repository *)")
public void repositoryClassMethods() {}
```

方法声明被称为**切入点签名**。它提供了一个名称，建议注释可以用它来引用这个切入点:

```
@Around("repositoryClassMethods()")
public Object measureMethodExecutionTime(ProceedingJoinPoint pjp) throws Throwable {
    ...
}
```

切入点表达式也可以作为`aop:pointcut`标签的`expression`属性的值出现:

```
<aop:config>
    <aop:pointcut id="anyDaoMethod" 
      expression="@target(org.springframework.stereotype.Repository)"/>
</aop:config>
```

## 3。切入点指示器

切入点表达式以一个**切入点指示符(PCD)** 开始，这是一个告诉 Spring AOP 匹配什么的关键字。有几个切入点指示符，比如方法的执行、类型、方法参数或者注释。

### 3.1。`execution`

主要的 Spring PCD 是`execution`，它匹配方法执行连接点:

```
@Pointcut("execution(public String com.baeldung.pointcutadvice.dao.FooDao.findById(Long))")
```

这个示例切入点将完全匹配`FooDao`类的`findById`方法的执行。这是可行的，但是不太灵活。假设我们想要匹配`FooDao`类的所有方法，这些方法可能有不同的签名、返回类型和参数。为此，我们可以使用通配符:

```
@Pointcut("execution(* com.baeldung.pointcutadvice.dao.FooDao.*(..))")
```

这里，第一个通配符匹配任何返回值，第二个匹配任何方法名，`(..)`模式匹配任何数量的参数(零个或更多)。

### 3.2。`within`

获得与上一节相同结果的另一种方法是使用`within` PCD，它将匹配限制在某些类型的连接点:

```
@Pointcut("within(com.baeldung.pointcutadvice.dao.FooDao)")
```

我们还可以匹配`com.baeldung`包或子包中的任何类型:

```
@Pointcut("within(com.baeldung..*)")
```

### 3.3。`this`和`target`

`this`将匹配限制在 bean 引用是给定类型的实例的连接点，而`target` 将匹配限制在目标对象是给定类型的实例的连接点。前者在 Spring AOP 创建基于 CGLIB 的代理时起作用，后者在创建基于 JDK 的代理时使用。假设目标类实现了一个接口:

```
public class FooDao implements BarDao {
    ...
}
```

在这种情况下，Spring AOP 将使用基于 JDK 的代理，我们应该使用`target` PCD，因为被代理的对象将是`Proxy`类的实例并实现`BarDao`接口:

```
@Pointcut("target(com.baeldung.pointcutadvice.dao.BarDao)")
```

另一方面，如果`FooDao`没有实现任何接口，或者`proxyTargetClass`属性被设置为 true，那么被代理的对象将是`FooDao` 的子类，我们可以使用`this` PCD:

```
@Pointcut("this(com.baeldung.pointcutadvice.dao.FooDao)")
```

### 3.4。`args`

我们可以使用这个 PCD 来匹配特定的方法参数:

```
@Pointcut("execution(* *..find*(Long))")
```

这个切入点匹配任何以 find 开始并且只有一个类型为`Long`的参数的方法。如果我们想要匹配一个有任意数量参数的方法，但是仍然有第一个类型为`Long`的参数，我们可以使用下面的表达式:

```
@Pointcut("execution(* *..find*(Long,..))")
```

### 3.5。`@target`

`@target` PCD(不要与上面描述的`target` PCD 混淆)将匹配限制在执行对象的类具有给定类型注释的连接点:

```
@Pointcut("@target(org.springframework.stereotype.Repository)")
```

### 3.6。`@args`

这个 PCD 将匹配限制在连接点上，在这些连接点上，传递的实际参数的运行时类型具有给定类型的注释。假设我们想要跟踪所有接受用`@Entity`注释标注的 beans 的方法:

```
@Pointcut("@args(com.baeldung.pointcutadvice.annotations.Entity)")
public void methodsAcceptingEntities() {}
```

要访问该参数，我们应该为建议提供一个`JoinPoint`参数:

```
@Before("methodsAcceptingEntities()")
public void logMethodAcceptionEntityAnnotatedBean(JoinPoint jp) {
    logger.info("Accepting beans with @Entity annotation: " + jp.getArgs()[0]);
}
```

### 3.7。`@within`

这个 PCD 限制匹配具有给定注释的类型中的连接点:

```
@Pointcut("@within(org.springframework.stereotype.Repository)")
```

这相当于:

```
@Pointcut("within(@org.springframework.stereotype.Repository *)")
```

### 3.8。`@annotation`

这个 PCD 将匹配限制在连接点的主题具有给定注释的连接点上。例如，我们可以创建一个`@Loggable`注释:

```
@Pointcut("@annotation(com.baeldung.pointcutadvice.annotations.Loggable)")
public void loggableMethods() {}
```

然后，我们可以记录由该注释标记的方法的执行情况:

```
@Before("loggableMethods()")
public void logMethod(JoinPoint jp) {
    String methodName = jp.getSignature().getName();
    logger.info("Executing method: " + methodName);
}
```

## 4。组合切入点表达式

切入点表达式可以使用 **& &** 、 **||** 和**进行组合！**操作员:

```
@Pointcut("@target(org.springframework.stereotype.Repository)")
public void repositoryMethods() {}

@Pointcut("execution(* *..create*(Long,..))")
public void firstLongParamMethods() {}

@Pointcut("repositoryMethods() && firstLongParamMethods()")
public void entityCreationMethods() {}
```

## 5。结论

在这篇关于 Spring AOP 和切入点的简短文章中，我们举例说明了一些切入点表达式的例子。

完整的例子可以在 GitHub 上找到[。](https://web.archive.org/web/20221003115607/https://github.com/eugenp/tutorials/tree/master/spring-aop)