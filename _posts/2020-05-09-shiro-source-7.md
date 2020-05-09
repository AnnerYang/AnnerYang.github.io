---
layout: post
title: Shiro源码分析(7) - 安全管理器(SecurityManager)
categories: Shiro
description: Shiro源码分析(7) - 安全管理器(SecurityManager)
keywords: Shiro, SecurityManager
---

摘要:原创出处 https://my.oschina.net/xiaoqiyiye/blog/1618279 「xiaoqiyiye」欢迎转载，保留摘要，谢谢！

在Shiro中SecurityManager是最核心的组件，Shiro架构的心脏，用来协调内部各安全组件，管理内部组件实例，并通过它来提供安全管理的各种服务。当Shiro与一个Subject进行交互时，实质上是幕后的SecurityManager处理所有繁重的Subject安全操作。

SecurityManager继承了3个核心接口Authenticator, Authorizer, SessionManager，这3个接口我们在前面都已经详细的分析过了。另外SecurityManager还提供了用于登入登出和创建Subject的方法。

```java
// 使用用户提供的AuthenticationToken进行登录
Subject login(Subject subject, AuthenticationToken authenticationToken) throws AuthenticationException;

// 退出当前Subject用户
void logout(Subject subject);

// 创建Subject实例
Subject createSubject(SubjectContext context);
```

在SecurityManager的实现类中，逐一实现了缓存功能(CachingSecurityManager)，Realm功能(RealmSecurityManager)，认证功能(AuthenticatingSecurityManager)，授权功能(AuthorizingSecurityManager)和Session功能(SessionsSecurityManager)。上面的这些类都是抽象类，真正实现的是DefaultSecurityManager类。下面我们先逐一对这些抽象类进行分析，最后分析DefaultSecurityManager。

## 缓存功能的实现

CachingSecurityManager实现SecurityManager接口的同时，还实现了Destroyable和CacheManagerAware。引用缓存管理器CacheManager来管理缓存，在设置缓存管理器时提供了一个钩子方法afterCacheManagerSet()用于做设置后的操作。另外，实现Destroyable的destroy()，用于当对象销毁时，清除缓存管理器中管理的缓存。

在Shiro 1.3版本之后还引入了事件总线(EventBus)的概念。

## Realm功能的实现

RealmSecurityManager继承自CachingSecurityManager，在原有功能上添加了对Realm的操作管理。RealmSecurityManager主要是对Realm的设置，但设置Realm时关联已存在的缓存管理器到Realm对象（如果Realm实现了CacheManagerAware接口）。下面是相关方法。

```java
public void setRealms(Collection<Realm> realms) {
	if (realms == null) {
		throw new IllegalArgumentException("Realms collection argument cannot be null.");
	}
	if (realms.isEmpty()) {
		throw new IllegalArgumentException("Realms collection argument cannot be empty.");
	}
	this.realms = realms;
	afterRealmsSet();
}

protected void afterRealmsSet() {
    //关联缓存管理器到Realms中去 
	applyCacheManagerToRealms();
	applyEventBusToRealms();
}

protected void applyCacheManagerToRealms() {
	CacheManager cacheManager = getCacheManager();
    // 判断Realm是否实现了CacheManagerAware接口，如果实现则设置到Realm中
	Collection<Realm> realms = getRealms();
	if (cacheManager != null && realms != null && !realms.isEmpty()) {
		for (Realm realm : realms) {
			if (realm instanceof CacheManagerAware) {
				((CacheManagerAware) realm).setCacheManager(cacheManager);
			}
		}
	}
}
```

另外，还重写了destroy()方法。如果Realm实现了Destroyable接口在销毁时就处理。

```java
public void destroy() {
    LifecycleUtils.destroy(getRealms());
    this.realms = null;
    super.destroy();
}
```

## 认证功能的实现

在AuthenticatingSecurityManager抽象类中实现了SecurityManager的认证功能。虽然说SecurityManager实现了Authenticator接口，但是在AuthenticatingSecurityManager中并没有真正地对Authenticator接口进行实现，而是通过组合的形式委派给一个Authenticator类型的属性来处理。很明显，这个委派的对象就是我们前面分析过的ModularRealmAuthenticator。

```java
    public AuthenticatingSecurityManager() {
        super();
        this.authenticator = new ModularRealmAuthenticator();
    }
```

在AuthenticatingSecurityManager中需要注意，需要将设置realms信息设置到ModularRealmAuthenticator中去。

```java
protected void afterRealmsSet() {
	super.afterRealmsSet();
    // 判断是否为ModularRealmAuthenticator认证接口
	if (this.authenticator instanceof ModularRealmAuthenticator) {
		((ModularRealmAuthenticator) this.authenticator).setRealms(getRealms());
	}
}
```

真实的认证过程由Authenticator接口来处理，也就是ModularRealmAuthenticator实例对象。

```java
    public AuthenticationInfo authenticate(AuthenticationToken token) throws AuthenticationException {
        return this.authenticator.authenticate(token);
    }
```

## 授权功能的实现

AuthorizingSecurityManager的实现方式和AuthenticatingSecurityManager是一样的，授权处理工作都委托给Authorizer去实现。

```java
public abstract class AuthorizingSecurityManager extends AuthenticatingSecurityManager {

    // 授权器
    private Authorizer authorizer;

    public AuthorizingSecurityManager() {
        super();
        // 默认提供的授权器
        this.authorizer = new ModularRealmAuthorizer();
    }

    protected void afterRealmsSet() {
        super.afterRealmsSet();
        // 关联realms到授权器
        if (this.authorizer instanceof ModularRealmAuthorizer) {
            ((ModularRealmAuthorizer) this.authorizer).setRealms(getRealms());
        }
    }

    public void destroy() {
        // 销毁授权器相关资源
        LifecycleUtils.destroy(getAuthorizer());
        this.authorizer = null;
        super.destroy();
    }
    
    // 省略了Authorizer接口的实现方法，所有实现都委派给authorizer去处理。
}
```

## Session功能的实现

SessionsSecurityManager的实现和上面也是一样的，具体的实现交由SessionManager来实现。

```java
public abstract class SessionsSecurityManager extends AuthorizingSecurityManager {

    // 实现Session管理功能
    private SessionManager sessionManager;

    public SessionsSecurityManager() {
        super();
        this.sessionManager = new DefaultSessionManager();
        applyCacheManagerToSessionManager();
    }

    public void setSessionManager(SessionManager sessionManager) {
        this.sessionManager = sessionManager;
        afterSessionManagerSet();
    }

    protected void afterSessionManagerSet() {
        applyCacheManagerToSessionManager();
    }

    protected void afterCacheManagerSet() {
        super.afterCacheManagerSet();
        applyCacheManagerToSessionManager();
    }

    protected void applyCacheManagerToSessionManager() {
        // 关联缓存管理器到Session管理器
        if (this.sessionManager instanceof CacheManagerAware) {
            ((CacheManagerAware) this.sessionManager).setCacheManager(getCacheManager());
        }
    }

    public void destroy() {
        LifecycleUtils.destroy(getSessionManager());
        this.sessionManager = null;
        super.destroy();
    }
}
```

总的来说，上面的分析都很简单，SecurityManager虽然实现了Authenticator, Authorizer, SessionManager接口，但都是通过委派的方式来实现的。这样可以更好的扩展各个接口的功能实现。在Shiro中有两个实现类DefaultSecurityManager和DefaultWebSecurityManager是我们使用的实例对象。下面我们看看DefaultSecurityManager是如何作为Shiro的核心协调整个服务的。

## 安全管理器(DefaultSecurityManager)

我们还是从属性和构造方法来了解在DefaultSecurityManager想要做哪些功能。

```java
// rememberMe管理器
protected RememberMeManager rememberMeManager;
// 用来存储Subject对象的，和SessionDAO类似。
protected SubjectDAO subjectDAO;
// 从名称上看就知道是创建Subject实例的工厂对象
protected SubjectFactory subjectFactory;

public DefaultSecurityManager() {
	super();
	this.subjectFactory = new DefaultSubjectFactory();
	this.subjectDAO = new DefaultSubjectDAO();
}

public DefaultSecurityManager(Realm singleRealm) {
	this();
	setRealm(singleRealm);
}

public DefaultSecurityManager(Collection<Realm> realms) {
	this();
	setRealms(realms);
}
```

从上面的代码中我们会发现，由SubjectFactory来创建Subject，由SubjectDAO来存储Subject。使用RememberMeManager来管理rememberMe功能。

### 登录流程

登录流程很简单。首先进行认证，然后创建一个登录的Subject实例返回，登录成功后会处理rememberMe。我们把思绪返回到[Shrio源码分析(5) - 用户主体(Subject)](https://my.oschina.net/u/3409112/blog/1619214)中的如下代码片段中。

```java
// 委派给SecurityManager去执行登录，如果登录成功会返回一个
// 携带有认证成功数据的Subject对象
Subject subject = securityManager.login(this, token);
```

下面就开始分析securityManager.login(this, token)方法。

```java
public Subject login(Subject subject, AuthenticationToken token) throws AuthenticationException {
    AuthenticationInfo info;
    try {
        // 认证获得AuthenticationInfo信息，如果异常，处理失败情况
        info = authenticate(token);
    } catch (AuthenticationException ae) {
        try {
            // 登录失败后处理rememberMe
            onFailedLogin(token, ae, subject);
        } catch (Exception e) {
        }
        // 注意，异常会继续抛出
        throw ae;
    }

    // 认证成功就创建一个已经登录的Subject实例
    // 注意: subject就是用户登录时调用subject.login(token)的那个实例
    Subject loggedIn = createSubject(token, info, subject);

    // 登录成功后处理rememberMe
    onSuccessfulLogin(token, info, loggedIn);

    // 返回登录成功后的Subject实例，这个实例的信息会设置到subject中去
    return loggedIn;
}

protected Subject createSubject(AuthenticationToken token, AuthenticationInfo info, Subject existing) {
    // 创建Subject上下文，设置相关的认证信息
    SubjectContext context = createSubjectContext();
    context.setAuthenticated(true);
    context.setAuthenticationToken(token);
    context.setAuthenticationInfo(info);
    // 把登录的Subject也关联到上下文
    if (existing != null) {
        context.setSubject(existing);
    }
    // 真实的去创建一个Subject实例
    return createSubject(context);
}

public Subject createSubject(SubjectContext subjectContext) {
    // 拷贝一份SubjectContext，不去修改传进来的参数
    SubjectContext context = copy(subjectContext);

    // 确保存在SecurityManager实例，如果没有就从SecurityUtils.getSecurityManager()获取
    context = ensureSecurityManager(context);

    // 为context设置Session
    context = resolveSession(context);

    // 为context设置身份信息
    context = resolvePrincipals(context);

    // 创建Subject，是由SubjectFactory去创建的
    Subject subject = doCreateSubject(context);

    // 使用SubjectDAO保存subject
    save(subject);

    return subject;
}
```

在上面，我们还有一些细节没有分析到。在创建Subject时，SubjectContext中的上下文数据很重要，特别是身份信息是怎么获取的？rememberMe是怎么实现的？感兴趣的可以自己去分析。

### 登出流程

登出流程主要是做一些数据的清除销毁。包括rememberMe登出处理，认证器的登出处理，删除Subject，停止Session等工作。

```java
public void logout(Subject subject) {

    if (subject == null) {
        throw new IllegalArgumentException("Subject method argument cannot be null.");
    }

    // 登出前置处理，处理rememberMe功能的登出
    beforeLogout(subject);

    PrincipalCollection principals = subject.getPrincipals();
    if (principals != null && !principals.isEmpty()) {
        // 认证器登出处理，只要是清理缓存的一些身份等信息，发起监听器监听登出操作
        Authenticator authc = getAuthenticator();
        if (authc instanceof LogoutAware) {
            ((LogoutAware) authc).onLogout(principals);
        }
    }

    try {
        // 从SubjectDAO中删除Subject
        delete(subject);
    } catch (Exception e) {
        if (log.isDebugEnabled()) {
            String msg = "Unable to cleanly unbind Subject.  Ignoring (logging out).";
            log.debug(msg, e);
        }
    } finally {
        try {
            // 将Subject关联的Session停用
            stopSession(subject);
        } catch (Exception e) {
            if (log.isDebugEnabled()) {
                String msg = "Unable to cleanly stop Session for Subject [" + subject.getPrincipal() + "] " +
                        "Ignoring (logging out).";
                log.debug(msg, e);
            }
        }
    }
}
```