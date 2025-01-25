---
slug: 221
title: 'Spring AOP(面向切面编程)'
# draft: true
author: yexca
date: '2025-01-21T16:05:57+09:00'
categories:
    - 后端
    - Spring
tags:
    - SpringBoot
    - JavaWeb
---

Aspect Oriented Programming (面向切面编程、面向方面编程) 是面向特定方法编程

动态代理是面向切面编程最主流的实现。而 SpringAOP 是 Spring 框架的高级技术，旨在管理 bean 对象的过程中，主要通过底层的动态代理机制，对特定的方法进行编程

## 统计方法运行时间

导入依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```

编写 AOP 程序，针对特定方法根据业务需要进行编程

```java
@Slf4j
@Component
@Aspect // AOP类
public class TimeAspect {
    // 切入点表达式
    @Around("execution(* net.yexca.service.*.*(..))")
    public Object recordTime(ProceedingJoinPoint proceedingJoinPoint) throws Throwable {
        long begin = System.currentTimeMillis();
        // 调用原方法
        Object object = proceedingJoinPoint.proceed();
        long end = System.currentTimeMillis();
        log.info(proceedingJoinPoint.getSignature() + "方法执行时间：{}ms", end-begin);
        return object;
    }
}
```

> AOP 的应用场景有记录操作日志、权限控制、事务管理等

## 核心概念

连接点：JoinPoint，可以被AOP控制的方法 (暗含方法执行时的相关信息)

通知：Advice，指哪些重复的逻辑，也就是共性功能 (最终体现为一个方法)

切入点：PointCut，匹配连接点的条件，通知仅会在切入点方法执行时被应用

切面：Aspect，描述通知与切入点的对应关系 (通知+切入点)

目标对象：Target，通知所应用的对象

上例中，没写出来的 Service 的所有方法都是连接点，被切入点表达式选中的方法都是切入点，而 AOP 类的 recordTime 方法为通知，注解 @Around 与通知共同为切面，而 TimeAspect 类称为切面类

## 通知

### 通知类型

@Around：环绕通知，此注解标注的通知方法在目标方法前、后都被执行

@Before：前置通知，此注解标注的通知方法在目标方法前被执行

@After：后置通知，此注解标注的通知方法在目标方法后被执行，无论是否有异常都会执行

@AfterReturning：返回后通知，此注解标注的通知方法在目标方法后被执行，有异常不会执行

@AfterThrowing：异常后通知，此注解标注的通知方法发生异常后执行

```java
@Component
@Aspect
public class MyAspect {
    
    @Before("execution(* net.yexca.service.impl.*(..))")
    public void before(){
        System.out.println("Before");
    }
    
    @Around("execution(* net.yexca.service.impl.*(..))")
    public Object around(ProceedingJoinPoint proceedingJoinPoint) throws Throwable {
        System.out.println("Around before");
        Object result = proceedingJoinPoint.proceed();
        System.out.println("Around after");
        return result;
    }
    
    @After("execution(* net.yexca.service.impl.*(..))")
    public void after(){
        System.out.println("After");
    }
    
    @AfterReturning("execution(* net.yexca.service.impl.*(..))")
    public void afterRetruning(){
        System.out.println("AfterReturning");
    }
    
    @AfterThrowing("execution(* net.yexca.service.impl.*(..))")
    public void afterThrowing(){
        System.out.println("AfterThrowing");
    }
}
```

> @Around 环绕通知需要自己调用 `ProceedingJoinPoint.proceed()` 来让原始方法执行，其他通知不需要考虑目标方法执行
>
> @Around 环绕通知方法的返回值，必须指定为 Object，来接收原始方法的返回值

---

上述的 5 个注解的切点表达式都相同，可以提取，如下所示

```java
public class MyAspect {
    
    @Pointcut("execution(* net.yexca.service.impl.*(..))")
    public void pt(){}

    @Before("pt()")
    public void before(){
        System.out.println("Before");
    }
}
```

方法 pt() 若是 public 权限符，则可以在其他的类中引用

### 通知顺序

当有多个切面的切入点都匹配到了目标方法，目标方法运行时，多个通知方法都会被执行

```yml
net: 
  yexca: 
    aop: 
      - MyAspect1
      - MyAspect2
      - MyAspect3
```

假设三个 AOP 类都选中了同一个方法，不同切面类中，默认是按照切面类名字母排序

* 目标方法前的通知方法：字母排名靠前的先执行
* 目标方法后的通知方法：字母排名靠后的先执行

假设三个 AOP 类都有 @Before 和 @After，执行顺序为

```markdown
MyAspect1 before
MyAspect2 before
MyAspect3 before
MyAspect3 after
MyAspect2 after
MyAspect1 after
```

> 可以使用 @Order(num) 注解加在切面类上来控制顺序，num 越小越先执行，@Before 和 @After 执行同上

## 切入点表达式

描述切入点方法的一种表达式，主要用来决定项目中的哪些方法需要加入通知

常见形式有 `execution(...)` 根据方法的签名匹配和 `annotation` 根据注解匹配

### execution

主要根据方法的返回值、包名、类名、方法名、方法参数等信息来匹配，语法为

```markdown
execution(访问修饰符 返回值 包名.类名.方法名(方法参数) throws 异常)
```

其中访问修饰符、包名.类名、throws 异常可以省略，不过不建议省略包名.类名

也可以使用通配符描述切入点

* `*`：单个独立的任意符号，可以通配任意返回值、包名、类名、方法名、任意类型的一个参数，也可以通配包、类、方法名的一部分

```html
execution(* com.*.service.*.update*(*))
```

* `..`：多个连续的任意符号，可以通配任意层级的包，或任意类型、任意个数的参数

```html
execution(* com.yexca..service.*(..))
```

> 还可以使用 `&&`, `||`, `!` 来组合比较复杂的切入点表达式

书写建议

* 所有业务方法名在命名时尽量规范，方便切入点表达式快速匹配
* 描述切入点方法通常基于接口描述，而非实现类，增强拓展性
* 在满足业务需要的前提下，尽量缩小切入点的匹配范围

### @annotation

@annotation 切入点表达式，用于匹配标识有特定注解的方法，使用需先自定义注解

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface MyLog {
}
```

然后在方法上加入该注解，在 AOP 类方法上

```java
@Before("@annotation(net.yexca.aop.MyLog)")
```

## 连接点

在 Spring 中用 JoinPoint 抽象了连接点，用它可以获得方法执行时的相关信息

* 对于 @Around 通知，获取连接点信息只能使用 ProceedingJoinPoint
* 对于其他四种通知，获取连接点信息只能使用 JoinPoint，它是 ProceedingJoinPoint 的父类

```java
public Object recordTime(ProceedingJoinPoint proceedingJoinPoint) throws Throwable {
    // 获取目标对象的类名
    String className = proceedingJoinPoint.getTarget().getClass().getName();
        
    // 获取目标方法的方法名
   String methodNAme = proceedingJoinPoint.getSignature().getName();
        
    // 获取目标方法运行时传入的参数
    Object[] args = proceedingJoinPoint.getArgs();
    
    // 调用原方法
    Object object = proceedingJoinPoint.proceed();

    return object;
}
```
