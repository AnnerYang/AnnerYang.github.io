---
layout: post
title: Shiro源码分析(5) - 授权器(Authorizer)
categories: Shiro
description: Shiro源码分析(5) - 授权器(Authorizer)
keywords: Shiro, Authorizer
---

摘要:原创出处 https://my.oschina.net/xiaoqiyiye/blog/1618279 「xiaoqiyiye」欢迎转载，保留摘要，谢谢！

在Shiro中Authorizer接口用来处理用户的角色和权限控制访问操作，其接口代码如下。

```java
/**
 * 判断是否有指定的权限
 */
boolean isPermitted(PrincipalCollection principals, String permission);

/**
 * 判断是否有指定的权限
 */
boolean isPermitted(PrincipalCollection subjectPrincipal, Permission permission);

/**
 * 判断是否有指定的权限集合
 */
boolean[] isPermitted(PrincipalCollection subjectPrincipal, String... permissions);

/**
 * 判断是否有指定的所有权限集合
 */
boolean isPermittedAll(PrincipalCollection subjectPrincipal, String... permissions);

/**
 * 判断是否有指定的所有权限集合
 */
boolean isPermittedAll(PrincipalCollection subjectPrincipal, Collection<Permission> permissions);

/**
 * 检测是否存在权限，否则抛异常
 */
void checkPermission(PrincipalCollection subjectPrincipal, String permission) throws AuthorizationException;

/**
 * 检测是否存在权限，否则抛异常
 */
void checkPermission(PrincipalCollection subjectPrincipal, Permission permission) throws AuthorizationException;

/**
 * 检测是否存在权限，否则抛异常
 */
void checkPermissions(PrincipalCollection subjectPrincipal, String... permissions) throws AuthorizationException;

/**
 * 检测是否存在权限，否则抛异常
 */
void checkPermissions(PrincipalCollection subjectPrincipal, Collection<Permission> permissions) throws AuthorizationException;

/**
 * 判断是否有指定的角色
 */
boolean hasRole(PrincipalCollection subjectPrincipal, String roleIdentifier);

/**
 * 判断是否有指定的角色集合
 */
boolean[] hasRoles(PrincipalCollection subjectPrincipal, List<String> roleIdentifiers);

/**
 * 判断是否有指定的所有角色集合
 */
boolean hasAllRoles(PrincipalCollection subjectPrincipal, Collection<String> roleIdentifiers);

/**
 * 检测角色
 */
void checkRole(PrincipalCollection subjectPrincipal, String roleIdentifier) throws AuthorizationException;

/**
 * 检测角色
 */
void checkRoles(PrincipalCollection subjectPrincipal, Collection<String> roleIdentifiers) throws AuthorizationException;

/**
 * 检测角色
 */
void checkRoles(PrincipalCollection subjectPrincipal, String... roleIdentifiers) throws AuthorizationException;
```

Authorizer接口中主要提供了判断和检测角色权限的方法。AuthorizingRealm是Authorizer的实现子类，这个类同时实现了Authorizer、PermissionResolverAware、RolePermissionResolverAware接口，另外还继承了AuthenticatingRealm抽象类，关于Realm的分析可以参看上一篇内容。

下面我们正式开始分析AuthorizingRealm类，我们还是从属性和构造方法分析。请看下面代码。

```java
//是否使用缓存
private boolean authorizationCachingEnabled;
//缓存被认证过的信息
private Cache<Object, AuthorizationInfo> authorizationCache;
//授权缓存名称
private String authorizationCacheName;
//权限解析器
private PermissionResolver permissionResolver;
//角色解析器
private RolePermissionResolver permissionRoleResolver;

public AuthorizingRealm(CacheManager cacheManager, CredentialsMatcher matcher) {
	super();
	if (cacheManager != null) setCacheManager(cacheManager);
	if (matcher != null) setCredentialsMatcher(matcher);
    // 默认开启缓存
	this.authorizationCachingEnabled = true;
    //通配符权限解析器
	this.permissionResolver = new WildcardPermissionResolver();
    //设置缓存名称
	int instanceNumber = INSTANCE_COUNT.getAndIncrement();
	this.authorizationCacheName = getClass().getName() + DEFAULT_AUTHORIZATION_CACHE_SUFFIX;
	if (instanceNumber > 0) {
		this.authorizationCacheName = this.authorizationCacheName + "." + instanceNumber;
	}
}
```

因为AuthorizingRealm实现了初始化接口Initializable，所以在AuthorizingRealm创建时会先调用onInit()方法进行初始化操作。初始化主要获取缓存对象Cache。

```java
    private Cache<Object, AuthorizationInfo> getAuthorizationCacheLazy() {
        // 首先authorizationCache会为空
        if (this.authorizationCache == null) {

            if (log.isDebugEnabled()) {
                log.debug("No authorizationCache instance set.  Checking for a cacheManager...");
            }

            CacheManager cacheManager = getCacheManager();

            if (cacheManager != null) {
                // 根据缓存名称获取缓存对象
                String cacheName = getAuthorizationCacheName();
                this.authorizationCache = cacheManager.getCache(cacheName);
            } else {
                if (log.isDebugEnabled()) {
                    log.debug("No cache or cacheManager properties have been set.  Authorization cache cannot " +
                            "be obtained.");
                }
            }
        }

        return this.authorizationCache;
    }
```

在Authorizer接口中方法都很类似，我们以分别选出角色和权限的方法进行分析，其他方法一样。我们即将对下面两个方法进行分析。

```java
/**
 * 判断是否有指定的权限
 */
boolean isPermitted(PrincipalCollection principals, String permission);

/**
 * 判断是否有指定的角色
 */
boolean hasRole(PrincipalCollection subjectPrincipal, String roleIdentifier);
```

### isPermitted方法分析

下面先给出关于isPermitted方法的源代码，如下:

```java
public boolean isPermitted(PrincipalCollection principals, String permission) {
    //使用权限解析器解析获取权限对象
	Permission p = getPermissionResolver().resolvePermission(permission);
    //判断是否存在权限
	return isPermitted(principals, p);
}

public boolean isPermitted(PrincipalCollection principals, Permission permission) {
    // 通过凭证获取认证信息
	AuthorizationInfo info = getAuthorizationInfo(principals);
    // 判断认证信息是否存在给定的权限
	return isPermitted(permission, info);
}

protected boolean isPermitted(Permission permission, AuthorizationInfo info) {
    // 获取认证信息的所有权限集合
	Collection<Permission> perms = getPermissions(info);
    //遍历判断是否存在权限
	if (perms != null && !perms.isEmpty()) {
		for (Permission perm : perms) {
			if (perm.implies(permission)) {
				return true;
			}
		}
	}
	return false;
}
```

在上面的代码中，首先通过getPermissionResolver().resolvePermission(permission)将指定字符串的权限(可以理解为权限码或权限名称)封装成Permission权限对象，这里使用的是WildcardPermission实现类。接下来，调用getAuthorizationInfo(principals)方法通过凭证获取认证信息。最后就是调用Permission接口提供的implies(permission)方法来判断是否真实包含权限。

我们来分析下getAuthorizationInfo(principals)方法。

```java
protected AuthorizationInfo getAuthorizationInfo(PrincipalCollection principals) {

	if (principals == null) {
		return null;
	}

	AuthorizationInfo info = null;

	if (log.isTraceEnabled()) {
		log.trace("Retrieving AuthorizationInfo for principals [" + principals + "]");
	}
        
    // 获取缓存对象，如果没有开启缓存功能，则返回null
    // 这里会调用我们在前面分析过的getAuthorizationCacheLazy()方法
	Cache<Object, AuthorizationInfo> cache = getAvailableAuthorizationCache();
	if (cache != null) {
		if (log.isTraceEnabled()) {
			log.trace("Attempting to retrieve the AuthorizationInfo from cache.");
		}
        // 获取凭证使用授权时的缓存Key，这个方法直接返回principals本身
        // 在业务处理过程中可以对该方法进行重写
		Object key = getAuthorizationCacheKey(principals);
        //从缓存中获取AuthorizationInfo认证信息
		info = cache.get(key);
		if (log.isTraceEnabled()) {
			if (info == null) {
				log.trace("No AuthorizationInfo found in cache for principals [" + principals + "]");
			} else {
				log.trace("AuthorizationInfo found in cache for principals [" + principals + "]");
			}
		}
	}

        // 如果没有开启缓存或缓存中不存在，则需要重doGetAuthorizationInfo(principals)方法中获取
        // 这个方法是抽象方法由子类实现如何获取
	if (info == null) 
		info = doGetAuthorizationInfo(principals);
		if (info != null && cache != null) {
			if (log.isTraceEnabled()) {
				log.trace("Caching authorization info for principals: [" + principals + "].");
			}
           //如果获取成功，则存放到缓存中去
			Object key = getAuthorizationCacheKey(principals);
			cache.put(key, info);
		}
	}

	return info;
}

protected Object getAuthorizationCacheKey(PrincipalCollection principals) {
	return principals;
}
```

### hasRole方法分析

hasRole方法很简单，获取AuthorizationInfo对象后，直接判断是否包含给定的角色信息。

源代码如下：

```java
public boolean hasRole(PrincipalCollection principal, String roleIdentifier) {
	AuthorizationInfo info = getAuthorizationInfo(principal);
	return hasRole(roleIdentifier, info);
}

protected boolean hasRole(String roleIdentifier, AuthorizationInfo info) {
    // 判断AuthorizationInfo中是否存在角色信息
	return info != null && info.getRoles() != null && info.getRoles().contains(roleIdentifier);
}
```

从上面分析isPermitted和hasRole这两个方法时我们会想，角色和权限信息从哪里获取来并存放在AuthorizationInfo对象中去的，很显然是抽象方法doGetAuthorizationInfo(PrincipalCollection principals)。

在使用Shiro进行认证时，肯定需要根据业务定义自己的Realm数据域来源，而AuthorizingRealm实现授权器(Authorizer)的同时还继承了AuthenticatingRealm，也就是说如果我们需要进行授权操作，自定义的Realm需要从AuthorizingRealm继承，从而根据业务实现doGetAuthorizationInfo(PrincipalCollection principals)方法。

关于授权器(Authorizer)的分析基本上就这么多，下面我们随便把上面说过的Permission权限对象分析一下。在上面我们提及过使用了WildcardPermission实现类，这是一种通配符匹配的实现方式。

## WildcardPermission分析

我们回到上面的代码片段：

```java
Permission p = getPermissionResolver().resolvePermission(permission);
```

这句代码真实调用如下：

```java
public class WildcardPermissionResolver implements PermissionResolver {
    public Permission resolvePermission(String permissionString) {
        return new WildcardPermission(permissionString);
    }
}
```

在上面创建了WildcardPermission实例，由WildcardPermission对象来对permissionString做通配符匹配处理。在WildcardPermission初始化的过程中对permissionString进行了解析。

```java
public WildcardPermission(String wildcardString) {
    //默认请求下对字符串大小写是不敏感的 
	this(wildcardString, DEFAULT_CASE_SENSITIVE);
}

public WildcardPermission(String wildcardString, boolean caseSensitive) {
	setParts(wildcardString, caseSensitive);
}

protected void setParts(String wildcardString, boolean caseSensitive) {
    // 清除字符串前后空格
	wildcardString = StringUtils.clean(wildcardString);

	if (wildcardString == null || wildcardString.isEmpty()) {
		throw new IllegalArgumentException("Wildcard string cannot be null or empty. Make sure permission strings are properly formatted.");
	}
    // 如果大小写不敏感，全部转为小写处理
	if (!caseSensitive) {
		wildcardString = wildcardString.toLowerCase();
	}
    // 首先使用冒号(:)进行一次分隔
	List<String> parts = CollectionUtils.asList(wildcardString.split(PART_DIVIDER_TOKEN));

	this.parts = new ArrayList<Set<String>>();
	for (String part : parts) {
                // 然后再使用逗号(,)进行一次分隔
		Set<String> subparts = CollectionUtils.asSet(part.split(SUBPART_DIVIDER_TOKEN));

		if (subparts.isEmpty()) {
			throw new IllegalArgumentException("Wildcard string cannot contain parts with only dividers. Make sure permission strings are properly formatted.");
		}
        // 将分割后的所有权限信息存储到this.parts中
		this.parts.add(subparts);
	}

	if (this.parts.isEmpty()) {
		throw new IllegalArgumentException("Wildcard string cannot contain only dividers. Make sure permission strings are properly formatted.");
	}
}
```

接下来就开始判断认证信息AuthorizationInfo中的权限集合是否可以和WildcardPermission进行匹配了。

```java
    public boolean implies(Permission p) {
        //判断类型是否相同
        if (!(p instanceof WildcardPermission)) {
            return false;
        }

        WildcardPermission wp = (WildcardPermission) p;
        // 获取parts属性
        List<Set<String>> otherParts = wp.getParts();

        // otherParts会和getParts()逐一匹配，如果getParts()在匹配过程中比otherParts数量少，就暗指省略可以匹配，返回true
        // 否则的话需要一一进行比较，在比较的过程中如果part不包含通配符(*)，且part不能完全包含otherPart集合，就认为没有权限，返回false。
        int i = 0;
        for (Set<String> otherPart : otherParts) {
            if (getParts().size() - 1 < i) {
                return true;
            } else {
                Set<String> part = getParts().get(i);
                if (!part.contains(WILDCARD_TOKEN) && !part.containsAll(otherPart)) {
                    return false;
                }
                i++;
            }
        }

        // 处理getParts()比otherParts长的的情况，如果这样，后面部分必须都是通配符(*)，否则返回false
        for (; i < getParts().size(); i++) {
            Set<String> part = getParts().get(i);
            if (!part.contains(WILDCARD_TOKEN)) {
                return false;
            }
        }
        return true;
    }
```