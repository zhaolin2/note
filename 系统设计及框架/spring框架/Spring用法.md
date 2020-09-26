

# Spring

## Spring导入

### 1.@Bean() 导入

```java
@Bean("beanId")
//自动注入ioc容器中，方法名为id
public Color color(){
   return new Color();
}
```

### 2.@Import({Class[])导入

​    可以导入多个组件，并且调用的是无参构造器，id为全路径名

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Import {

    //可以导入三种组件 分别是 ImportSelector，ImportBeanDefinitionRegistrar，classes
	/**
	 * {@link Configuration}, {@link ImportSelector}, {@link ImportBeanDefinitionRegistrar}
	 * or regular component classes to import.
	 */
	Class<?>[] value();

}
```



### 3.@ImportSelector.class导入

 根据元注解信息自定义返回需要导入的组件

```java
public interface ImportSelector {

    //返回要导入的全类名数组就行  并且不要返回null值 
	String[] selectImports(AnnotationMetadata importingClassMetadata);

}
```

```

```

### 4.@ImportBeanDefinitionRegister 
根据元注解和注册器 手动注册到容器中

```java
public interface ImportBeanDefinitionRegistrar {  
    //根据给定的注解元信息，根据需要来导入一些组件
    
    //一个是当前类的元注解信息，一个是BeanDefinition注册类
    public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry);}
```

```java
public class MyImportBeanDefinitionRegister implements ImportBeanDefinitionRegistrar {
    @Override
    public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {
        
        boolean hasColor = registry.containsBeanDefinition("org.project.beans.Color");
        boolean hasRed = registry.containsBeanDefinition("org.project.beans.Red");
        if (hasColor && hasRed){
            RootBeanDefinition rootBeanDefinition = new RootBeanDefinition(Blue.class);
            rootBeanDefinition.setScope(ConfigurableBeanFactory.SCOPE_SINGLETON);
            registry.registerBeanDefinition("blue",rootBeanDefinition);
        }
    }
}
```



### 5.FactoryBean
实现Factory来进行导入

```java
public interface FactoryBean<T> {
	String OBJECT_TYPE_ATTRIBUTE = "factoryBeanObjectType";
    
    //返回一个由该工厂创建的实例
	@Nullable
	T getObject() throws Exception;

    //返回该工厂创建的实例的class
	@Nullable
	Class<?> getObjectType();

	//是否是单例
	default boolean isSingleton() {
		return true;
	}

}
```



```java
	AnnotationConfigApplicationContext annotationConfigApplicationContext = new AnnotationConfigApplicationContext(MyImportConfig.class);
	//	console:	class org.project.importComponent.MyFactoryBean 
    //	得到的是工厂对象
    Object bean = annotationConfigApplicationContext.getBean("&myFactoryBean");
	//	console:	class org.project.beans.Color
    //	得到的是工厂产生的对象
    Object myFactoryBean = annotationConfigApplicationContext.getBean("myFactoryBean");
    System.out.println(myFactoryBean.getClass().toString());
```

```java
public interface BeanFactory {
	//在这定义
	String FACTORY_BEAN_PREFIX = "&";
	}
```


### 6.@CompontScan()

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Documented
@Repeatable(ComponentScans.class)
public @interface ComponentScan {

	@AliasFor("basePackages")
	String[] value() default {};

	@AliasFor("value")
	String[] basePackages() default {};
	//需要扫描的包中的类数组
	Class<?>[] basePackageClasses() default {};
    
    //在Spring中使用哪个类来堆检测到的组件进行命名
	Class<? extends BeanNameGenerator> nameGenerator() default BeanNameGenerator.class;
    
    //在指定域来进行解析
	Class<? extends ScopeMetadataResolver> scopeResolver() default AnnotationScopeMetadataResolver.class;

   	//是否为检测到的组件来生成代理，当以代理的方式使用作用域是必须的
	ScopedProxyMode scopedProxy() default ScopedProxyMode.DEFAULT;

	/**
	 * Controls the class files eligible for component detection.
	 * <p>Consider use of {@link #includeFilters} and {@link #excludeFilters}
	 * for a more flexible approach.
	 */
    //正则的匹配规则
	String resourcePattern() default ClassPathScanningCandidateComponentProvider.DEFAULT_RESOURCE_PATTERN;
    //默认匹配规则
    //static final String DEFAULT_RESOURCE_PATTERN = "**/*.class";
	
    //是否使用默认的过滤器
	boolean useDefaultFilters() default true;

	//包括哪些过滤去
	Filter[] includeFilters() default {};

	//排除哪些过滤器
	Filter[] excludeFilters() default {};

	//是否懒加载
	boolean lazyInit() default false;

	@Retention(RetentionPolicy.RUNTIME)
	@Target({})
	@interface Filter {

		//Filter的类型
		FilterType type() default FilterType.ANNOTATION;

		@AliasFor("classes")
		Class<?>[] value() default {};

		
        //被用于当前过滤器的类数组
		@AliasFor("value")
		Class<?>[] classes() default {};

        //根据Filter的type来进行不同的匹配
		String[] pattern() default {};

	}

}
```

Filter.type:

```java
public enum FilterType {

	ANNOTATION,
    //指定类型
	ASSIGNABLE_TYPE,

	ASPECTJ,

	REGEX,
	//定制
	CUSTOM
}
```

### 7.@conditional()
根据条件判断当前类或者方法的配置是否生效

```java
public class MyWindowsCondition implements Condition {

    /**
     * @param context 判断条件所能使用的上下文
     * @param metadata 注解元信息
     * @return 当前条件是否满足
     */
    @Override
    public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
        Environment environment = context.getEnvironment();
        String property = environment.getProperty("os.name");

        if (property.toLowerCase().contains("windows")){
            return true;
        }

        return false;
    }
}
```

### 8.@Scope

```java
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Scope {

	@AliasFor("scopeName")
	String value() default "";

	//下边这几个取值都可以
	/**
	 * @see ConfigurableBeanFactory#SCOPE_PROTOTYPE
	 * @see ConfigurableBeanFactory#SCOPE_SINGLETON
	 * @see org.springframework.web.context.WebApplicationContext#SCOPE_REQUEST
	 * @see org.springframework.web.context.WebApplicationContext#SCOPE_SESSION
	 * @see #value
	 */
	@AliasFor("value")
	String scopeName() default "";

	ScopedProxyMode proxyMode() default ScopedProxyMode.DEFAULT;

}
```

### 9.@Configuration示例

```java
@Configuration
@Import({Color.class,  MyImportSelector.class, MyImportBeanDefinitionRegister.class})
@ComponentScan(basePackages = {"org.project"},basePackageClasses = {MainService.class},useDefaultFilters = false,includeFilters = {
        @ComponentScan.Filter(type = FilterType.ANNOTATION,classes = {MyComponent.class}),
        @ComponentScan.Filter(type = FilterType.CUSTOM,classes = MyTypeFilter.class)
},excludeFilters = {
        @ComponentScan.Filter(type = FilterType.ANNOTATION,classes = {Component.class})
})
public class MyImportConfig {

    @Scope("singleton")
    @Bean
    public Color color(){
        return new Color();
    }
s
    @Scope(ConfigurableBeanFactory.SCOPE_SINGLETON)
    @Lazy
    @Bean("myFactoryBean")
    public MyFactoryBean myFactoryBean(){
        return new MyFactoryBean();
    }

    @Conditional(MyWindowsCondition.class)
    @Bean
    public Color windowsColor(){
        return new Color("windows");
    }

    @Conditional(MyLinuxCondition.class)
    @Bean
    public Color linuxColor(){
        return new Color("linux");
    }
}

```

## 属性赋值

### 1.导入Properties

@PropertySource(value={}) 导入properties文件

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Repeatable(PropertySources.class)
public @interface PropertySource {

	String name() default "";
    
    // For example, {@code "classpath:/com/myco/app.properties"} or
	//{@code "file:/path/to/file"}.
	String[] value();
    //路径不存在之后，是否忽略
	boolean ignoreResourceNotFound() default false;

	// A specific character encoding for the given resources, e.g. "UTF-8".
	String encoding() default "";

    //可以使用定制化的Factory进行解析
	Class<? extends PropertySourceFactory> factory() default PropertySourceFactory.class;

}
```



### 2.@Value



@Value可以支持：

- Spel表达式
- ${} 取Properties
- 普通字符串
- 文件资源
- Url资源

```java
	@Value("classpath:com/hry/spring/configinject/config.txt")
    private Resource resourceFile; // 注入文件资源

    @Value("http://www.baidu.com")
    private Resource testUrl; // 注入URL资源
```

## 自动装配





自动装配::  Spring利用依赖注入（DI），完成对IOC容器中中各个组件的依赖关系赋值

### 1.@Autowired

一般用于自动注入

```java
@Target({ElementType.CONSTRUCTOR, ElementType.METHOD, ElementType.PARAMETER, ElementType.FIELD, ElementType.ANNOTATION_TYPE})
```

#### Field

```java
@Autowired：自动注入：     
    1）、默认优先按照类型去容器中找对应的组件:applicationContext.getBean(BookDao.class);找到就赋值*     
    2）、如果找到多个相同类型的组件，再将属性的名称作为组件的id去容器中查找   applicationContext.getBean("bookDao")*    
    3）、@Qualifier("bookDao")：使用@Qualifier指定需要装配的组件的id，而不是使用属性名*     
    4）、自动装配默认一定要将属性赋值好，没有就会报错；*        可以使用@Autowired(required=false);*     
    5）、@Primary：让Spring进行自动装配的时候，默认使用首选的bean；*           
       也可以继续使用@Qualifier指定需要装配的bean的名字*    
```

- Method  一般放在set方法上    @Bean+方法参数：：参数从容器中获取;默认不写@Autowired效果是一样的；都能自动装配
- Constructor  可以默认调用有参构造器   如果组件只有一个有参构造器，这个有参构造器的@Autowired可以省略，参数位置的组件还是可以自动从容器中获取
- Paramater  可以从容器中获取参数对象

 @Inject + @Named 

```java
public class SimpleMovieLister {

    @Inject
    public void setMovieFinder(@Nullable MovieFinder movieFinder) {
        ...
    }
}
```



剪刀石头布    必然-凯文凯利 	道德情操理论



```java
@ImportResource("classpath:/com/acme/properties-config.xml")
```





### 2.@Qualifier

@Qualifier("maindaoLabel")

在存在多个相同类型的bean时，使用这个注解来指定要注入哪个对象

### 3.@Primary

在存在多个相同类型的bean时，以哪个作为默认的Bean

### 4.java自带注解

```java
Spring还支持使用@Resource(JSR250)和@Inject(JSR330)[java规范的注解]
    
 @Resource:可以和@Autowired一样实现自动装配功能；默认是按照组件名称进行装配的
     
 没有能支持@Primary功能没有支持@Autowired（reqiured=false）;
 @Inject: 需要导入javax.inject的包，和Autowired的功能一样。没有required=false的功能；
     
 @Autowired:Spring定义的； @Resource、@Inject都是java规范
```

### 5.Aware接口

Spring提供了一系列Aware接口，自定义组件可以使用这些接口来使用一下Spring底层的一些组件

<img src="/images/AwareInterface.png" alt="1581654199390" style="zoom: 67%; float: left;" />

```java
xxxAware：功能使用xxxProcessor；       
			ApplicationContextAware==》ApplicationContextAwareProcessor；
```

### 6.@Profile

Profile：

​	Spring为我们提供的可以根据当前环境，动态的激活和切换一系列组件的功能；

 		开发环境、测试环境、生产环境；
		 数据源：(/A)(/B)(/C)；


​	@Profile：指定组件在哪个环境的情况下才能被注册到容器中，不指定，任何环境下都能注册这个组件

​			1）、加了环境标识的bean，只有这个环境被激活的时候才能注册到容器中。默认是default环境
​			2）、写在配置类上，只有是指定的环境的时候，整个配置类里面的所有配置才能开始生效
​			3）、没有标注环境标识的bean在，任何环境下都是加载的；

ProFile激活：

- 命令行：使用命令行动态参数: 在虚拟机参数位置加载 -Dspring.profiles.active=test
- 代码：手动创建ApplicationContext,设置环境

```java
		AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext();
		//1、创建一个applicationContext
		//2、设置需要激活的环境
		applicationContext.getEnvironment().setActiveProfiles("dev");
		//3、注册主配置类
		applicationContext.register(MainConfigOfProfile.class);
		//4、启动刷新容器
		applicationContext.refresh();
		
		
		String[] namesForType = applicationContext.getBeanNamesForType(DataSource.class);
		for (String string : namesForType) {
			System.out.println(string);
		}
		
		Yellow bean = applicationContext.getBean(Yellow.class);
		System.out.println(bean);
		applicationContext.close();
```





## Bean的生命周期

### 1.@Bean注解

```java
@Target({ElementType.METHOD, ElementType.ANNOTATION_TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Bean {

	@AliasFor("name")
	String[] value() default {};

	@AliasFor("value")
	String[] name() default {};

	//是否自动注入
	Autowire autowire() default Autowire.NO;

	//初始化方法，在对象创建好，赋值好之后，调用
	String initMethod() default "";

	//在applicationContext关闭之前，指定销毁方法
	String destroyMethod() default AbstractBeanDefinition.INFER_METHOD;

}
```

通过实现InitializingBean DisposableBean

```java
public interface InitializingBean {
    //在factory创建完，并设置好属性之后，调用当前方法
	void afterPropertiesSet() throws Exception;

}
```

```java
public interface DisposableBean {

    //在销毁bean的时候，由BeanFacory调用
	void destroy() throws Exception;

}
```

@PostContruct，@Predestrory

```java
//初始化，以及依赖注入之后，才进行调用
@Documented
@Retention (RUNTIME)
@Target(METHOD)
public @interface PostConstruct {
}
```



```java
//作为回调的注释通知实例正在被容器移除
@Documented
@Retention (RUNTIME)
@Target(METHOD)
public @interface PreDestroy {
}
```



BeanPostProcessor

```java
public interface BeanPostProcessor {

    //在bean进行initialization callbacks( afterPropertiesSet a custom init-method)之前进行调用
	@Nullable
	default Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
		return bean;
	}

    //在bean进行初始化之后进行调用
	@Nullable
	default Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
		return bean;
	}

}
```

```text
Console:
MyBeanPostProcesser::postProcessBeforeInitialization...person
Color::init...
MyBeanPostProcesser::postProcessAfterInitialization...person
二月 14, 2020 10:08:15 上午 org.springframework.context.support.AbstractApplicationContext doClose
信息: Closing org.springframework.context.annotation.AnnotationConfigApplicationContext@6d1e7682: startup date [Fri Feb 14 10:08:15 CST 2020]; root of context hierarchy
Color::destroy...
```

生命周期中会有默认的初始化方法和销毁方法，默认方法名为init和 destroy 

复杂的Bean初始化逻辑用FactoryBean完成



@Autoweired(require=false) 优先按照相同的类型去寻找对应的组件 
 如果找到多个 就将属性的名称作为组件id去容器中寻找
@Qualifier() 可以指定需要装配的组件id
@Primary 表示首选 没明确指定的情况下 默认选用当前装配的bean

@Resource @Inject  java规范

@Resource 默认按照属性名称作为组件名称进行装配 不支持@primary和require

@Inject  需要导入javax.inject 和Autowired功能一样 没有require=false

@Autowired 
方法位置  @Bean+方法参数 参数从容器中获取 默认不写AutoWired
构造器位置 如果是有参构造器 这个有参构造器的参数组件会从ioc容器中获取，所以@Autowired可以省略
放在参数位置

@Profile 选择配置环境 没有添加这个注解的 所有环境下都会添加组件

写在配置类上 只有指定开发环境的时候 整个配置类才能生效
没有环境标识的bean 任何环境都加载
可以使用命令行加入 -Dspring.profiles.ative=test
还可以使用代码来实现 
new ApplicationContext（）
getEnvirent
setProfiles
regist
refresh

小结：多看官方文档

## Aop
```java
 AOP：【动态代理】
		指在程序运行期间动态的将某段代码切入到指定方法指定位置进行运行的编程方式；
 
	1、导入aop模块；Spring AOP：(spring-aspects)
	2、定义一个业务逻辑类（MathCalculator）；在业务逻辑运行的时候将日志进行打印（方法之前、方法运行结束、方法出现异常，xxx）
	3、定义一个日志切面类（LogAspects）：切面类里面的方法需要动态感知MathCalculator.div运行到哪里然后执行；
		通知方法：
			前置通知(@Before)：logStart：在目标方法(div)运行之前运行
			后置通知(@After)：logEnd：在目标方法(div)运行结束之后运行（无论方法正常结束还是异常结束）
  			返回通知(@AfterReturning)：logReturn：在目标方法(div)正常返回之后运行
  			异常通知(@AfterThrowing)：logException：在目标方法(div)出现异常以后运行
  			环绕通知(@Around)：动态代理，手动推进目标方法运行（joinPoint.procced()）
 	4、给切面类的目标方法标注何时何地运行（通知注解）；
  	5、将切面类和业务逻辑类（目标方法所在类）都加入到容器中;
  	6、必须告诉Spring哪个类是切面类(给切面类上加一个注解：@Aspect)
  	7、给配置类中加 @EnableAspectJAutoProxy 【开启基于注解的aop模式】
  		在Spring中很多的 @EnableXXX;
  
```



```java
@Component
@Aspect
public class LogAspects {
	
	//抽取公共的切入点表达式
	//1、本类引用
	//2、其他的切面引用
	@Pointcut("execution(public int org.project.aop.MathCalculator.*(..))")
	public void pointCut(){};
	
	//@Before在目标方法之前切入；切入点表达式（指定在哪个方法切入）
	@Before("pointCut()")
	public void logStart(JoinPoint joinPoint){
		Object[] args = joinPoint.getArgs();
		System.out.println(""+joinPoint.getSignature().getName()+"@Before:Paramer：{"+Arrays.asList(args)+"}");
	}
	
	@After("pointCut()")
	public void logEnd(JoinPoint joinPoint){
		System.out.println(""+joinPoint.getSignature().getName()+"@After");
	}
	
	//JoinPoint一定要出现在参数表的第一位
	@AfterReturning(value="pointCut()",returning="result")
	public void logReturn(JoinPoint joinPoint, Object result){
		System.out.println(""+joinPoint.getSignature().getName()+"正常返回...@AfterReturning:运行结果：{"+result+"}");
	}
	
	@AfterThrowing(value="pointCut()",throwing="exception")
	public void logException(JoinPoint joinPoint, Exception exception){
		System.out.println(""+joinPoint.getSignature().getName()+"异常。。。异常信息：{"+exception+"}");
	}

	@Around("pointCut()")
	public void around(ProceedingJoinPoint joinPoint) throws Throwable {
		System.out.println("LogAspects::around...");
		joinPoint.proceed();
	}

}
```

aop原理

```java
@EnableAspectJAutoProxy 中会自动注入 org.springframework.aop.aspectj.annotation.AnnotationAwareAspectJAutoProxyCreator类
    registry.registerBeanDefinition(AUTO_PROXY_CREATOR_BEAN_NAME(org.springframework.aop.config.internalAutoProxyCreator), beanDefinition);
    AnnotationAwareAspectJAutoProxyCreator 
    	AspectJAwareAdvisorAutoProxyCreator
    		AbstractAdvisorAutoProxyCreator
    			AbstractAutoProxyCreator implements SmartInstantiationAwareBeanPostProcessor, BeanFactoryAware
    
     AbstractAutoProxyCreator.setBeanFactory()
 * AbstractAutoProxyCreator.有后置处理器的逻辑；
 * 
 * AbstractAdvisorAutoProxyCreator.setBeanFactory()-》initBeanFactory()
 * 
 * AnnotationAwareAspectJAutoProxyCreator.initBeanFactory()
    	
```
流程：

```java
this();
register(annotatedClasses);
refresh(); -->registerBeanPostProcessors(beanFactory); 在这个方法中注册
    注册BeanPostProcessors
    // Separate between BeanPostProcessors that implement PriorityOrdered, Ordered, and the rest.
    //获取BeanPostProcessor
    beanFactory.getBean(ppName, BeanPostProcessor.class);
	先尝试从Cache中获取，如果获取失败，则进行创建
        MethodInterceptor
        
        
```



## 事务

```java
@EnableTransactionManagement注解中注入这两个对象
	org.springframework.context.annotation.AutoProxyRegistrar    implements ImportBeanDefinitionRegistrar
	org.springframework.transaction.annotation.ProxyTransactionManagementConfiguration  @Conifguration
    
2.AutoProxyRegistrar：注册InfrastructureAdvisorAutoProxyCreator.class
    InfrastructureAdvisorAutoProxyCreator：
  ProxyTransactionManagementConfiguration：
    public BeanFactoryTransactionAttributeSourceAdvisor transactionAdvisor  事务增强器：
    	1.解析事务注解
    	2.事务拦截器    TransactionInterceptor extends TransactionAspectSupport implements MethodInterceptor
    TransactionInterceptor；保存了事务属性信息，事务管理器；
 					他是一个 MethodInterceptor；
					在目标方法执行的时候；
					执行拦截器链；
						事务拦截器：
 							1）、先获取事务相关的属性
							2）、再获取PlatformTransactionManager，如果事先没有添加指定任何transactionmanger
							最终会从容器中按照类型获取一个PlatformTransactionManager；
							3）、执行目标方法
							如果异常，获取到事务管理器，利用事务管理回滚操作；
								如果正常，利用事务管理器，提交事务
    		
```

```java
public interface SmartInstantiationAwareBeanPostProcessor extends InstantiationAwareBeanPostProcessor {

	// 预测Bean的类型，返回第一个预测成功的Class类型，如果不能预测返回null
	@Nullable
	default Class<?> predictBeanType(Class<?> beanClass, String beanName) throws BeansException {
		return null;
	}

	// 选择合适的构造器，比如目标对象有多个构造器，在这里可以进行一些定制化，选择合适的构造器
	// beanClass参数表示目标实例的类型，beanName是目标实例在Spring容器中的name
	// 返回值是个构造器数组，如果返回null，会执行下一个PostProcessor的determineCandidateConstructors方法；否则选取该PostProcessor选择的构造器
	@Nullable
	default Constructor<?>[] determineCandidateConstructors(Class<?> beanClass, String beanName)
			throws BeansException {

		return null;
	}

	// 获得提前暴露的bean引用。主要用于解决循环引用的问题
	// 只有单例对象才会调用此方法
	default Object getEarlyBeanReference(Object bean, String beanName) throws BeansException {
		return bean;
	}

}
```


```java
public enum DispatcherType {
    FORWARD,
    INCLUDE,
    REQUEST,
    ASYNC,
    ERROR;

    private DispatcherType() {
    }
}
```

```java
public interface BeanDefinition extends AttributeAccessor, BeanMetadataElement {
    String SCOPE_SINGLETON = "singleton";
    String SCOPE_PROTOTYPE = "prototype";
    int ROLE_APPLICATION = 0;
    int ROLE_SUPPORT = 1;
    int ROLE_INFRASTRUCTURE = 2;

    void setParentName(String var1);

    String getParentName();

    void setBeanClassName(String var1);

    String getBeanClassName();

    void setScope(String var1);

    String getScope();

    void setLazyInit(boolean var1);

    boolean isLazyInit();

    void setDependsOn(String... var1);

    String[] getDependsOn();

    void setAutowireCandidate(boolean var1);

    boolean isAutowireCandidate();

    void setPrimary(boolean var1);

    boolean isPrimary();

    void setFactoryBeanName(String var1);

    String getFactoryBeanName();

    void setFactoryMethodName(String var1);

    String getFactoryMethodName();

    ConstructorArgumentValues getConstructorArgumentValues();

    MutablePropertyValues getPropertyValues();

    boolean isSingleton();

    boolean isPrototype();

    boolean isAbstract();

    int getRole();

    String getDescription();

    String getResourceDescription();

    BeanDefinition getOriginatingBeanDefinition();
}

```



```java
Srping范围：
    
    表格 1_.Bean 范围

范围	描述
singleton	(默认)根据 Spring IoC 容器将单个 bean 定义范围限定为单个 object 实例。
原型	为任意数量的 object 实例定义单个 bean 定义。
请求	将单个 bean 定义范围限定为单个 HTTP 请求的生命周期;也就是说，每个 HTTP 请求都有自己的 bean 实例，它是在单个 bean 定义的后面创建的。仅在 web-aware Spring ApplicationContext的 context 中有效。
session	将单个 bean 定义范围限定为 HTTP Session的生命周期。仅在 web-aware Spring ApplicationContext的 context 中有效。
globalSession	为 global HTTP Session的生命周期定义单个 bean 定义。通常仅在 Portlet context 中使用时有效。仅在 web-aware Spring ApplicationContext的 context 中有效。
应用	将单个 bean 定义范围限定为ServletContext的生命周期。仅在 web-aware Spring ApplicationContext的 context 中有效。
WebSocket	将单个 bean 定义范围限定为WebSocket的生命周期。仅在 web-aware Spring ApplicationContext的 context 中有效。
```



```
CGLIB 子类化只能覆盖 non-static 方法。因此，直接调用另一个@Bean方法将具有标准 Java 语义，从而导致直接从工厂方法本身返回独立实例。
```

```
PropertyEditorRegistrar
PropertyEditors
Application-level
beanfactorypostprocessor场景
ResourceLoader
Lifecycle
控制反转
GenericConverter
SPI
full-blown
```

```java
@GetMapping("/{id}")
@GetMapping("/{id}")
    public Person getPerson(@PathVariable Long id) {
        // ...
    }
    
    @GetMapping("/{name:[a-z-]+}-{version:\\d\\.\\d\\.\\d}{ext:\\.[a-z]+}")
public void handle(@PathVariable String version, @PathVariable String ext) {
    // ...
}

@Configuration
public class MyConfig {

    @Autowired
    public void setHandlerMapping(RequestMappingHandlerMapping mapping, UserHandler handler) 
            throws NoSuchMethodException {

        RequestMappingInfo info = RequestMappingInfo
                .paths("/user/{id}").methods(RequestMethod.GET).build(); 

        Method method = UserHandler.class.getMethod("getUser", Long.class); 

        mapping.registerMapping(info, handler, method); 
    }
}

@MatrixVariable

@PostMapping("/")
public String handle(@RequestPart("meta-data") MetaData metadata,
        @RequestPart("file-data") MultipartFile file) {
    // ...
}

@Controller
public class FormController {

    @InitBinder 
    public void initBinder(WebDataBinder binder) {
        SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd");
        dateFormat.setLenient(false);
        binder.registerCustomEditor(Date.class, new CustomDateEditor(dateFormat, false));
    }

    // ...
}-=/.
    
    location
    
    @PostMapping
public Callable<String> processUpload(final MultipartFile file) {

    return new Callable<String>() {
        public String call() throws Exception {
            // ...
            return "someView";
        }
    };
}

Cache-Control
    ETag过滤
```

