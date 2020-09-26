spring web初始化过程：

```java
@HandlesTypes({WebApplicationInitializer.class})
public class SpringServletContainerInitializer implements ServletContainerInitializer {
}

//创建根容器
createRootApplicationContext()
    //org.springframework.web.context.support.AnnotationConfigWebApplicationContext@275aecb8
    createRootApplicationContext();
	//根容器创建好了之后，会产生一个listener
	ContextLoaderListener listener = new ContextLoaderListener(rootAppContext);
	servletContext.addListener(listener);
//注册dispatcherServlet
registerDispatcherServlet(servletContext);
	//创建一个根容器
	WebApplicationContext servletAppContext = createServletApplicationContext();
	//添加servlet
	ServletRegistration.Dynamic registration = servletContext.addServlet(servletName, dispatcherServlet);
	//添加映射
	registration.addMapping(getServletMappings());
设置为true
registration.setAsyncSupported(isAsyncSupported());
    
```

## mvc过程：

```java
得到handler
得到HandlerExecutionChain
得到HandlerAdapter
    执行handle方法得到modelandview
doDispatch(HttpServletRequest request, HttpServletResponse response)
    // Determine handler for the current request.
    mappedHandler = getHandler(processedRequest);
		for (HandlerMapping hm : this.handlerMappings)
            HandlerExecutionChain handler = hm.getHandler(request);
				//得到handler对象
				Object handler = getHandlerInternal(request);
					//在获取的时候加锁(RequestMappingHandlerMapping) 实现方法加锁
					this.mappingRegistry.acquireReadLock();
					this.mappingRegistry.releaseReadLock();
				//得到执行链
				HandlerExecutionChain executionChain = getHandlerExecutionChain(handler, request);
					//先强转为HandlerExecutionChain对象
					for (HandlerInterceptor interceptor : this.adaptedInterceptors)
                        //判断url匹配之后
                      	chain.addInterceptor(mappedInterceptor.getInterceptor());
	// Determine handler adapter for the current request.
	HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());
		for (HandlerAdapter ha : this.handlerAdapters) 
            if (ha.supports(handler)) {
				return ha;
			}e
	//执行preHandle
	mappedHandler.applyPreHandle(processedRequest, response)
        HandlerInterceptor[] interceptors = getInterceptors();
			interceptor.preHandle(request, response, this.handler)
    // Actually invoke the handler. 执行controller里边的方法
	mv = ha.handle(processedRequest, response, mappedHandler.getHandler());
		// No synchronization on session demanded at all...
		mav = invokeHandlerMethod(request, response, handlerMethod);
	//执行posthandle方法
	mappedHandler.applyPostHandle(processedRequest, response, mv);
	processDispatchResult(processedRequest, response, mappedHandler, mv, dispatchException);
		render(mv, request, response);
             // Determine locale for request and apply it to the response.
			Locale locale = this.localeResolver.resolveLocale(request);
			response.setLocale(locale);
			view = resolveViewName(mv.getViewName(), mv.getModelInternal(), locale, request);
				for (ViewResolver viewResolver : this.viewResolvers) {
					View view = viewResolver.resolveViewName(viewName, locale);
                            if (!isCache()) {
                                return createView(viewName, locale);
                            }
                    		//双重校验
                    		if (view == null) {
            				synchronized (this.viewCreationCache)
                             view = createView(viewName, locale);
                    			if (viewName.startsWith(REDIRECT_URL_PREFIX))
                                 //根据配置的前后缀来创建视图
                                 super.createView(viewName, locale);
                             this.viewCreationCache.put(cacheKey, view);
      view.render(mv.getModelInternal(), request, response);
      mappedHandler.triggerAfterCompletion(request, response, null);
                    
handlermapping 
    实现
    
    	/**
		 * Acquire the read lock when using getMappings and getMappingsByUrl.
		 */
		public void acquireReadLock() {
			this.readWriteLock.readLock().lock();
		}

org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping
    org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping
    org.springframework.web.servlet.config.annotation.WebMvcConfigurationSupport$EmptyHandlerMapping
    org.springframework.web.servlet.config.annotation.WebMvcConfigurationSupport$EmptyHandlerMapping
    org.springframework.web.servlet.handler.SimpleUrlHandlerMapping
    得到handlerMapping 会使用读写锁
    
    org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter
    org.springframework.web.servlet.mvc.HttpRequestHandlerAdapter
    org.springframework.web.servlet.mvc.SimpleControllerHandlerAdapter
    
    mappedHandler.applyPreHandle(processedRequest, response)
    
    mv = ha.handle(processedRequest, response, mappedHandler.getHandler());
org.springframework.web.servlet.resource.ResourceUrlProviderExposingInterceptor
    org.springframework.web.servlet.handler.ConversionServiceExposingInterceptor
    com.project.interceptor.MyFirstInterceptor
    
    mappedHandler.applyPostHandle(processedRequest, response, mv);
    
render(mv, request, response);
    Locale locale = this.localeResolver.resolveLocale(request);
		response.setLocale(locale);
org.springframework.web.servlet.view.ViewResolverComposite
    synchronized
    
    RequestDispatcher
    interceptor.afterCompletion
    
    
    
    mappedHandler.triggerAfterCompletion(request, response, null);
publishRequestHandledEvent
```

## aop

 1）spring 容器启动，每个bean的实例化之前都会先经过AbstractAutoProxyCreator类的postProcessAfterInitialization（）这个方法，然后接下来是调用wrapIfNecessary方法。 

```java
/**
 * Create a proxy with the configured interceptors if the bean is
 * identified as one to proxy by the subclass.
 * @see #getAdvicesAndAdvisorsForBean
 */
public Object <strong>postProcessAfterInitialization</strong>(Object bean, String beanName) throws BeansException {
	if (bean != null) {
		Object cacheKey = getCacheKey(bean.getClass(), beanName);
		if (!this.earlyProxyReferences.containsKey(cacheKey)) {
			return wrapIfNecessary(bean, beanName, cacheKey);
		}
	}
	return bean;
}
```
 2）进入wrapIfNecessary方法后，我们直接看重点实现逻辑的方法getAdvicesAndAdvisorsForBean，这个方法会提取当前bean 的所有增强方法，然后获取到适合的当前bean 的增强方法，然后对增强方法进行排序，最后返回。 

```java
/**
	 * Wrap the given bean if necessary, i.e. if it is eligible for being proxied.
	 * @param bean the raw bean instance
	 * @param beanName the name of the bean
	 * @param cacheKey the cache key for metadata access
	 * @return a proxy wrapping the bean, or the raw bean instance as-is
	 */
	protected Object wrapIfNecessary(Object bean, String beanName, Object cacheKey) {
		if (beanName != null && this.targetSourcedBeans.containsKey(beanName)) {
			return bean;
		}
		if (Boolean.FALSE.equals(this.advisedBeans.get(cacheKey))) {
			return bean;
		}
		if (isInfrastructureClass(bean.getClass()) || shouldSkip(bean.getClass(), beanName)) {
			this.advisedBeans.put(cacheKey, Boolean.FALSE);
			return bean;
		}
 
		// Create proxy if we have advice.	
		Object[] specificInterceptors = getAdvicesAndAdvisorsForBean(bean.getClass(), beanName, null);
		if (specificInterceptors != DO_NOT_PROXY) {
			this.advisedBeans.put(cacheKey, Boolean.TRUE);
			Object proxy = createProxy(bean.getClass(), beanName, specificInterceptors, new SingletonTargetSource(bean));
			this.proxyTypes.put(cacheKey, proxy.getClass());
			return proxy;
		}
 
		this.advisedBeans.put(cacheKey, Boolean.FALSE);
		return bean;
	}
```
3）获取到当前bean的增强方法后，便调用createProxy方法，创建代理。先创建代理工厂proxyFactory，然后获取当前bean 的增强器advisors，把当前获取到的增强器添加到代理工厂proxyFactory，然后设置当前的代理工的代理目标对象为当前bean，最后根据配置创建JDK的动态代理工厂，或者CGLIB的动态代理工厂，然后返回proxyFactory

```java
/**
	 * Create an AOP proxy for the given bean.
	 * @param beanClass the class of the bean
	 * @param beanName the name of the bean
	 * @param specificInterceptors the set of interceptors that is
	 * specific to this bean (may be empty, but not null)
	 * @param targetSource the TargetSource for the proxy,
	 * already pre-configured to access the bean
	 * @return the AOP proxy for the bean
	 * @see #buildAdvisors
	 */
	protected Object createProxy(
			Class<?> beanClass, String beanName, Object[] specificInterceptors, TargetSource targetSource) {
 
        //通过代理工厂来创建代理
		ProxyFactory proxyFactory = new ProxyFactory();
		// Copy our properties (proxyTargetClass etc) inherited from ProxyConfig.
		proxyFactory.copyFrom(this);
 
		if (!shouldProxyTargetClass(beanClass, beanName)) {
			// Must allow for introductions; can't just set interfaces to
			// the target's interfaces only.
			Class<?>[] targetInterfaces = ClassUtils.getAllInterfacesForClass(beanClass, this.proxyClassLoader);
			for (Class<?> targetInterface : targetInterfaces) {
				proxyFactory.addInterface(targetInterface);
			}
		}
 
		Advisor[] advisors = buildAdvisors(beanName, specificInterceptors);
		for (Advisor advisor : advisors) {
			proxyFactory.addAdvisor(advisor);
		}
 
		proxyFactory.<strong>setTargetSource</strong>(targetSource);
		customizeProxyFactory(proxyFactory);
 
		proxyFactory.setFrozen(this.freezeProxy);
		if (advisorsPreFiltered()) {
			proxyFactory.setPreFiltered(true);
		}
 
		return proxyFactory.getProxy(this.proxyClassLoader);
	}
```
 AOP动态代理的实现选择方式 

```java
public AopProxy createAopProxy(AdvisedSupport config) throws AopConfigException {
	if (config.isOptimize() || config.isProxyTargetClass() || hasNoUserSuppliedProxyInterfaces(config)) {
		Class targetClass = config.getTargetClass();
		if (targetClass == null) {
			throw new AopConfigException("TargetSource cannot determine target class: " +
					"Either an interface or a target is required for proxy creation.");
		}
		if (targetClass.isInterface()) {
			return new JdkDynamicAopProxy(config);
		}
		return CglibProxyFactory.createCglibProxy(config);
	}
	else {
		return new JdkDynamicAopProxy(config);
	}
}
```
jdk动态代理

```java
public Object invoke(Object proxy, Method method, Object[] args) throwsThrowable {
       MethodInvocation invocation = null;
       Object oldProxy = null;
       boolean setProxyContext = false;
 
       TargetSource targetSource = this.advised.targetSource;
       Class targetClass = null;
       Object target = null;
 
       try {
           //eqauls()方法，具目标对象未实现此方法
           if (!this.equalsDefined && AopUtils.isEqualsMethod(method)){
                return (equals(args[0])? Boolean.TRUE : Boolean.FALSE);
           }
 
           //hashCode()方法，具目标对象未实现此方法
           if (!this.hashCodeDefined && AopUtils.isHashCodeMethod(method)){
                return newInteger(hashCode());
           }
 
           //Advised接口或者其父接口中定义的方法,直接反射调用,不应用通知
           if (!this.advised.opaque &&method.getDeclaringClass().isInterface()
                    &&method.getDeclaringClass().isAssignableFrom(Advised.class)) {
                // Service invocations onProxyConfig with the proxy config...
                return AopUtils.invokeJoinpointUsingReflection(this.advised,method, args);
           }
 
           Object retVal = null;
 
           if (this.advised.exposeProxy) {
                // Make invocation available ifnecessary.
                oldProxy = AopContext.setCurrentProxy(proxy);
                setProxyContext = true;
           }
 
           //获得目标对象的类
           target = targetSource.getTarget();
           if (target != null) {
                targetClass = target.getClass();
           }
 
           //获取可以应用到此方法上的Interceptor列表
           List chain = this.advised.getInterceptorsAndDynamicInterceptionAdvice(method,targetClass);
 
           //如果没有可以应用到此方法的通知(Interceptor)，此直接反射调用 method.invoke(target, args)
           if (chain.isEmpty()) {
                retVal = AopUtils.invokeJoinpointUsingReflection(target,method, args);
           } else {
                //创建MethodInvocation
                invocation = newReflectiveMethodInvocation(proxy, target, method, args, targetClass, chain);
                retVal = invocation.proceed();
           }
 
           // Massage return value if necessary.
           if (retVal != null && retVal == target &&method.getReturnType().isInstance(proxy)
                    &&!RawTargetAccess.class.isAssignableFrom(method.getDeclaringClass())) {
                // Special case: it returned"this" and the return type of the method
                // is type-compatible. Notethat we can't help if the target sets
                // a reference to itself inanother returned object.
                retVal = proxy;
           }
           return retVal;
       } finally {
           if (target != null && !targetSource.isStatic()) {
                // Must have come fromTargetSource.
               targetSource.releaseTarget(target);
           }
           if (setProxyContext) {
                // Restore old proxy.
                AopContext.setCurrentProxy(oldProxy);
           }
       }
    }
```

1）获取拦截器
2）判断拦截器链是否为空，如果是空的话直接调用切点方法
3）如果拦截器不为空的话那么便创建ReflectiveMethodInvocation类，把拦截器方法都封装在里面，也就是执行getInterceptorsAndDynamicInterceptionAdvice方法

```java
public List<Object> getInterceptorsAndDynamicInterceptionAdvice(Method method, Class targetClass) {
                   MethodCacheKeycacheKey = new MethodCacheKey(method);
                   List<Object>cached = this.methodCache.get(cacheKey);
                   if(cached == null) {
                            cached= this.advisorChainFactory.getInterceptorsAndDynamicInterceptionAdvice(
                                               this,method, targetClass);
                            this.methodCache.put(cacheKey,cached);
                   }
                   returncached;
         }
```

 4)其实实际的获取工作其实是由AdvisorChainFactory. getInterceptorsAndDynamicInterceptionAdvice()这个方法来完成的，获取到的结果会被缓存，下面来分析下这个方法的实现： 

```java
/**
    * 从提供的配置实例config中获取advisor列表,遍历处理这些advisor.如果是IntroductionAdvisor,
    * 则判断此Advisor能否应用到目标类targetClass上.如果是PointcutAdvisor,则判断
    * 此Advisor能否应用到目标方法method上.将满足条件的Advisor通过AdvisorAdaptor转化成Interceptor列表返回.
    */
    publicList getInterceptorsAndDynamicInterceptionAdvice(Advised config, Methodmethod, Class targetClass) {
       // This is somewhat tricky... we have to process introductions first,
       // but we need to preserve order in the ultimate list.
       List interceptorList = new ArrayList(config.getAdvisors().length);
 
       //查看是否包含IntroductionAdvisor
       boolean hasIntroductions = hasMatchingIntroductions(config,targetClass);
 
       //这里实际上注册一系列AdvisorAdapter,用于将Advisor转化成MethodInterceptor
       AdvisorAdapterRegistry registry = GlobalAdvisorAdapterRegistry.getInstance();
 
       Advisor[] advisors = config.getAdvisors();
        for (int i = 0; i <advisors.length; i++) {
           Advisor advisor = advisors[i];
           if (advisor instanceof PointcutAdvisor) {
                // Add it conditionally.
                PointcutAdvisor pointcutAdvisor= (PointcutAdvisor) advisor;
                if(config.isPreFiltered() ||pointcutAdvisor.getPointcut().getClassFilter().matches(targetClass)) {
                    //TODO: 这个地方这两个方法的位置可以互换下
                    //将Advisor转化成Interceptor
                    MethodInterceptor[]interceptors = registry.getInterceptors(advisor);
 
                    //检查当前advisor的pointcut是否可以匹配当前方法
                    MethodMatcher mm =pointcutAdvisor.getPointcut().getMethodMatcher();
 
                    if (MethodMatchers.matches(mm,method, targetClass, hasIntroductions)) {
                        if(mm.isRuntime()) {
                            // Creating a newobject instance in the getInterceptors() method
                            // isn't a problemas we normally cache created chains.
                            for (intj = 0; j < interceptors.length; j++) {
                               interceptorList.add(new InterceptorAndDynamicMethodMatcher(interceptors[j],mm));
                            }
                        } else {
                            interceptorList.addAll(Arrays.asList(interceptors));
                        }
                    }
                }
           } else if (advisor instanceof IntroductionAdvisor){
                IntroductionAdvisor ia =(IntroductionAdvisor) advisor;
                if(config.isPreFiltered() || ia.getClassFilter().matches(targetClass)) {
                    Interceptor[] interceptors= registry.getInterceptors(advisor);
                    interceptorList.addAll(Arrays.asList(interceptors));
                }
           } else {
                Interceptor[] interceptors =registry.getInterceptors(advisor);
                interceptorList.addAll(Arrays.asList(interceptors));
           }
       }
       return interceptorList;
}
```

 这个方法执行完成后，Advised中配置能够应用到连接点或者目标类的Advisor全部被转化成了MethodInterceptor.
6）接下来货到invoke方法中的proceed方法 ，我们再看下得到的拦截器链是怎么起作用的，也就是proceed方法的执行过程 



| 缓存                  | 用途                                                         |
| :-------------------- | :----------------------------------------------------------- |
| singletonObjects      | 用于存放完全初始化好的 bean，从该缓存中取出的 bean 可以直接使用 |
| earlySingletonObjects | 用于存放还在初始化中的 bean，用于解决循环依赖                |
| singletonFactories    | 用于存放 bean 工厂。bean 工厂所产生的 bean 是还未完成初始化的 bean。如代码所示，bean 工厂所生成的对象最终会被缓存到 earlySingletonObjects 中 |

