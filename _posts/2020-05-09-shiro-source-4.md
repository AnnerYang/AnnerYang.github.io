---
layout: post
title: Shrio源码分析(4) - 数据域(Realm)
categories: Shiro
description: Shiro源码分析(4) - 数据域(Realm)
keywords: Shiro, Realm
---

摘要:原创出处 https://my.oschina.net/xiaoqiyiye/blog/1618279 「xiaoqiyiye」欢迎转载，保留摘要，谢谢！

本篇主要分析Shiro中的Realm接口。Shiro使用Realm接口作为外部数据源，主要处理认证和授权工作。

Realm接口如下:

```java
public interface Realm {

    /**
     * Realm必须要有一个唯一的名称
     */
    String getName();

    /**
     * 判断该Realm是否支持处理给定的token认证
     */
    boolean supports(AuthenticationToken token);

    /**
     * 认证token，并返回已认证的AuthenticationInfo
     * 如果没有账户可以认证，返回null，如果认证失败抛出异常
     */
    AuthenticationInfo getAuthenticationInfo(AuthenticationToken token) throws AuthenticationException;
}
```

## CachingRealm抽象类

CachingRealm是带有缓存功能的Realm抽象实现。在CachingRealm中提供了对Realm进行缓存功能的缓存管理器CacheManager，但并没有实现具体缓存什么。在CachingRealm中提供了对onLogout的处理，该方法从LogoutAware实现来，用来处理用户登出后清理缓存数据。Shiro默认对Realm开启缓存功能。

```java
// Realm名称
private String name;
// 是否开启缓存，默认构造方法开启缓存
private boolean cachingEnabled;
// 缓存管理器
private CacheManager cacheManager;

public CachingRealm() {
    this.cachingEnabled = true;
    this.name = getClass().getName() + "_" + INSTANCE_COUNT.getAndIncrement();
}
```

值得一提的是afterCacheManagerSet()这个钩子方法，在设置缓存处理器后会调用这个方法，在后面的分析中会由子类重写。

```java
public void setCacheManager(CacheManager cacheManager) {
	this.cacheManager = cacheManager;
	afterCacheManagerSet();
}

protected void afterCacheManagerSet() {
}
```

## AuthenticatingRealm抽象类

AuthenticatingRealm是一个可认证的Realm抽象实现类。 AuthenticatingRealm继承了CachingRealm，并实现了Initializable。Initializable提供的init()方法在初始化时会调用。下面是AuthenticatingRealm的属性和构造方法。

```java
// 凭证匹配器，用来匹配凭证是否正确
private CredentialsMatcher credentialsMatcher;

// 缓存通过认证的认证数据
private Cache<Object, AuthenticationInfo> authenticationCache;

// 是否认证缓存
private boolean authenticationCachingEnabled;

// 认证缓存的名称
private String authenticationCacheName;

/**
 * 定义Realm支持的AuthenticationToken类型
 */
private Class<? extends AuthenticationToken> authenticationTokenClass;

public AuthenticatingRealm() {
    this(null, new SimpleCredentialsMatcher());
}

public AuthenticatingRealm(CacheManager cacheManager) {
    this(cacheManager, new SimpleCredentialsMatcher());
}

public AuthenticatingRealm(CredentialsMatcher matcher) {
    this(null, matcher);
}

public AuthenticatingRealm(CacheManager cacheManager, CredentialsMatcher matcher) {
    // 默认支持UsernamePasswordToken类型
    authenticationTokenClass = UsernamePasswordToken.class;

    // 认证不缓存
    this.authenticationCachingEnabled = false;

    // 设置认证缓存的名称
    int instanceNumber = INSTANCE_COUNT.getAndIncrement();
    this.authenticationCacheName = getClass().getName() + DEFAULT_AUTHORIZATION_CACHE_SUFFIX;
    if (instanceNumber > 0) {
        this.authenticationCacheName = this.authenticationCacheName + "." + instanceNumber;
    }

    // 设置缓存管理器
    if (cacheManager != null) {
        setCacheManager(cacheManager);
    }
    
    // 设置凭证匹配器
    if (matcher != null) {
        setCredentialsMatcher(matcher);
    }
}
```

从属性和构造方法我们可以看出，AuthenticatingRealm会进行认证，对认证的结果AuthenticationInfo进行缓存，认证时需要使用凭证匹配器来匹配凭证是否正确。下面，我们根据这个思路可以去看看进行认证的方法getAuthenticationInfo(AuthenticationToken token)。

```java
public final AuthenticationInfo getAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {

    // 从认证缓存中获取认证结果
    AuthenticationInfo info = getCachedAuthenticationInfo(token);
    if (info == null) {
        // 这是一个抽象方法，子类去完成认证过程
        info = doGetAuthenticationInfo(token);
        if (token != null && info != null) {
            // 如果认证通过，则将认证结果缓存起来
            cacheAuthenticationInfoIfPossible(token, info);
        }
    }

    // 匹配凭证是否正确，如果不正确将会抛出异常
    if (info != null) {
        assertCredentialsMatch(token, info);
    }

    return info;
}
```

关于AuthenticationInfo缓存过程中的一些细节。在缓存的过程中是以AuthenticationToken中的身份进行缓存的，所有身份肯定要是唯一的。属性authenticationCache可以由外部提供，也可以通过缓存管理器生成，一般情况下authenticationCache不需要外部设置。

## AuthorizingRealm抽象类

AuthorizingRealm继承了AuthenticatingRealm，负责处理角色和权限。AuthorizingRealm的实现方式和AuthenticatingRealm一样，提供了一个抽象的doGetAuthorizationInfo(PrincipalCollection principals)方法。这里不做详细介绍，我们会在后面分析角色权限时介绍。

## 基于Jdbc的Realm(JdbcRealm类)

JdbcRealm类可以直接和数据库连接，从数据中获取用户名、密码、角色、权限等数据信息。通过和数据库的直接连接来判断认证是否正确，是否有角色权限功能。

在JdbcRealm中提供了一些Sql语句常量，通过这些sql来做数据库操作。当然，操作数据肯定需要数据库数据源。

```java
// 通过用户名查找密码的Sql语句
protected static final String DEFAULT_AUTHENTICATION_QUERY = "select password from users where username = ?";

/**
 * 通过用户名称查找密码和加密盐的Sql语句
 */
protected static final String DEFAULT_SALTED_AUTHENTICATION_QUERY = "select password, password_salt from users where username = ?";

/**
 * 通过用户名查找用户所有角色
 */
protected static final String DEFAULT_USER_ROLES_QUERY = "select role_name from user_roles where username = ?";

/**
 * 通过角色名称查找角色拥有的权限
 */
protected static final String DEFAULT_PERMISSIONS_QUERY = "select permission from roles_permissions where role_name = ?";

/**
 * 定义了几种加盐模式：
 *   NO_SALT - 密码没有加密盐
 *   CRYPT - unix加密（这种模式目前还支持）
 *   COLUMN - 加密盐存储在数据库表字段中 
 *   EXTERNAL - 加密盐没有存储在数据库
 */
public enum SaltStyle {NO_SALT, CRYPT, COLUMN, EXTERNAL};

// 数据库数据源
protected DataSource dataSource;

// 查询密码和加密盐的SQL
protected String authenticationQuery = DEFAULT_AUTHENTICATION_QUERY;

// 查询用户角色的SQL
protected String userRolesQuery = DEFAULT_USER_ROLES_QUERY;

// 查询角色拥有权限的SQL
protected String permissionsQuery = DEFAULT_PERMISSIONS_QUERY;

protected boolean permissionsLookupEnabled = false;

// 密码没有加密盐模式
protected SaltStyle saltStyle = SaltStyle.NO_SALT;
```

对于不同的数据库，这些默认的Sql是可以更改的，JdbcRealm都提供了相应的setter方法。那么，Jdbc是如何认证和获取角色权限的呢？下面继续分析doGetAuthenticationInfo和doGetAuthorizationInfo这两个方法。

1. 认证过程

```java
protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {

    // 只支持UsernamePasswordToken类型
    UsernamePasswordToken upToken = (UsernamePasswordToken) token;
    
    // 用户名
    String username = upToken.getUsername();

    // 用户名空判断
    if (username == null) {
        throw new AccountException("Null usernames are not allowed by this realm.");
    }

    Connection conn = null;
    SimpleAuthenticationInfo info = null;
    try {
    
        // 获取数据库连接
        conn = dataSource.getConnection();

        String password = null;
        String salt = null;
        switch (saltStyle) {
            case NO_SALT:
                password = getPasswordForUser(conn, username)[0];
                break;
            case CRYPT:
                // TODO: separate password and hash from getPasswordForUser[0]
                throw new ConfigurationException("Not implemented yet");
                //break;
            case COLUMN:
                String[] queryResults = getPasswordForUser(conn, username);
                password = queryResults[0];
                salt = queryResults[1];
                break;
            case EXTERNAL:
                password = getPasswordForUser(conn, username)[0];
                // 以用户名作为加密盐
                salt = getSaltForUser(username);
        }

        if (password == null) {
            throw new UnknownAccountException("No account found for user [" + username + "]");
        }

        // 创建一个认证信息
        info = new SimpleAuthenticationInfo(username, password.toCharArray(), getName());
        
        if (salt != null) {
            info.setCredentialsSalt(ByteSource.Util.bytes(salt));
        }

    } catch (SQLException e) {
        throw new AuthenticationException(message, e);
    } finally {
        // 关闭数据连接
        JdbcUtils.closeConnection(conn);
    }

    return info;
}
```

2 授权过程

```java
protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principals) {

    // 身份不能为空
    if (principals == null) {
        throw new AuthorizationException("PrincipalCollection method argument cannot be null.");
    }

    // 从身份中获取用户名
    String username = (String) getAvailablePrincipal(principals);

    Connection conn = null;
    Set<String> roleNames = null;
    Set<String> permissions = null;
    try {
        // 获取数据库连接
        conn = dataSource.getConnection();

        // 获取角色集合
        roleNames = getRoleNamesForUser(conn, username);
        if (permissionsLookupEnabled) {
            // 获取权限集合
            permissions = getPermissions(conn, username, roleNames);
        }
    } catch (SQLException e) {
        throw new AuthorizationException(message, e);
    } finally {
        JdbcUtils.closeConnection(conn);
    }

    // 返回带有角色权限的认证信息
    SimpleAuthorizationInfo info = new SimpleAuthorizationInfo(roleNames);
    info.setStringPermissions(permissions);
    return info;

}
```

在Shiro中还提供了一些其他的Realm。SimpleAccountRealm、TextConfigurationRealm、IniRealm、PropertiesRealm。这里就不一一介绍了，有兴趣可以自己去看。

## 总结

在Shiro中Realm接口作为一个与应用程序外接的接口，可以通过Realm提供认证和授权的数据信息。在开发使用中最常用的就是从AuthenticatingRealm或AuthorizingRealm抽象类来实现业务中具体的Realm实例。doGetAuthenticationInfo(AuthenticationToken token)处理认证过程，doGetAuthorizationInfo(PrincipalCollection principals)处理授权过程。