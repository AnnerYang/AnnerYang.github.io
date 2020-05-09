---
layout: post
title: Shiro源码分析(3) - 认证器(Authenticator)
categories: Shiro
description: Shiro源码分析(3) - 认证器(Authenticator)
keywords: Shiro, Authenticator
---

摘要:原创出处 https://my.oschina.net/xiaoqiyiye/blog/1618279 「xiaoqiyiye」欢迎转载，保留摘要，谢谢！

Authenticator就是认证器，在Shiro中负责认证用户提交的信息，在Shiro中我们用AuthenticationToken来表示提交的信息。Authenticator接口只提供了一个认证的方法。如下。

```java
/**
 * 认证用户提交的信息AuthenticationToken对象，AuthenticationToken包含了身份和凭证。
 * 如果认证成功则返回AuthenticationInfo对象，AuthenticationInfo对象代表了用户在Shiro中已经被认证过的账户数据。
 * 如果认证失败则抛出一下异常
 * @see ExpiredCredentialsException     凭证过期
 * @see IncorrectCredentialsException   凭证错误
 * @see ExcessiveAttemptsException      多次尝试失败
 * @see LockedAccountException          账户锁定
 * @see ConcurrentAccessException       并发访问异常(多点登录)
 * @see UnknownAccountException         账户未知
 */
public AuthenticationInfo authenticate(AuthenticationToken authenticationToken)
        throws AuthenticationException;
```

## AuthenticationToken分析

AuthenticationToken对象代表了身份和凭证。从下面的接口看提供的方法很简单，但返回的对象都是Object，也就是说在Shiro中对身份和凭证的类型没有限制，Shiro没有提供特有的类型来处理。

```java
public interface AuthenticationToken extends Serializable {

    /**
     *  获取身份
     */
    Object getPrincipal();

    /**
     *  获取凭证
     */
    Object getCredentials();

}
```

在Shiro中只提供了一种具体的实现类UsernamePasswordToken。UsernamePasswordToken类是以用户名作为身份，密码作为凭证。当然它也实现了RememberMeAuthenticationToken接口，提供rememberMe功能。rememberMe功能的实现在后面再分析，在这里不是重点。UsernamePasswordToken很简单，只有构造方法和setter/getter方法。我们需要对UsernamePasswordToken中的身份和凭证要有很好的理解，什么可以作为身份，什么又是凭证。

## AuthenticationInfo分析

AuthenticationInfo表示被Subject存储的账户，这个账户是经过认证的。而AuthenticationToken中的身份/凭证是用户提交的数据，还没有经过认证，如果认证成功才会被存储在AuthenticationInfo中。

AuthenticationInfo只有一个实现类SimpleAuthenticationInfo（备注：SimpleAccount也是其中一个实现类，但功能是完全依赖SimpleAuthenticationInfo实现的）。AuthenticationInfo还有两个子接口，分别是：SaltedAuthenticationInfo和MergableAuthenticationInfo。SaltedAuthenticationInfo提供了获取凭证加密盐的方法，MergableAuthenticationInfo可以合并验证后的身份信息。各自的接口都很简单，下面直接分析SimpleAuthenticationInfo具体实现。

### SimpleAuthenticationInfo详细分析

下面是SimpleAuthenticationInfo的属性和构造方法。下面有很多构造方法，但可以看出实现都依赖到SimplePrincipalCollection对象。SimplePrincipalCollection对象负责收集身份(principals)和域(realm)的关系。关于SimplePrincipalCollection我们暂时不展开分析。

```java
/**
 * 身份集合
 */
protected PrincipalCollection principals;

/**
 * 凭证
 */
protected Object credentials;

/**
 * 凭证盐
 */
protected ByteSource credentialsSalt;

public SimpleAuthenticationInfo() {
}

public SimpleAuthenticationInfo(Object principal, Object credentials, String realmName) {
    this.principals = new SimplePrincipalCollection(principal, realmName);
    this.credentials = credentials;
}

public SimpleAuthenticationInfo(Object principal, Object hashedCredentials, ByteSource credentialsSalt, String realmName) {
    this.principals = new SimplePrincipalCollection(principal, realmName);
    this.credentials = hashedCredentials;
    this.credentialsSalt = credentialsSalt;
}

public SimpleAuthenticationInfo(PrincipalCollection principals, Object credentials) {
    this.principals = new SimplePrincipalCollection(principals);
    this.credentials = credentials;
}

public SimpleAuthenticationInfo(PrincipalCollection principals, Object hashedCredentials, ByteSource credentialsSalt) {
    this.principals = new SimplePrincipalCollection(principals);
    this.credentials = hashedCredentials;
    this.credentialsSalt = credentialsSalt;
}
```

在SimpleAuthenticationInfo中，我们主要分析一下merge(AuthenticationInfo info)方法，也就是说可以合并其他的AuthenticationInfo信息。

```java
public void merge(AuthenticationInfo info) {

    // 判断是否有身份信息，如果没有就返回
    if (info == null || info.getPrincipals() == null || info.getPrincipals().isEmpty()) {
        return;
    }

    // 合并身份集合
    if (this.principals == null) {
        this.principals = info.getPrincipals();
    } else {
        if (!(this.principals instanceof MutablePrincipalCollection)) {
            this.principals = new SimplePrincipalCollection(this.principals);
        }
        ((MutablePrincipalCollection) this.principals).addAll(info.getPrincipals());
    }

    // 凭证盐只是在Realm凭证匹配过程中使用
    // 如果存在凭证盐，就不用管其他的了，如果没有就使用其他的凭证盐
    if (this.credentialsSalt == null && info instanceof SaltedAuthenticationInfo) {
        this.credentialsSalt = ((SaltedAuthenticationInfo) info).getCredentialsSalt();
    }

    // 合并凭证信息
    Object thisCredentials = getCredentials();
    Object otherCredentials = info.getCredentials();

    if (otherCredentials == null) {
        return;
    }

    if (thisCredentials == null) {
        this.credentials = otherCredentials;
        return;
    }

    // 使用集合来合并凭证
    if (!(thisCredentials instanceof Collection)) {
        Set newSet = new HashSet();
        newSet.add(thisCredentials);
        setCredentials(newSet);
    }

    Collection credentialCollection = (Collection) getCredentials();
    if (otherCredentials instanceof Collection) {
        credentialCollection.addAll((Collection) otherCredentials);
    } else {
        credentialCollection.add(otherCredentials);
    }
}
```

## AbstractAuthenticator抽象类

和AbstractSessionManager一样，AbstractAuthenticator主要功能也是提供监听器，对认证过程中的状态进行监听。在认证过程中监听成功、失败、登出情况。监听器是AuthenticationListener，下面看一下监听器提供的方法。

```java
public interface AuthenticationListener {

    /**
     * 监听认证成功
     */
    void onSuccess(AuthenticationToken token, AuthenticationInfo info);

    /**
     * 监听认证失败
     */
    void onFailure(AuthenticationToken token, AuthenticationException ae);

    /**
     * 监听用户登出
     */
    void onLogout(PrincipalCollection principals);
}
```

另外，AbstractAuthenticator实现了Authenticator#authenticate(AuthenticationToken token)，处理了对监听器通知的情况，但执行认证的具体过程提供抽象方法doAuthenticate(AuthenticationToken token)让子类完成。

```java
public final AuthenticationInfo authenticate(AuthenticationToken token) throws AuthenticationException {

    // Token参数异常
    if (token == null) {
        throw new IllegalArgumentException("Method argumet (authentication token) cannot be null.");
    }

    AuthenticationInfo info;
    try {
        // 执行认证过程的抽象方法，子类去实现
        info = doAuthenticate(token);
        if (info == null) {
            String msg = "No account information found for authentication token [" + token + "] by this " +
                    "Authenticator instance.  Please check that it is configured correctly.";
            throw new AuthenticationException(msg);
        }
    } catch (Throwable t) {
        AuthenticationException ae = null;
        if (t instanceof AuthenticationException) {
            ae = (AuthenticationException) t;
        }
        if (ae == null) {
            String msg = "Authentication failed for token submission [" + token + "].  Possible unexpected " +
                    "error? (Typical or expected login exceptions should extend from AuthenticationException).";
            ae = new AuthenticationException(msg, t);
        }
        try {
            // 认证失败了，通知监听器
            notifyFailure(token, ae);
        } catch (Throwable t2) {

        }

        throw ae;
    }

    // 认证成功，通知监听器
    notifySuccess(token, info);

    return info;
}
```

## ModularRealmAuthenticator类分析

在Shiro中只提供了一个具体的实现类ModularRealmAuthenticator，该类可以处理多个Realm的认证方式。

在ModularRealmAuthenticator中，把认证的权利交给域(Realm)去完成，在Shiro中Realm相当于数据的来源，可以自定义。ModularRealmAuthenticator支持多个Realm进行认证，在多个Realm认证时，需要设置认证策略，策略接口是AuthenticationStrategy。在Shiro中提供了三种认证策略。分别是：

1. AllSuccessfulStrategy：所有Realm认证成功。
2. AtLeastOneSuccessfulStrategy：至少有一个Realm认证成功。
3. FirstSuccessfulStrategy： 第一个Realm认证成功。

关于认证策略我们在后面在分析，现在继续分析ModularRealmAuthenticator。我们还是从属性和构造方法分析。

```
// 认证的过程交由Realm去处理
private Collection<Realm> realms;

// 指定认证策略
private AuthenticationStrategy authenticationStrategy;

public ModularRealmAuthenticator() {
    // 默认提供策略：至少有一个Realm认证成功就算认证成功
    this.authenticationStrategy = new AtLeastOneSuccessfulStrategy();
}
```

下面我看看ModularRealmAuthenticator是如何实现doAuthenticate(token)方法的。

```java
protected AuthenticationInfo doAuthenticate(AuthenticationToken authenticationToken) throws AuthenticationException {
    // 判断realms属性，必须要有Realm
    assertRealmsConfigured();
    Collection<Realm> realms = getRealms();
    // 分支：如果只有一个就按照单个流程处理，如果有多个Realm就按照多个流程走认证策略。
    if (realms.size() == 1) {
        return doSingleRealmAuthentication(realms.iterator().next(), authenticationToken);
    } else {
        return doMultiRealmAuthentication(realms, authenticationToken);
    }
}

// 处理单个Realm
protected AuthenticationInfo doSingleRealmAuthentication(Realm realm, AuthenticationToken token) {
    // 判断realm是否支持处理token
    if (!realm.supports(token)) {
        String msg = "Realm [" + realm + "] does not support authentication token [" +
                token + "].  Please ensure that the appropriate Realm implementation is " +
                "configured correctly or that the realm accepts AuthenticationTokens of this type.";
        throw new UnsupportedTokenException(msg);
    }
    
    // realm处理认证过程，处理过程中可能会抛出认证异常AuthenticationException。
    // 如果认证成功info不会用null。
    AuthenticationInfo info = realm.getAuthenticationInfo(token);
    if (info == null) {
        String msg = "Realm [" + realm + "] was unable to find account data for the " +
                "submitted AuthenticationToken [" + token + "].";
        throw new UnknownAccountException(msg);
    }
    return info;
}

// 处理多个Realm
protected AuthenticationInfo doMultiRealmAuthentication(Collection<Realm> realms, AuthenticationToken token) {

    AuthenticationStrategy strategy = getAuthenticationStrategy();

    // 返回一个空的聚合对象
    // AllSuccessfulStrategy - 返回空的SimpleAuthenticationInfo对象
    // AtLeastOneSuccessfulStrategy - 返回空的SimpleAuthenticationInfo对象
    // FirstSuccessfulStrategy - 返回null
    AuthenticationInfo aggregate = strategy.beforeAllAttempts(realms, token);

    for (Realm realm : realms) {
        // 认证前处理
        // AllSuccessfulStrategy - 判断realm.supports(token)，如果不支持直接抛异常，返回aggregate
        // AtLeastOneSuccessfulStrategy - 返回aggregate
        // FirstSuccessfulStrategy - 返回aggregate，也就是null
        aggregate = strategy.beforeAttempt(realm, token, aggregate);

        if (realm.supports(token)) {
            AuthenticationInfo info = null;
            Throwable t = null;
            try {
                //认证过程是由Realm处理的
                info = realm.getAuthenticationInfo(token);
            } catch (Throwable throwable) {
                t = throwable;
                if (log.isDebugEnabled()) {
                    String msg = "Realm [" + realm + "] threw an exception during a multi-realm authentication attempt:";
                    log.debug(msg, t);
                }
            }

            // 认证后处理，
            // AllSuccessfulStrategy - 如果有异常会抛出异常, 如果没有就合并info和aggregate
            // AtLeastOneSuccessfulStrategy - 如果有异常并不会抛出，只是会合并info和aggregate
            // FirstSuccessfulStrategy - 如果aggregate存在，则返回aggregate；否则返回info
            aggregate = strategy.afterAttempt(realm, token, info, aggregate, t);

        } else {
            log.debug("Realm [{}] does not support token {}.  Skipping realm.", realm, token);
        }
    }

    // AllSuccessfulStrategy - 返回aggregate
    // AtLeastOneSuccessfulStrategy - 判断aggregate不为空，否则抛异常
    // FirstSuccessfulStrategy - 返回aggregate
    aggregate = strategy.afterAllAttempts(token, aggregate);

    return aggregate;
}    
```

通过对上面的代码分析：

- AllSuccessfulStrategy策略流程：逐一处理Realm，每个Realm必须支持token处理，然后合并AuthenticationInfo。如果遇到异常，则抛出异常结束循环。
- AtLeastOneSuccessfulStrategy策略流程：逐一处理Realm，不支持处理token的Realm跳过。如果遇到异常，忽略对异常的处理。对于认证通过的AuthenticationInfo进行合并成aggregate，最后判断aggregate，aggregate不能为空，如果有空抛出异常。
- FirstSuccessfulStrategy策略流程：逐一处理Realm，不支持处理token的Realm跳过。如果遇到异常，忽略对异常的处理。在循环处理Realm前aggregate=null，重点是在strategy.afterAttempt(realm, token, info, aggregate, t)的处理上，并不会合并info和aggregate。如果aggregate为空，则返回info。所以在处理后返回的总是第一个认证成功的AuthenticationInfo。

## 总结

Authenticator负责对AuthenticationToken进行认证，然后返回一个已经被认证过的信息AuthenticationInfo。Authenticator也提供了监听器AuthenticationListener，对认证状态进行监听。Authenticator真实的认证过程是由Realm来处理的，可以支持都多个Realm来认证，认证的过程中可以选择不同的认证策略。