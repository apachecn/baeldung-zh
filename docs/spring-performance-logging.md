# 春季性能记录

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-performance-logging>

## 1。概述

在本教程中，我们将研究 Spring 框架为性能监控提供的几个基本选项。

## 2。`PerformanceMonitorInterceptor`

一个简单的解决方案是获得我们方法执行时间的基本监控功能，我们可以利用 Spring AOP(面向方面编程)中的`PerformanceMonitorInterceptor`类。

Spring AOP 允许在应用程序中定义横切关注点，这意味着代码拦截一个或多个方法的执行，以便添加额外的功能。

`PerformanceMonitorInterceptor`类是一个拦截器，它可以与任何要同时执行的自定义方法相关联。这个类使用一个`StopWatch`实例来确定方法运行的开始和结束时间。

让我们创建一个简单的`Person`类和一个`PersonService`类，其中包含我们将监控的两个方法:

```java
public class Person {
    private String lastName;
    private String firstName;
    private LocalDate dateOfBirth;

    // standard constructors, getters, setters
}
```

```java
public class PersonService {

    public String getFullName(Person person){
        return person.getLastName()+" "+person.getFirstName();
    }

    public int getAge(Person person){
        Period p = Period.between(person.getDateOfBirth(), LocalDate.now());
        return p.getYears();
    }
}
```

为了利用 Spring monitoring 拦截器，我们需要定义一个切入点和顾问:

```java
@Configuration
@EnableAspectJAutoProxy
@Aspect
public class AopConfiguration {

    @Pointcut(
      "execution(public String com.baeldung.performancemonitor.PersonService.getFullName(..))"
    )
    public void monitor() { }

    @Bean
    public PerformanceMonitorInterceptor performanceMonitorInterceptor() {
        return new PerformanceMonitorInterceptor(true);
    }

    @Bean
    public Advisor performanceMonitorAdvisor() {
        AspectJExpressionPointcut pointcut = new AspectJExpressionPointcut();
        pointcut.setExpression("com.baeldung.performancemonitor.AopConfiguration.monitor()");
        return new DefaultPointcutAdvisor(pointcut, performanceMonitorInterceptor());
    }

    @Bean
    public Person person(){
        return new Person("John","Smith", LocalDate.of(1980, Month.JANUARY, 12));
    }

    @Bean
    public PersonService personService(){
        return new PersonService();
    }
}
```

切入点包含一个表达式，它标识了我们想要被拦截的方法——在我们的例子中是`PersonService`类的`getFullName()`方法。

配置完`performanceMonitorInterceptor()` bean 后，我们需要将拦截器与切入点关联起来。这是通过一个顾问来实现的，如上例所示。

最后，`@EnableAspectJAutoProxy`注释使 [AspectJ 支持](/web/20220628105252/https://www.baeldung.com/aspectj)我们的 beans。简而言之，AspectJ 是一个库，它通过像`@Pointcut`这样方便的注释使 Spring AOP 的使用变得更加容易。

创建配置之后，我们需要**将拦截器类的日志级别设置为`TRACE`** ，因为这是它记录消息的级别。

比如使用 Jog4j，我们可以通过`log4j.properties`文件来实现这一点:

```java
log4j.logger.org.springframework.aop.interceptor.PerformanceMonitorInterceptor=TRACE, stdout
```

对于`getAge()`方法的每次执行，我们将在控制台日志中看到`TRACE`消息:

```java
2017-01-08 19:19:25 TRACE 
  PersonService:66 - StopWatch 
  'com.baeldung.performancemonitor.PersonService.getFullName': 
  running time (millis) = 10
```

## 3。自定义性能监控拦截器

如果我们希望**对性能监控**的完成方式有更多的控制，我们可以实现我们自己的定制拦截器。

为此，让我们扩展`AbstractMonitoringInterceptor`类并覆盖 `invokeUnderTrace()`方法来记录方法的开始、结束和持续时间，以及如果方法执行持续时间超过 10 ms 时的警告:

```java
public class MyPerformanceMonitorInterceptor extends AbstractMonitoringInterceptor {

    public MyPerformanceMonitorInterceptor() {
    }

    public MyPerformanceMonitorInterceptor(boolean useDynamicLogger) {
            setUseDynamicLogger(useDynamicLogger);
    }

    @Override
    protected Object invokeUnderTrace(MethodInvocation invocation, Log log) 
      throws Throwable {
        String name = createInvocationTraceName(invocation);
        long start = System.currentTimeMillis();
        log.info("Method " + name + " execution started at:" + new Date());
        try {
            return invocation.proceed();
        }
        finally {
            long end = System.currentTimeMillis();
            long time = end - start;
            log.info("Method "+name+" execution lasted:"+time+" ms");
            log.info("Method "+name+" execution ended at:"+new Date());

            if (time > 10){
                log.warn("Method execution longer than 10 ms!");
            }            
        }
    }
}
```

需要遵循与前一节中相同的将定制拦截器关联到一个或多个方法的步骤。

让我们为`PersonService`的 `getAge()`方法定义一个切入点，并将它与我们创建的拦截器关联起来:

```java
@Pointcut("execution(public int com.baeldung.performancemonitor.PersonService.getAge(..))")
public void myMonitor() { }

@Bean
public MyPerformanceMonitorInterceptor myPerformanceMonitorInterceptor() {
    return new MyPerformanceMonitorInterceptor(true);
}

@Bean
public Advisor myPerformanceMonitorAdvisor() {
    AspectJExpressionPointcut pointcut = new AspectJExpressionPointcut();
    pointcut.setExpression("com.baeldung.performancemonitor.AopConfiguration.myMonitor()");
    return new DefaultPointcutAdvisor(pointcut, myPerformanceMonitorInterceptor());
}
```

让我们将定制拦截器的日志级别设置为`INFO`:

```java
log4j.logger.com.baeldung.performancemonitor.MyPerformanceMonitorInterceptor=INFO, stdout
```

g `etAge()`方法的执行产生了以下输出:

```java
2017-01-08 19:19:25 INFO PersonService:26 - 
  Method com.baeldung.performancemonitor.PersonService.getAge 
  execution started at:Sun Jan 08 19:19:25 EET 2017
2017-01-08 19:19:25 INFO PersonService:33 - 
  Method com.baeldung.performancemonitor.PersonService.getAge execution lasted:50 ms
2017-01-08 19:19:25 INFO PersonService:34 - 
  Method com.baeldung.performancemonitor.PersonService.getAge 
  execution ended at:Sun Jan 08 19:19:25 EET 2017
2017-01-08 19:19:25 WARN PersonService:37 - 
  Method execution longer than 10 ms!
```

## 4。结论

在这个快速教程中，我们介绍了 Spring 中简单的性能监控。

和往常一样，本文的完整源代码可以在 Github 上找到[。](https://web.archive.org/web/20220628105252/https://github.com/eugenp/tutorials/tree/master/spring-aop-2)