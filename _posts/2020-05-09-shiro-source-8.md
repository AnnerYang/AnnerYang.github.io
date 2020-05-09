---
layout: post
title: Shiro源码分析(8) - 过滤器(ShiroFilterFactoryBean)
categories: Shiro
description: Shiro源码分析(8) - 过滤器(ShiroFilterFactoryBean)
keywords: Shiro, ShiroFilterFactoryBean
---

摘要:原创出处 https://my.oschina.net/xiaoqiyiye/blog/1618279 「xiaoqiyiye」欢迎转载，保留摘要，谢谢！

### 配置说明

ShiroFilterFactoryBean是Spring为Shiro框架提供的一个整合类，在项目使用过程中通常会使用如下的配置方式来处理。

```java
<bean id="shiroFilter" class="org.apache.shiro.spring.web.ShiroFilterFactoryBean">  
    <!-- 安全管理器-->
    <property name="securityManager" ref="securityManager"/>  
    <!-- 登录地址-->
    <property name="loginUrl" value="/login.jsp"/>  
    <!-- 未授权跳转地址-->
    <property name="unauthorizedUrl" value="/unauthorized.jsp"/>  
    <!--配置自定义的过滤器-->
    <property name="filters">  
        <util:map>  
            <entry key="authc" value-ref="formAuthenticationFilter"/>  
        </util:map>  
    </property>  
    <!--配置请求路径过滤规则-->
    <property name="filterChainDefinitions">  
        <value>  
            /index.jsp = anon  
            /login.jsp = authc  
            /logout = logout  
            /** = user  
        </value>  
    </property>  
</bean>   
```

在上面的配置中需要解释一下filterChainDefinitions这个属性的配置。例如/logout = logout表示请求地址为/logout的请求需要被logout过滤。下面重点是在分析对filterChainDefinitions配置解析处理。

## ShiroFilter

ShiroFilterFactoryBean作为一个工厂Bean，将ShiroFilter实例对象注入到Spring容器中去，这样Shiro基于url的方式进行请求过滤处理。

在ShiroFilterFactoryBean中主要需要分析的方法是createInstance()，该方法创建了ShiroFilter对象实例，将SpringShiroFilter注入到容器中去。

```java
protected AbstractShiroFilter createInstance() throws Exception {

	log.debug("Creating Shiro Filter instance.");
        //  安全管理器不能为空
	SecurityManager securityManager = getSecurityManager();
	if (securityManager == null) {
		String msg = "SecurityManager property must be set.";
		throw new BeanInitializationException(msg);
	}
        // WEB环境中，必须是WebSecurityManager类型
	if (!(securityManager instanceof WebSecurityManager)) {
		String msg = "The security manager does not implement the WebSecurityManager interface.";
		throw new BeanInitializationException(msg);
	}
    
        //管理着Shiro提供的默认过滤器信息
	FilterChainManager manager = createFilterChainManager();

	//获取路径匹配过滤器链的处理对象
	PathMatchingFilterChainResolver chainResolver = new PathMatchingFilterChainResolver();
	chainResolver.setFilterChainManager(manager);

	// 创建SpringShiroFilter对象
	return new SpringShiroFilter((WebSecurityManager) securityManager, chainResolver);
}
```

我们分三个步骤来分析上面的方法，分别是：

- 过滤器管理器(FilterChainManager)是如何管理过滤器
- PathMatchingFilterChainResolver 在SpringShiroFilter中的作用
- SpringShiroFilter中过滤器实现

### 过滤器链管理器(FilterChainManager)

FilterChainManager有一个实现类DefaultFilterChainManager负责管理存在的Filter信息。在DefaultFilterChainManager实例化的时候会加载Shiro提供的一些默认的Filter实例对象。

```
public DefaultFilterChainManager() {
	this.filters = new LinkedHashMap<String, Filter>();
	this.filterChains = new LinkedHashMap<String, NamedFilterList>();
        // 添加默认的过滤器，false参数表示是否需要初始化FilterConfig信息
	addDefaultFilters(false);
}

public DefaultFilterChainManager(FilterConfig filterConfig) {
	this.filters = new LinkedHashMap<String, Filter>();
	this.filterChains = new LinkedHashMap<String, NamedFilterList>();
	setFilterConfig(filterConfig);
        // 添加默认的过滤器，需要初始化FilterConfig信息
	addDefaultFilters(true);
}

protected void addDefaultFilters(boolean init) {
        // 默认的过滤器信息存放在DefaultFilter这个枚举类中
	for (DefaultFilter defaultFilter : DefaultFilter.values()) {
                // 将默认的过滤器实例添加到filters属性中去
		addFilter(defaultFilter.name(), defaultFilter.newInstance(), init, false);
	}
}
```

在Shiro中为我们提供了如下默认的过滤器，配置在DefaultFilter枚举类中。

```java
// 通用过滤器，任何请求允许访问
anon(AnonymousFilter.class),
// 表单认证过滤器
authc(FormAuthenticationFilter.class),
// 基于Http请求的认证过滤器
authcBasic(BasicHttpAuthenticationFilter.class),
// 登出过滤器
logout(LogoutFilter.class),
// 不创建Session过滤器
noSessionCreation(NoSessionCreationFilter.class),
// 权限认证过滤器
perms(PermissionsAuthorizationFilter.class),
// 端口过滤器
port(PortFilter.class),
// 请求处理为权限的一种过滤器
rest(HttpMethodPermissionFilter.class),
// 角色过滤器
roles(RolesAuthorizationFilter.class),
// SSL过滤器
ssl(SslFilter.class),
// 用户过滤器，检测用户是否登录
user(UserFilter.class);
```

现在我们还是回到createFilterChainManager()方法，这个方法创建了FilterChainManager对象，也就是上面分析的DefaultFilterChainManager构造方法。在创建了DefaultFilterChainManager后，需要对配置的过滤器和请求URL进行处理。下面我们具体分析过滤器是如何被管理的。

```java
protected FilterChainManager createFilterChainManager() {
        // 创建实例对象（上面提到的默认过滤器已经添加进来了）
	DefaultFilterChainManager manager = new DefaultFilterChainManager();
        //获取默认的过滤器
	Map<String, Filter> defaultFilters = manager.getFilters();
	// 为默认的过滤器设置登录URL、登录成功跳转URL和未授权跳转URL这些熟悉信息
	for (Filter filter : defaultFilters.values()) {
		applyGlobalPropertiesIfNecessary(filter);
	}

	//获取配置指定的过滤器(在xml配置示例中配置的filters属性)
	Map<String, Filter> filters = getFilters();
	if (!CollectionUtils.isEmpty(filters)) {
		for (Map.Entry<String, Filter> entry : filters.entrySet()) {
			String name = entry.getKey();
			Filter filter = entry.getValue();
			applyGlobalPropertiesIfNecessary(filter);
			if (filter instanceof Nameable) {
				((Nameable) filter).setName(name);
			}
                        // 将配置的过滤器也添加到过滤器链管理器中去
			manager.addFilter(name, filter, false);
		}
	}

	//获取配置的filterChainDefinitions属性信息：
        //          /index.jsp = anon  
        //          /login.jsp = authc  
        //          /logout = logout  
        //          /** = user  
	Map<String, String> chains = getFilterChainDefinitionMap();
	if (!CollectionUtils.isEmpty(chains)) {
		for (Map.Entry<String, String> entry : chains.entrySet()) {
			String url = entry.getKey();
			String chainDefinition = entry.getValue();
                        // url表示请求的地址
                        // chainDefinition表示该地址需要使用的过滤器链信息
			manager.createChain(url, chainDefinition);
		}
	}

	return manager;
}
```

为了搞明白配置的filterChainDefinitions属性是如何使用的，我们继续分析刚才的manager.createChain(url, chainDefinition)调用。这个方法将配置的值解析成键值对的过滤器链存放在filterChains属性中去。具体方法如下：

```java
public void createChain(String chainName, String chainDefinition) {
        // 检测配置key-value必须存在
	if (!StringUtils.hasText(chainName)) {
		throw new NullPointerException("chainName cannot be null or empty.");
	}
	if (!StringUtils.hasText(chainDefinition)) {
		throw new NullPointerException("chainDefinition cannot be null or empty.");
	}

	if (log.isDebugEnabled()) {
		log.debug("Creating chain [" + chainName + "] from String definition [" + chainDefinition + "]");
	}

        //这里需要解析一些过滤器的配置
        // 因为配置的值可能是 "authc, roles[admin,user], perms[file:edit]"
        // 解析后得到的对象值为{ "authc", "roles[admin,user]", "perms[file:edit]" }
	String[] filterTokens = splitChainDefinition(chainDefinition);

        // 继续解析过滤器，例如:  foo[bar, baz]
        // 解析后得到：{ "foo", "bar, baz" }
	for (String token : filterTokens) {
		String[] nameConfigPair = toNameConfigPair(token);
                // 将得到的过滤器信息添加到过滤器链中
		addToChain(chainName, nameConfigPair[0], nameConfigPair[1]);
	}
}

public void addToChain(String chainName, String filterName, String chainSpecificFilterConfig) {
	if (!StringUtils.hasText(chainName)) {
		throw new IllegalArgumentException("chainName cannot be null or empty.");
	}

        // 从这里根据名称获取过滤器实例(包括默认的和配置的Filter)
        // 过滤器的名称一定要配置正确，否则找不到就抛异常
	Filter filter = getFilter(filterName);
	if (filter == null) {
		throw new IllegalArgumentException("There is no filter with name '" + filterName +
				"' to apply to chain [" + chainName + "] in the pool of available Filters.  Ensure a " +
				"filter with that name/path has first been registered with the addFilter method(s).");
	}
        
        // 为过滤器设置一些特定的配置信息，由PathConfigProcessor接口来完成这个功能
	applyChainConfig(chainName, filter, chainSpecificFilterConfig);
        // 添加到过滤器链中
        // 这里处理的原理为：每个chainName(也就是配置中的请求URL)都关联一个NamedFilterList chain实例对象，并存放在filterChains Map中管理。
	NamedFilterList chain = ensureChain(chainName);
	chain.add(filter);
}
```

在DefaultFilterChainManager类中还有一个方法需要说明，那就是FilterChain proxy(FilterChain original, String chainName)方法。当进行请求过滤时，这个方法会代理处理过滤器链。

```java
public FilterChain proxy(FilterChain original, String chainName) {
        // 根据chainName获取到相应的NamedFilterList对象
        // 这个就是上面分析中看到的，将过滤器信息存放在NamedFilterList对象中
	NamedFilterList configured = getChain(chainName);
        // 没有匹配到就抛异常
	if (configured == null) {
		String msg = "There is no configured chain under the name/key [" + chainName + "].";
		throw new IllegalArgumentException(msg);
	}
        // 对原来的FilterChain做代理
	return configured.proxy(original);
}
```

代理处理在ProxiedFilterChain#doFilter(ServletRequest request, ServletResponse response) 方法中。

```java
public void doFilter(ServletRequest request, ServletResponse response) throws IOException, ServletException {
        // 如果filters为空，或代理已经处理完，则处理原先的FilterChain
	if (this.filters == null || this.filters.size() == this.index) {
		if (log.isTraceEnabled()) {
			log.trace("Invoking original filter chain.");
		}
		this.orig.doFilter(request, response);
	} else {
		if (log.isTraceEnabled()) {
			log.trace("Invoking wrapped filter at index [" + this.index + "]");
		}
                // 逐一调用filters中的过滤器
                // 注意，这个传入的FilterChain为ProxiedFilterChain本身。
		this.filters.get(this.index++).doFilter(request, response, this);
	}
}
```

------

### SpringShiroFilter分析

在Shiro框架中提供了对javax.servlet.Filter接口的实现，其实现的抽象类层次为OncePerRequestFilter、AdviceFilter、PathMatchingFilter、AccessControlFilter、AuthenticationFilter、AuthenticatingFilter。对于Shiro中的Filter实现，我们将逐一进行分析。

#### OncePerRequestFilter

在OncePerRequestFilter类中处理了两个功能：第一，标记那些过滤器执行过，确保不重复执行；第二，使用enable来表示是否需要执行当前的过滤器，enable默认为true，isEnabled(request, response) 方法作为过滤器是否执行的判断依据。

```java
public final void doFilter(ServletRequest request, ServletResponse response, FilterChain filterChain)
		throws ServletException, IOException {
        // 标记过滤器是否已经被执行过，确保不重复执行
	String alreadyFilteredAttributeName = getAlreadyFilteredAttributeName();
	if ( request.getAttribute(alreadyFilteredAttributeName) != null ) {
		log.trace("Filter '{}' already executed.  Proceeding without invoking this filter.", getName());
		filterChain.doFilter(request, response);
	} else 
        // enable表示是否开启当前执行器，默认enable=true
        if (!isEnabled(request, response) ) {
		log.debug("Filter '{}' is not enabled for the current request.  Proceeding without invoking this filter.", getName());
		filterChain.doFilter(request, response);
	} else {
		// 执行当前过滤器
		log.trace("Filter '{}' not yet executed.  Executing now.", getName());
                // 标记已经执行
		request.setAttribute(alreadyFilteredAttributeName, Boolean.TRUE);
		try {
                        // 抽象方法，子类实现过滤器执行
			doFilterInternal(request, response, filterChain);
		} finally {
                        // 
			request.removeAttribute(alreadyFilteredAttributeName);
		}
	}
}
```

#### AdviceFilter

AdviceFilter过滤器对OncePerRequestFilter进行了进一步的抽象，对过滤器的执行添加了前置处理、后置处理和完成后的处理。分别提供下面方法进行：

```java
// 前置处理方法，返回true表示过滤器可以继续执行下去
boolean preHandle(ServletRequest request, ServletResponse response)

// 后置处理方法
void postHandle(ServletRequest request, ServletResponse response)    

// 完成后的处理方法，这里对异常情况做处理
void afterCompletion(ServletRequest request, ServletResponse response, Exception exception)
```

下面完整的分析下从OncePerRequestFilter实现的doFilterInternal(ServletRequest request, ServletResponse response, FilterChain chain)方法。

```java
public void doFilterInternal(ServletRequest request, ServletResponse response, FilterChain chain)
		throws ServletException, IOException {

	Exception exception = null;

	try {
                // 过滤器前置处理方法(钩子方法)，如果返回true表示继续执行过滤器链
		boolean continueChain = preHandle(request, response);
		if (log.isTraceEnabled()) {
			log.trace("Invoked preHandle method.  Continuing chain?: [" + continueChain + "]");
		}
                // 检测是否需要继续执行过滤器链
		if (continueChain) {
                        //执行过滤器链
			executeChain(request, response, chain);
		}
                // 过滤器后置处理方法(钩子方法)
		postHandle(request, response);
		if (log.isTraceEnabled()) {
			log.trace("Successfully invoked postHandle method");
		}

	} catch (Exception e) {
		exception = e;
	} finally {
                //在这里调用了afterCompletion(request, response, exception)方法
                //对异常做处理
		cleanup(request, response, exception);
	}
}
```

#### PathMatchingFilter

PathMatchingFilter在AdviceFilter的基础上对前置处理方法preHandle(ServletRequest request, ServletResponse response)进行了重写，PathMatchingFilter类对请求的路径做过滤器匹配，判断是否可以继续执行过滤器链。

在分析preHandle(ServletRequest request, ServletResponse response)方法之前，我们先看看PathConfigProcessor接口，PathMatchingFilter实现了PathConfigProcessor接口，它用来标记配置的过滤器路径信息。

我们在上面分析DefaultFilterChainManager#addToChain(String chainName, String filterName, String chainSpecificFilterConfig)方法的时候说过。在添加过滤器到DefaultFilterChainManager中时，如果过滤器实现了PathConfigProcessor接口，那么会调用PathConfigProcessor#processPathConfig(String path, String config)方法，将键值对添加到PathConfigProcessor的appliedPaths 属性中去。例如配置中存在：/foo/*.html = roles[admin,user]。那么添加到appliedPaths 中的就会是/foo/*.html = admin,user，这时的过滤器实例就是RolesAuthorizationFilter。

```java
public Filter processPathConfig(String path, String config) {
	String[] values = null;
	if (config != null) {
		values = split(config);
	}
        // 上面的例子中
        // path = /foo/*.html
        // values = [admin,user]
	this.appliedPaths.put(path, values);
	return this;
}
```

我们了解了appliedPaths属性是如何来的之后，我们开始分析preHandle(ServletRequest request, ServletResponse response)方法。

```java
protected boolean preHandle(ServletRequest request, ServletResponse response) throws Exception {
        // 过滤器没有特殊的配置，允许通过
	if (this.appliedPaths == null || this.appliedPaths.isEmpty()) {
		if (log.isTraceEnabled()) {
			log.trace("appliedPaths property is null or empty.  This Filter will passthrough immediately.");
		}
		return true;
	}

	for (String path : this.appliedPaths.keySet()) {
		// 判断请求路径是否和配置路径匹配
		if (pathsMatch(path, request)) {
			log.trace("Current requestURI matches pattern '{}'.  Determining filter chain execution...", path);
                        // 获取配置的具体信息
			Object config = this.appliedPaths.get(path);
                        // 进一步判断过滤器是否可以执行
                        // 在这里委派给子类去实现onPreHandle(request, response, pathConfig)方法来决定是否需要继续执行过滤器链
			return isFilterChainContinued(request, response, path, config);
		}
	}

	// 允许通过
	return true;
}
```

从上面的分析来看，在PathMatchingFilter类中对请求路径进行了匹配过滤，通过路径来判断当前的请求是否需要经过当前过滤器来处理。另外，提供了onPreHandle(request, response, pathConfig)这个钩子方法来判断是否执行过滤器。

#### AccessControlFilter

AccessControlFilter类是一个访问控制过滤器，在AccessControlFilter中提供了loginUrl属性来表示登录的URL地址。提供了两个抽象方法：

- isAccessAllowed(request, response, mappedValue)，表示方式是否被允许
- onAccessDenied(request, response, mappedValue)，当isAccessAllowed方法返回false时，使用这个方法来代替处理是否继续执行过滤器链。

下面是被重写的onPreHandle方法。

```java
    public boolean onPreHandle(ServletRequest request, ServletResponse response, Object mappedValue) throws Exception {
        return isAccessAllowed(request, response, mappedValue) || onAccessDenied(request, response, mappedValue);
    }
```