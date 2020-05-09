---
layout: post
title: template page
categories: [cate1, cate2]
description: some word here
keywords: keyword1, keyword2
---

Content here

# 1 Spring AOP 简介

## 1.1 AOP 概述

### 1.1.1 AOP 是什么?

AOP（Aspect Orient Programming）是一种设计思想，是软件设计领域中的面向切面编程，它是面向对象编程(OOP)的一种补充和完善。它以通过预编译方式和运行期动态代理方式，实现在不修改源代码的情况下给程序动态统一添加额外功能的一种技术。如图所示：

![image-20200506091902536](E:\Git\AnnerYang.github.io\images\posts\java\image-20200506091902536.png)

AOP与OOP字面意思相近，但其实两者完全是面向不同领域的设计思想。实际项目中我们通常将面向对象理解为一个静态过程(例如一个系统有多少个模块，一个模块有哪些对象，对象有哪些属性)，面向切面的运行期代理方式，理解为一个动态过程，可以在对象运行时动态织入一些扩展功能或控制对象执行。

### 1.1.2 AOP 应用场景分析

实际项目中通常会将系统分为两大部分，一部分是核心业务，一部分是非核业务。在编程实现时我们首先要完成的是核心业务的实现，非核心业务一般是通过特定方式切入到系统中，这种特定方式一般就是借助AOP进行实现。

AOP就是要基于OCP(开闭原则)，在不改变原有系统核心业务代码的基础上动态添加一些扩展功能并可以"控制"对象的执行。例如AOP应用于项目中的日志处理，事务处理，权限处理，缓存处理等等。如图所示：

![image-20200506092009482](E:\Git\AnnerYang.github.io\images\posts\java\image-20200506092009482.png)

思考:现有一业务,在没有AOP编程时,如何基于OCP原则实现功能扩展?

### 1.1.3 AOP 应用原理分析

Spring AOP底层基于代理机制实现功能扩展：

1. 假如目标对象(被代理对象)实现接口，则底层可以采用JDK动态代理机制为目标对象创建代理对象（目标类和代理类会实现共同接口）。
2. 假如目标对象(被代理对象)没有实现接口，则底层可以采用CGLIB代理机制为目标对象创建代理对象（默认创建的代理类会继承目标对象类型）。

Spring AOP 原理分析，如图所示：

![image-20200506092115683](E:\Git\AnnerYang.github.io\images\posts\java\image-20200506092115683.png)

- 通过新增代理类 **继承** 业务层来实现功能拓展:

代码示例:

1. 有如下业务sendMsg获取执行时长,传统方式是直接修改业务层源码,但是违背了OCP(开闭原则)

```java
public class MailService {

	 void sendMsg(String msg) {
		// long t1=System.nanoTime();
		 System.out.println("send msg");
		// long t2=System.nanoTime();
		// System.out.println("执行时长:"+(t2-t1));
	 }
}
```

2. 创建新的拓展功能类继承原有业务层功能进行拓展

```Java
public class TimeMailService extends MailService {

	@Override
	void sendMsg(String msg) {
		long t1=System.nanoTime();
		super.sendMsg(msg);
		long t2=System.nanoTime();
		System.out.println("执行时长:"+(t2-t1));
	}
}
```

3. 测试

```java
public class MailServiceTests {

	public static void main(String[] args) {
		MailService mailService=new TimeMailService();
		mailService.sendMsg("hello test");
	}
}
```

- 通过新增业务层实现类与原有实现类 **组合** 实现功能拓展

1. Service层代码

```java
public interface SearchService {
    
	  Object seach(String key);
}
```

2. 默认功能实现

```Java
public final class DefaultSearchService implements SearchService {

	@Override
	public Object seach(String key) {
	    System.out.println("seach by "+key);
		return null;
	}

}
```

3. 新增实现类拓展执行时长功能

```Java
public class LogSearchService implements SearchService {

	//has a 
	private DefaultSearchService searchService;
	public LogSearchService(DefaultSearchService searchService) {
		this.searchService=searchService;
	}
	@Override
	public Object seach(String key) {
		System.out.println("start:"+System.nanoTime());
		Object result=searchService.seach(key);
		System.out.println("end:"+System.nanoTime());
		return result;
	}

}
```

4. 使用实现类组合方式

```Java
public class SeachServiceTests {

	public static void main(String[] args) {
		SearchService service=new LogSearchService(new DefaultSearchService());
		service.seach("tedu");
	}
}
```



说明:Spring boot2.x 中AOP现在默认使用的CGLIB代理,假如需要使用JDK动态代理可以在配置文件(applicatiion.properties)中进行如下配置:

`spring.aop.proxy-target-class=false`

>开闭原则（Open Closed Principle，OCP）由勃兰特・梅耶（Bertrand Meyer）提出，他在 1988 年的著作《面向对象软件构造》（Object Oriented Software Construction）中提出：软件实体应当对扩展开放，对修改关闭（Software entities should be open for extension，but closed for modification），这就是开闭原则的经典定义。
>
>开闭原则的意思就是说，你设计的时候，时刻要考虑，尽量让这个类是足够好，写好了就不要去修改了，如果新需求来，我们增加一些类就完事了，原来的代码能不动则不动。
>
>这个原则有两个特性:
>
>- 一个是说对于扩展是开放的
>- 一个是说对于更改是封闭的
>
>面对需求，对程序的改动是通过增加新代码进行的，而不是更改现有的代码。这就是“开放-封闭原则”的精神所在。

## 1.2 AOP相关术语分析

- 切面(aspect): 横切面对象,一般为一个具体类对象(可以借助@Aspect声明)。
- 通知(Advice):在切面的某个特定连接点上执行的动作(扩展功能)，例如around,before,after等。
- 连接点(joinpoint):程序执行过程中某个特定的点，一般指被拦截到的的方法。
- 切入点(pointcut):对多个连接点(Joinpoint)一种定义,一般可以理解为多个连接点的集合。

连接点与切入点定义如图所示：

![image-20200506121312003](E:\Git\AnnerYang.github.io\images\posts\java\image-20200506121312003.png)

说明：我们可以简单的将机场的一个安检口理解为连接点，多个安检口为切入点，安全检查过程看成是通知。总之，概念很晦涩难懂，多做例子，做完就会清晰。先可以按白话去理解。

# 2 Spring AOP快速实践

## 2.1 业务描述

基于项目中的核心业务，添加简单的日志操作，借助SLF4J日志API输出目标方法的执行时长。

## 2.2 项目创建及配置

创建maven项目或在已有项目基础上添加AOP启动依赖：

```xml
 <dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-aop</artifactId>
  </dependency>
```

说明：基于此依赖spring可以整合AspectJ框架快速完成AOP的基本实现。AspectJ 是一个面向切面的框架，他定义了AOP的一些语法，有一个专门的字节码生成器来生成遵守java规范的class文件。

## 2.3 扩展业务分析及实现

### 2.3.1 创建日志切面类对象

将此日志切面类作为核心业务增强（一个横切面对象）类，用于输出业务执行时长，其关键代码如下：

```java
package com.cy.pj.common.aspect;

@Aspect
@Slf4j
@Component

public class SysLogAspect {

         @Pointcut("bean(sysUserServiceImpl)")
         public void logPointCut() {}

         @Around("logPointCut()")
         public Object around(ProceedingJoinPoint jp)
         throws Throwable{
                 try {
                   log.info("start:"+System.currentTimeMillis());
                   Object result=jp.proceed();//调用下一个切面方法或目标方法
                   log.info("after:"+System.currentTimeMillis());
                   return result;
                 }catch(Throwable e) {
                   log.error(e.getMessage());
                   throw e;
                 }
         }
}
```

说明：

- @Aspect 注解用于标识或者描述AOP中的切面类型，基于切面类型构建的对象用于为目标对象进行功能扩展或控制目标对象的执行。
- @Pointcut注解用于描述切面中的方法，并定义切面中的切入点（基于特定表达式的方式进行描述），在本案例中切入点表达式用的是bean表达式，这个表达式以bean开头，bean括号中的内容为一个spring管理的某个bean对象的名字。
- @Around注解用于描述切面中方法，这样的方法会被认为是一个环绕通知（核心业务方法执行之前和之后要执行的一个动作），@Aournd注解内部value属性的值为一个切入点表达式或者是切入点表达式的一个引用(这个引用为一个@PointCut注解描述的方法的方法名)。
- ProceedingJoinPoint类为一个连接点类型，此类型的对象用于封装要执行的目标方法相关的一些信息。一般用于@Around注解描述的方法参数。

### 2.3.2 业务切面测试实现

启动项目测试或者进行单元测试，其中Spring Boot项目中的单元测试代码如下：

```java
@SpringBootTest

public class AopTests {

         @Autowired
         private SysUserService userService;

         @Test
         public void testSysUserService() {
                 PageObject<SysUserDeptVo> po=
                 userService.findPageObjects("admin",1);
                 System.out.println("rowCount:"+po.getRowCount());
         }
}
```

对于测试类中的userService对象而言,它有可能指向JDK代理,也有可能指向CGLIB代理,具体是什么类型的代理对象,要看application.yml配置文件中的配置.

### 2.3.3 应用总结分析

在业务应用,AOP相关对象分析,如图所示:

![image-20200506123350712](E:\Git\AnnerYang.github.io\images\posts\java\image-20200506123350712.png)

## 2.4 扩展业务织入增强分析

### 2.4.1 基于JDK代理方式实现

假如目标对象有实现接口,则可以基于JDK为目标对象创建代理代理对象,然后为目标对象进行功能扩展,如图所示：

![image-20200506123456392](E:\Git\AnnerYang.github.io\images\posts\java\image-20200506123456392.png)

### 2.4.2 基于CGLIB代理方式实现

假如目标对象没有实现接口，可以基于CGLIB代理方式为目标织入功能扩展，如图所示：

![image-20200506123557436](E:\Git\AnnerYang.github.io\images\posts\java\image-20200506123557436.png)

说明：目标对象实现了接口也可以基于CGLIB为目标对象创建代理对象。

# 3 Spring AOP编程增强

## 3.1 切面通知应用增强

### 3.1.1 通知类型

在基于Spring AOP编程的过程中，基于AspectJ框架标准，spring中定义了五种类型的通知(通知描述的是一种扩展业务)，它们分别是：

- 前置通知 (@Before) 。
- 返回通知 (@AfterReturning) 。
- 异常通知 (@AfterThrowing) 。
- 后置通知 (@After)。
- 环绕通知 (@Around) :重点掌握（优先级最高）

### 3.1.2 通知执行顺序

假如这些通知全部写到一个切面对象中，其执行顺序及过程，如图所示：

![image-20200506141337732](E:\Git\AnnerYang.github.io\images\posts\java\image-20200506141337732.png)

说明：实际项目中可能不会在切面中定义所有的通知，具体定义哪些通知要结合业务进行实现。

### 3.1.3 通知实践过程分析

代码事件分析如下:

```Java
package com.cy.pj.common.aspect;

import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.*;
import org.springframework.stereotype.Component;

@Aspect
@Component
public class SysTimeAspect {

    @Pointcut("bean(sysUserServiceImpl)")
    public void doTime(){}

    /**
     * 目标方法执行之前执行
     */
    @Before("doTime()")
    public void doBefore(){
        System.out.println("@Before");
    }

    /**
     * 目标方法执行之后执行
     */
    @After("doTime()")
    public void doAfter(){
        System.out.println("@After");
    }

    /**
     * 在After之后执行,前提是程序正常结束
     */
    @AfterReturning("doTime()")
    public void doAfterReturning(){
        System.out.println("@AfterReturning");
    }

    /**
     * After之后执行,前提是程序异常结束
     */
    @AfterThrowing("doTime()")
    public void doAfterThrowing(){
        System.out.println("@AfterThrowing");
    }

    @Around("doTime()")
    public Object doAround(ProceedingJoinPoint jp) throws Throwable{
        System.out.println("@Around.before");
        try {
            Object result = jp.proceed();
            System.out.println("@Around.afterReturning");
            return result;
        }catch (Throwable e){
            System.out.println(e.getMessage());
            throw e;
        }finally {
            System.out.println("@Around.after");
        }
    }
}
```

- 业务正常执行输出结果顺序:

![image-20200506144946465](E:\Git\AnnerYang.github.io\images\posts\java\image-20200506144946465.png)

- 业务异常执行输出结果顺序

![image-20200506145131616](E:\Git\AnnerYang.github.io\images\posts\java\image-20200506145131616.png)

说明：对于@AfterThrowing通知只有在出现异常时才会执行，所以当做一些异常监控时可在此方法中进行代码实现。

定义一个异常监控切面,对目标页面方法进行异常监控,并以日志信息形式输出异常

```java
package com.cy.pj.common.aspect;

import lombok.extern.slf4j.Slf4j;
import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.annotation.AfterThrowing;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.reflect.MethodSignature;
import org.springframework.stereotype.Component;

@Component
@Aspect
@Slf4j
public class SysExceptionAspect {

    @AfterThrowing(pointcut="bean(sysLogServiceImpl)",throwing = "throwable")
    public void doHandleException(Throwable throwable){
        log.error("SysExceptionAspect's exception msg is {}",throwable.getMessage());
    }
}
```

![image-20200506152220859](E:\Git\AnnerYang.github.io\images\posts\java\image-20200506152220859.png)

说明：AfterThrowing中throwing属性的值，需要与它描述的方法的异常参数名相同。

## 3.2 切入点表达式增强

Spring中通过切入点表达式定义具体切入点，其常用AOP切入点表达式定义及说明：

| **指示符**  | **作用**                           |
| ----------- | ---------------------------------- |
| bean        | 用于匹配指定bean对象的所有方法     |
| within      | 用于匹配指定包下所有类内的所有方法 |
| execution   | 用于按指定语法规则匹配到具体方法   |
| @annotation | 用于匹配指定注解修饰的方法         |

### 3.2.1 bean表达式（重点）

**bean表达式一般应用于类级别，实现粗粒度的切入点定义，案例分析：**

- bean("userServiceImpl")指定一个userServiceImpl类中所有方法。
- bean("*ServiceImpl")指定所有后缀为ServiceImpl的类中所有方法。

说明:bean表达式内部的对象是由spring容器管理的一个bean对象,表达式内部的名字应该是spring容器中某个bean的name。

### 3.2.2 within表达式（了解）

**within表达式应用于类级别，实现粗粒度的切入点表达式定义，案例分析：**

- within("aop.service.UserServiceImpl")指定当前包中这个类内部的所有方法。
- within("aop.service.*") 指定当前目录下的所有类的所有方法。
- within("aop.service..*") 指定当前目录以及子目录中类的所有方法。

### 3.2.3 execution表达式（了解）

**execution表达式应用于方法级别，实现细粒度的切入点表达式定义，案例分析：**

语法：execution(返回值类型 包名.类名.方法名(参数列表))。

- execution(void aop.service.UserServiceImpl.addUser())匹配addUser方法。
- execution(void aop.service.PersonServiceImpl.addUser(String)) 方法参数必须为String的addUser方法。
- execution(* aop.service..*.*(..)) 万能配置。

### 3.2.4 @annotation表达式（重点）

**@annotaion表达式应用于方法级别，实现细粒度的切入点表达式定义，案例分析**

- @annotation(anno.RequiredLog) 匹配有此注解描述的方法。
- @annotation(anno.RequiredCache) 匹配有此注解描述的方法。

其中:RequiredLog为我们自己定义的注解,当我们使用@RequiredLog注解修饰业务层方法时,系统底层会在执行此方法时进行日扩展操作。

**拓展**:定义一Cache相关切面,使用注解表达式定义切入点,并使用此注解对需要使用cache的业务方法进行描述,代码分析如下:

1. 自定义注解RequiredCache 和RemoveCache

```Java
package com.cy.pj.common.annotation;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Retention(RetentionPolicy.RUNTIME)//指示注释类型的注释要保留多久
@Target(ElementType.METHOD)//指示注释类型所适用的程序元素的种类
public @interface RequiredCache {
    String value() default "cache";
}
```

```Java
package com.cy.pj.common.annotation;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface RemoveCache {
    String key();
}

```

2. 定义SysCacheAspect切面对象

```Java
package com.cy.pj.common.aspect;

import com.cy.pj.common.annotation.RemoveCache;
import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.AfterReturning;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Pointcut;
import org.aspectj.lang.reflect.MethodSignature;
import org.jetbrains.annotations.NotNull;
import org.springframework.stereotype.Component;

import java.lang.reflect.Method;
import java.util.Map;
import java.util.concurrent.ConcurrentHashMap;

/**
 * FAQ?
 * 1.一个切面中可以定义多个通知么? 可以
 * 2.一个切面中的多个通知可以不同的切入点么? 可以
 * 3.注解切入点表达式如何应用?(定义注解,使用注解描述方法,基于注解定义切入点)
 */
@Component
@Aspect
public class SysCacheAspect {

    //借助此对象缓存从数据库中查询到的数据
    private Map<String,Object> cache = new ConcurrentHashMap<>();


    @Pointcut("@annotation(com.cy.pj.common.annotation.RequiredCache)")
    public void doCache() {}

    @Around("doCache()")
    public Object around(ProceedingJoinPoint jp) throws Throwable{
        //1.从cache获取数据
        Object result = cache.get("cacheKey");//自己定义一个key
        if (result!=null){
            return result;
        }
        //2.cache中没有则查询数据库
        result = jp.proceed();
        //3.将查询到的数据存储到cache
        cache.put("cacheKey",result);
        //4.返回查询结果
        return result;
    }

    //清除cache缓存,保持与数据库数据一致性
    @Pointcut("@annotation(com.cy.pj.common.annotation.RemoveCache)")
    public void removeCache(){}

    @AfterReturning("removeCache()")
    public void doAfterReturning(JoinPoint jp) throws Exception{
        Method targetMethod = doGetMethod(jp);
        //获取目标方法的注解
        RemoveCache removeCache = targetMethod.getAnnotation(RemoveCache.class);
        //获取注解上的key
        String key = removeCache.key();
        cache.remove(key);//通过获取注解获取key
    }

    @NotNull
    private  Method doGetMethod(@NotNull JoinPoint jp) throws NoSuchMethodException {
        //通过连接点和获取方法签名(方法签名中包含你要执行的目标方法信息)
        MethodSignature ms = (MethodSignature) jp.getSignature();
        //获取目标类的字节码对象
        Class<?> targetCls = jp.getTarget().getClass();
        //获取目标方法
        return targetCls.getDeclaredMethod(ms.getName(),ms.getParameterTypes());
    }
}
```

3.

```java
@Service
public class SysDeptServiceImpl implements SysDeptService {

	//@Autowired
	@Resource
	private SysDeptDao sysDeptDao;

	@RequiredCache
	@Override
	public List<Map<String, Object>> findObjects() {
		List<Map<String, Object>> list=sysDeptDao.findObjects();
		...
		return list;
	}

	@RemoveCache
	@Override
	public int saveObject(SysDept entity) {
		...
		int rows=sysDeptDao.insertObject(entity);
		return rows;
	}

	@RemoveCache
	@Override
	public int deleteObject(Integer id) {
		...
		int rows=sysDeptDao.deleteObject(id);
		return rows;
	}
}
```

## 3.3 切面优先级设置实现

切面的优先级需要借助@Order注解进行描述，数字越小优先级越高，默认优先级比较低。例如：

定义日志切面并指定优先级:

```java
@Order(1)
@Aspect
@Component
public class SysLogAspect {
    ...
}
```

定义缓存切面并指定优先级：

```java
@Order(2)
@Aspect
@Component
public class SysCacheAspect {
	...
}
```

说明：当多个切面作用于同一个目标对象方法时，这些切面会构建成一个切面链，类似过滤器链、拦截器链，其执行分析如图所示：

![image-20200506171201649](E:\Git\AnnerYang.github.io\images\posts\java\image-20200506171201649.png)

## 3.4 关键对象与术语总结

Spring 基于AspectJ框架实现AOP设计的关键对象概览，如图所示：

![image-20200506171317820](E:\Git\AnnerYang.github.io\images\posts\java\image-20200506171317820.png)

**@Aspect(切面)**是**@Pointcut(切入点)** 和**@Advice(通知)**的集合

其中通过**@Order**可以设置此切面在程序执行过程中的优先级(@Order(1)数字越小,优先级越高)

**JoinPoint**表示可以被此切面拦截到的连接点,在spring中,这些点指的是方法,因为spring只支持方法类型的连接点.

**@Pointcut(切入点)**是指我们要对哪些Joinpoint进行拦截的定义,通俗点讲就是我们要对哪些核心业务进行拓展,就以这些核心业务的具体实现类为切入点;其中还可以通过bean,within,execution,@annotation等表达式来增强切入点的设置

**@Advice(通知)**是指拦截到Joinpoint(连接点,也就是方法)之后,我们可以通过设置@Before,@AfterReturning,@AfterThrowing,@After,@Around来完成我们的需求.

# 4 Spring AOP事务处理

## 4.1 Spring中事务简介

### 4.1.1 事务定义

事务(Transaction)是一个业务,是一个不可分割的逻辑工作单元，基于事务可以更好的保证业务的正确性。

### 4.1.2 事务特性

事务具备ACID特性，分别是：

- 原子性（Atomicity）：一个事务中的多个操作要么都成功要么都失败。
- 一致性（Consistency): (例如存钱操作,存之前和存之后的总钱数应该是一致的。
- 隔离性（Isolation）：事务与事务应该是相互隔离的。
- 持久性（Durability）：事务一旦提交,数据要持久保存。

说明:目前市场上在事务一致性方面,通常会做一定的优化,比方说只要最终一致就可以了,这样的事务我们通常会称之为柔性事务(只要最终一致就可以了).

## 4.2 Spring 中事务管理

### 4.2.1 Spring中事务方式概述

Spring框架中提供了一种声明式事务的处理方式，此方式基于AOP代理,可以将具体业务逻辑与事务处理进行解耦。也就是让我们的业务代码逻辑不受污染或少量污染,就可以实现事务控制。

在SpringBoot项目中,其内部提供了事务的自动配置，当我们在项目中添加了指定依赖spring-boot-starter-jdbc时，框架会自动为我们的项目注入事务管理器对象，最常用的为DataSourceTransactionManager对象。

### 4.2.2 Spring中事务管理实现

重点讲解实际项目中最常用的注解方式的事务管理，以注解@Transactional配置方式为例，进行实践分析。

基于@Transactional 注解进行声明式事务管理的实现步骤分为两步：

1. 启用声明式事务管理，在配置类上添加@EnableTransactionManagement，新版本中也可不添加（例如新版Spring Boot项目）。
2. 将@Transactional注解添加到合适的业务类或方法上，并设置合适的属性信息。

```Java
 @Transactional(timeout = 30,
               readOnly = false,
               isolation = Isolation.READ_COMMITTED,
               rollbackFor = Throwable.class,
               propagation = Propagation.REQUIRED)
  @Service
  public class SysUserServiceImpl implements SysUserService {
        @Transactional(readOnly = true)
    	@Override
        public PageObject<SysUserDeptVo> findPageObjects(
            String username, Integer pageCurrent) {
            ...
       }
}
```

其中，代码中的@Transactional注解用于描述类或方法，告诉spring框架我们要在此类的方法执行时进行事务控制，其具体说明如下：。

- 当@Transactional注解应用在类上时表示类中所有方法启动事务管理，并且一般用于事务共性的定义。
- 当@Transactional描述方法时表示此方法要进行事务管理，假如类和方法上都有@Transactional注解，则方法上的注解一般用于事务特性的定义。

**@Transactional 常用属性应用说明：**

- timeout：事务的超时时间，默认值为-1,表示没有超时显示。如果配置了具体时间,则超过该时间限制但事务还没有完成，则自动回滚事务。
- read-only：指定事务是否为只读事务，默认值为 false；为了忽略那些不需要事务的方法，比如读取数据，可以设置 read-only 为 true。
- rollback-for：用于指定能够触发事务回滚的异常类型，如果有多个异常类型需要指定，各类型之间可以通过逗号分隔。
- no-rollback- for ：抛出 no-rollback-for 指定的异常类型，不回滚事务。
- isolation：事务的隔离级别，默认值采用 DEFAULT。
- propagation：事务的传播特性

**Spring 中事务控制过程分析,如图所示:**

![image-20200507105641288](E:\Git\AnnerYang.github.io\images\posts\java\image-20200507105641288.png)

Spring事务管理是基于接口代理（JDK）或动态字节码（CGLIB）技术，然后通过AOP实施事务增强的。当我们执行添加了事务特性的目标方式时，系统会通过目标对象的代理对象调用DataSourceTransactionManager对象，在事务开始的时，执行doBegin方法，事务结束时执行doCommit或doRollback方法。

### 4.2.3 Spring 中事务传播特性

事务传播(Propagation)特性指"不同业务(service)对象"中的事务方法之间相互调用时,事务的传播方式,如图所示:

![image-20200507212330873](E:\Git\AnnerYang.github.io\images\posts\java\image-20200507212330873.png)

其中,常用事务传播方式如下:

- **@Transactional(propagation=Propagation.REQUIRED) 。**

如果没有事务创建新事务, 如果当前有事务参与当前事务, Spring 默认的事务传播行为是PROPAGATION_REQUIRED，它适合于绝大多数的情况。假设 ServiveX#methodX() 都工作在事务环境下（即都被 Spring 事务增强了），假设程序中存在如下的调用链：

Service1#method1()->Service2#method2()->Service3#method3()，那么这 3 个服务类的 3 个方法通过 Spring 的事务传播机制都工作在同一个事务中。如图所示：

![image-20200507212406128](E:\Git\AnnerYang.github.io\images\posts\java\image-20200507212406128.png)

代码示例如下：

```Java
@Transactional(propagation = Propagation.REQUIRED)
@Override
public List<Node> findZtreeMenuNodes() {
    return sysMenuDao.findZtreeMenuNodes();
}
```

当有一个业务对象调用如上方法时，此方法始终工作在一个已经存在的事务方法，或者是由调用者创建的一个事务方法中。

- **@Transactional(propagation=Propagation.REQUIRES_NEW)。**

必须是新事务, 如果有当前事务, 挂起当前事务并且开启新事务，如图所示：

![image-20200507212455323](E:\Git\AnnerYang.github.io\images\posts\java\image-20200507212455323.png)

代码示例如下：

```Java
@Transactional(propagation = Propagation.REQUIRES_NEW)
@Override
public void saveObject(SysLog entity) {
    sysLogDao.insertObject(entity);
}
```

当有一个业务对象调用如上业务方法时，此方法会始终运行在一个新的事务中。

## 4.3 Spring 中事务管理小结

Spring 声明式事务是 Spring 最核心，最常用的功能。由于 Spring 通过 IOC 和 AOP的功能非常透明地实现了声明式事务的功能，对于一般的开发者基本上无须了解 Spring声明式事务的内部细节，仅需要懂得如何配置就可以了。但对于中高端开发者还需要了解其内部机制。

# 5 Spring AOP异步操作实现

## 5.1 异步场景分析

在开发系统的过程中，通常会考虑到系统的性能问题，提升系统性能的一个重要思想就是“串行”改“并行”。说起“并行”自然离不开“异步”，今天我们就来聊聊如何使用Spring的@Async的异步注解。

## 5.2 Spring 业务的异步实现

### 5.2.1启动异步配置

在基于注解方式的配置中，借助@EnableAsync注解进行异步启动声明，Spring Boot版的项目中，将@EnableAsync注解应用到启动类上，代码示例如下：

```java
@EnableAsync //表示启动异步配置,容器启动时会初始化一个线程池对象
@SpringBootApplication
public class Application {
   public static void main(String[] args) {
     	SpringApplication.run(Application.class, args);
   }
}
```

### 5.2.2 Spring中@Async注解应用

在需要异步执行的业务方法上，使用@Async方法进行异步声明。

```java
@Async
@Transactional(propagation = Propagation.REQUIRES_NEW)
@Override
public void saveObject(SysLog entity) {
    System.out.println("SysLogServiceImpl.save:" + Thread.currentThread().getName());
    sysLogDao.insertObject(entity);
    //try{Thread.sleep(5000);}catch(Exception e) {}
}
```

假如需要获取业务层异步方法的执行结果，可参考如下代码设计进行实现：

```java
@Transactional(propagation = Propagation.REQUIRES_NEW)
@Async
@Override
public Future<Integer> saveObject(SysLog entity) {
    System.out.println("SysLogServiceImpl.save:" + Thread.currentThread().getName());
    int rows=sysLogDao.insertObject(entity);
    //try{Thread.sleep(5000);}catch(Exception e) {}
    return new AsyncResult<Integer>(rows);
}
```

其中，AsyncResult对象可以对异步方法的执行结果进行封装，假如外界需要异步方法结果时，可以通过Future对象的get方法获取结果。

当我们需要自己对spring框架提供的连接池进行一些简易配置，可以参考如下代码：

```xml
spring:
  task:
    execution:
      pool:
		core-size: 5 		#核心线程数
		max-size: 128		#最大线程数
        queue-capacity: 128 #任务队列容量(核心线程数没有空闲的话,任务会存放到任务队列)      
        keep-alive: 60000	#线程空闲时长
      thread-name-prefix: db-service-task-	#线程前缀
```

线程池配置的执行逻辑:

```Java
ThreadPoolExecutor executor = new ThreadPoolExecutor(
                2,          //corePoolSize 核心线程数
                3,      //maximumPoolSize最大线程数
                30,        //maximumPoolSize 最大空闲空间
                TimeUnit.SECONDS,       //时间单位
                workQueue,              //工作队列/任务队列
                threadFactory,          //创建线程对象的工程
                new ThreadPoolExecutor.CallerRunsPolicy()//当池中已经达到最大吞吐量,可以设置拒绝此略,此处是设置为使用主线程来执行);
```

​	最大核心线程数**core-size < 5**时,每次处理任务都会创建一个核心线程一直达到**core-size =5**,再有任务请求时任务会存放在任务队列中**queue-capacity**,当任务队列满了之后**queue-capacity=128**,所有的线程又都在忙,此时再来新的任务创建新线程一直到**max-size=128**,当第129个任务请求执行时主线程会拒绝执行次线程Exception in thread "main" java.util.concurrent.RejectedExecutionException,可以通过设置拒绝策略来处理().

对于spring框架中线程池配置参数的涵义，可以参考ThreadPoolExecutor对象中的解释。

说明：对于@Async注解默认会基于ThreadPoolTaskExecutor对象获取工作线程，然后调用由@Async描述的方法，让方法运行于一个工作线程，以实现异步操作。但是假如系统中的默认拒绝处理策略,任务执行过程的异常处理不能满足我们自身业务需求的话,我可以对异步线程池进行自定义.(SpringBoot中默认的异步配置可以参考自动配置对象TaskExecutionAutoConfiguration).

### 5.2.3 Spring 自定义异步池的实现(拓展)

为了让Spring中的异步池更好的服务于我们的业务,同时也尽量避免OOM，可以自定义线程池优化设计如下：关键代码如下：

```Java

```

其中:@ConfigurationProperties("async-thread-pool")的含义是读取application.yml配置文件中以"async-thread-pool"名为前缀的配置信息,并通过所描述类的set方法赋值给对应的属性,在application.yml中连接器池的关键配置如下:

```xml
async-thread-pool:
       corePoolSize: 20
       maxPoolSize: 1000
       keepAliveSeconds: 30
       queueCapacity: 1000
```

后续在业务类中,假如我们使用@Async注解描述业务方法，默认会使用ThreadPoolTaskExecutor池对象中的线程执行异步任务。







# 6 Spring AOP中Cache操作实现

## 6.1 缓存场景分析

在业务方法中我们可能调用数据层方法获取数据库中数据,假如访问数据的频率比较为高,为了提高的查询效率,降低数据库的访问压力,可以在业务层对数据进行缓存.

# 6.2 Spring 中业务缓存应用实现

### 6.2.1 启动缓存配置

在项目(SpringBoot项目)的启动类上添加@EnableCaching注解,以启动缓存配置。关键代码如下：

```Java
package com.cy;

/**
 * 异步的自动配置生效).
 * @EnableCaching 注解表示启动缓存配置
 */
@EnableCaching
@SpringBootApplication
public class Application {
        public static void main(String[] args) {
                SpringApplication.run(Application.class, args);
        }
}
```

### 6.2.2 业务方法上应用缓存配置

在需要进行缓存的业务方法上通过@Cacheable注解对方法进行相关描述.表示方法的

返回值要存储到Cache中,假如在更新操作时需要将cache中的数据移除,可以在更新方法上使用@CacheEvict注解对方法进行描述。例如：

第一步:在相关模块查询相关业务方法中,使用缓存，关键代码如下：

```Java
@Cacheable(value = "deptCache")
@Transactional(readOnly = true)
public List<Map<String,Object>> findObjects() {
....
}
```

其中，value属性的值表示要使用的缓存对象,名字自己指定，其中底层为一个map对象，当向cache中添加数据时，key默认为方法实际参数的组合。

第二步：在相关模块更新时,清除指定缓存数据,关键代码如下:

```Java
@CacheEvict(value="deptCache",allEntries=true)
@Override
public int saveObject(SysDept entity) {...}
```

其中，allEntries表示清除所有。

说明：spring中的缓存应用原理，如图所示：

![image-20200508090900740](E:\Git\AnnerYang.github.io\images\posts\java\image-20200508090900740.png)

### 6.2.3 Spring中自定义缓存的实现(拓展)

在Spring中默认cache底层实现是一个Map对象,假如此map对象不能满足我们实际需要,在实际项目中我们可以将数据存储到第三方缓存系统中.