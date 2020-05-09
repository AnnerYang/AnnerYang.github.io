---
layout: post
title: Shiro源码分析(1) - Shiro开篇
categories: Shiro
description: Shiro源码分析(1) - Shiro开篇
keywords: Shiro, SecurityUtils
---

Content here

## 简介

- SecurityManager：安全管理器，Shiro最核心组件。Shiro通过SecurityManager来管理内部组件实例，并通过它来提供安全管理的各种服务。
- Authenticator：认证器，认证AuthenticationToken是否有效。
- Authorizer：授权器，处理角色和权限。
- SessionManager：Session管理器，管理Session。
- Subject：当前操作主体，表示当前操作用户。
- SubjectContext：Subject上下文数据对象。
- AuthenticationToken：认证的token信息(用户名、密码等)。
- ThreadContext：线程上下文对象，负责绑定对象到当前线程。

在学习和使用Shiro过程中，我们都知道SecurityManager接口在Shiro中是最为核心的接口。我们就沿着这个接口进行分析。

下面的代码是SecurityManager接口的定义：

```java
public interface SecurityManager extends Authenticator, Authorizer, SessionManager {

    /**
     * 登录
     */
    Subject login(Subject subject, AuthenticationToken authenticationToken) throws AuthenticationException;

    /**
     * 登出
     */
    void logout(Subject subject);

    /**
     * 创建Subject
     */
    Subject createSubject(SubjectContext context);

}
```

在SecurityManager 中定义了三个方法，分别是登录、登出和创建Subject。通常我们在使用的时候是这样使用的。首先创建Subject对象，然后通过调用login方法传入认证信息token对登录进行认证。

```java
Subject subject = SecurityUtils.getSubject(); 
UsernamePasswordToken token = new UsernamePasswordToken("zhang", "123");
subject.login(token);
```

## SecurityUtils分析

在Shiro中提供了一个方便使用的工具类SecurityUtils，SecurityUtils核心功能是获取SecurityManager以及Subject。这两个接口是Shiro提供的外围接口，供开发时使用。

在SecurityUtils使用static定义SecurityManager，也就是说SecurityManager对象在应用中是单一存在的。

```java
private static SecurityManager securityManager;
```

### 1. 获取SecurityManager

首先从ThreadContext中获取，如果没有，则从SecurityUtils属性securityManager中获取。一定要存在一个SecurityManager实例对象，否则抛异常。

```java
public static SecurityManager getSecurityManager() throws UnavailableSecurityManagerException {
	SecurityManager securityManager = ThreadContext.getSecurityManager();
	if (securityManager == null) {
		securityManager = SecurityUtils.securityManager;
	}
	if (securityManager == null) {
		String msg = "No SecurityManager accessible to the calling code, either bound to the " +
				ThreadContext.class.getName() + " or as a vm static singleton.  This is an invalid application " +
				"configuration.";
		throw new UnavailableSecurityManagerException(msg);
	}
	return securityManager;
}
```

### 2. 获取Subject

首先从ThreadContext中获取，如果不存在，则创建新的Subject，再存放到ThreadContext中，以便下次可以获取。

```java
public static Subject getSubject() {
	Subject subject = ThreadContext.getSubject();
	if (subject == null) {
		subject = (new Subject.Builder()).buildSubject();
		ThreadContext.bind(subject);
	}
	return subject;
}
```

在上面的代码中重要是通过 Subject.Builder类提供的buildSubject()方法来创建Subject。在创建Subject时同时还创建了SubjectContext对象，也就是说Subject和SubjectContext是一一对应的。下面的代码是Subject.Builder类的构造方法。

```java
public Builder(SecurityManager securityManager) {
	if (securityManager == null) {
		throw new NullPointerException("SecurityManager method argument cannot be null.");
	}
	this.securityManager = securityManager;
	// 创建了SubjectContext实例对象
	this.subjectContext = newSubjectContextInstance();
	if (this.subjectContext == null) {
		throw new IllegalStateException("Subject instance returned from 'newSubjectContextInstance' " +
				"cannot be null.");
	}
	this.subjectContext.setSecurityManager(securityManager);
}
```

而buildSubject()方法则实际上是调用SecurityManager接口中的createSubject(SubjectContext subjectContext)方法。

```java
public Subject buildSubject() {
	return this.securityManager.createSubject(this.subjectContext);
}
```

## 总结

本篇主要通过SecurityUtils.getSubject()对SecurityManager接口中的createSubject(SubjectContext subjectContext)方法进行了详细的分析。另外两个方法我们在分析Subject时做详细分析。

另外，我们会发现SecurityManager继承了 Authenticator, Authorizer, SessionManager三个接口，这样才能实现SecurityManager提供安全管理的各种服务。在接下来的文章中会对Authenticator, Authorizer, SessionManager分别进行分析，这样我们对SecurityManager基本上就掌握了。