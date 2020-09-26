Spring容器

Spring容器是管理service和dao的。

 

SpringMVC容器

SpringMVC容器是管理controller对象的。



## 什么是spring

 Spring是一个轻量级Java开发框架  它为企业级开发提供给了丰富的功能，但是这些功能的底层都依赖于它的两个核心特性，也就是依赖注入（dependency injection，DI）和面向切面编程（aspect-oriented programming，AOP）。 

## IOC

 Ioc—Inversion of Control，即“控制反转”，不是什么技术，而是一种设计思想 

 把创建和查找依赖对象的控制权交给了容器，由容器进行注入组合对象，所以对象与对象之间是松散耦合 

### bean的生命周期

```java
// 忽略了无关代码
protected Object doCreateBean(final String beanName, final RootBeanDefinition mbd, final @Nullable Object[] args)
      throws BeanCreationException {

   // Instantiate the bean.
   BeanWrapper instanceWrapper = null;
   if (instanceWrapper == null) {
       // 实例化阶段
      instanceWrapper = createBeanInstance(beanName, mbd, args);
   }

   // Initialize the bean instance.
   Object exposedObject = bean;
   try {
       // 属性赋值阶段
      populateBean(beanName, mbd, instanceWrapper);
       // 初始化阶段
      exposedObject = initializeBean(beanName, exposedObject, mbd);
   }

   
   }
```

1. 实例化 Instantiation
2. 属性赋值 Populate
3. 初始化 Initialization
4. 销毁 Destruction

其中包含了很多钩子：

InstantiationAwareBeanPostProcessor.postProcessBeforeInstantiation
    **Instantiation**
//在赋值前进行调用 能够影响是否赋值
InstantiationAwareBeanPostProcessor.postProcessAfterInstantiation
    
InstantiationAwareBeanPostProcessor.postProcessPropertyValues(pvs, filteredPds, bw.getWrappedInstance(), beanName);
	**applyPropertyValues**(beanName, mbd, bw, pvs);



invokeAwareMethods

applyBeanPostProcessorsBeforeInitialization

​	**invokeInitMethods**

applyBeanPostProcessorsAfterInitialization



各种Aware接口

Aware Group1

BeanNameAware
BeanClassLoaderAware
BeanFactoryAware
    
Aware Group2
EnvironmentAware
EmbeddedValueResolverAware 这个知道的人可能不多，实现该接口能够获取Spring EL解析器，用户的自定义注解需要支持spel表达式的时候可以使用，非常方便。
ApplicationContextAware(ResourceLoaderAware\ApplicationEventPublisherAware\MessageSourceAware) 这几个接口可能让人有点懵，实际上这几个接口可以一起记，其返回值实质上都是当前的ApplicationContext对象，因为ApplicationContext是一个复合接口



applyBeanPostProcessorsBeforeInitialization

applyBeanPostProcessorsAfterInitialization

```java
	// 见名知意，初始化阶段调用的方法
    protected Object initializeBean(final String beanName, final Object bean, @Nullable RootBeanDefinition mbd) {

        // 这里调用的是Group1中的三个Bean开头的Aware
        invokeAwareMethods(beanName, bean);

        Object wrappedBean = bean;
        
        // 这里调用的是Group2中的几个Aware，
        // 而实质上这里就是前面所说的BeanPostProcessor的调用点！
        // 也就是说与Group1中的Aware不同，这里是通过BeanPostProcessor（ApplicationContextAwareProcessor）实现的。
        wrappedBean = applyBeanPostProcessorsBeforeInitialization(wrappedBean, beanName);
        // 下文即将介绍的InitializingBean调用点
        invokeInitMethods(beanName, wrappedBean, mbd);
        // BeanPostProcessor的另一个调用点
        wrappedBean = applyBeanPostProcessorsAfterInitialization(wrappedBean, beanName);

        return wrappedBean;
    }
```

### ioc容器加载过程

一般执行的时候 按照下边接口的顺序来进行执行

- 首先是一个synchronized加锁，当然要加锁，不然你先调一次refresh()然后这次还没处理完又调一次，就会乱套了；
-   接着往下看prepareRefresh();这个方法是做准备工作的，记录容器的启动时间、标记“已启动”状态、处理配置文件中的占位符，可以点进去看看，这里就不多说了。
-  下一步ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();这个就很重要了，这一步是把配置文件解析成一个个Bean，并且注册到BeanFactory中，注意这里只是注册进去，并没有初始化。先继续往下看，等会展开这个方法详细解读
-  然后是prepareBeanFactory(beanFactory);这个方法的作用是：设置 BeanFactory 的类加载器，添加几个 BeanPostProcessor，手动注册几个特殊的 bean，这里都是spring里面的特殊处理，然后继续往下看
-  postProcessBeanFactory(beanFactory);方法是提供给子类的扩展点，到这里的时候，所有的 Bean 都加载、注册完成了，但是都还没有初始化，具体的子类可以在这步的时候添加一些特殊的 BeanFactoryPostProcessor 的实现类，来完成一些其他的操作。
-  接下来是invokeBeanFactoryPostProcessors(beanFactory);这个方法是调用 BeanFactoryPostProcessor 各个实现类的 postProcessBeanFactory(factory) 方法；
-   然后是registerBeanPostProcessors(beanFactory);这个方法注册 BeanPostProcessor 的实现类，和上面的BeanFactoryPostProcessor 是有区别的，这个方法调用的其实是PostProcessorRegistrationDelegate类的registerBeanPostProcessors方法；这个类里面有个内部类BeanPostProcessorChecker，BeanPostProcessorChecker里面有两个方法postProcessBeforeInitialization和postProcessAfterInitialization，这两个方法分别在 Bean 初始化之前和初始化之后得到执行。然后回到refresh()方法中继续往下看
-   initMessageSource();方法是初始化当前 ApplicationContext 的 MessageSource，国际化处理，继续往下
-  initApplicationEventMulticaster();方法初始化当前 ApplicationContext 的事件广播器继续往下
-  onRefresh();方法初始化一些特殊的 Bean（在初始化 singleton beans 之前）；继续往下
-  registerListeners();方法注册事件监听器，监听器需要实现 ApplicationListener 接口；继续往下
-  重点到了：finishBeanFactoryInitialization(beanFactory);初始化所有的 singleton beans（单例bean），懒加载（non-lazy-init）的除外，这个方法也是等会细说
-  finishRefresh();方法是最后一步，广播事件，ApplicationContext 初始化完成

```java
PriorityOrdered.class
Ordered.class
最后执行其他的
```

```java
synchronized (this.startupShutdownMonitor) {
			// Prepare this context for refreshing.
    		//启动时间 内容校验
			prepareRefresh();

    		//创建一个beanFactory
			// Tell the subclass to refresh the internal bean factory.
			ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

    		//给beanFactory设置各种属性
    		//包括environment   systemProperties 系统属性   systemEnvironment  系统环境
			// Prepare the bean factory for use in this context.
			prepareBeanFactory(beanFactory);

			try {
				// Allows post-processing of the bean factory in context subclasses.
				postProcessBeanFactory(beanFactory);

				// Invoke factory processors registered as beans in the context.
                // 执行BeanDefinitionRegistryPostProcessor接口
				invokeBeanFactoryPostProcessors(beanFactory);

				// Register bean processors that intercept bean creation.
                //注册BeanPostProcessor
				registerBeanPostProcessors(beanFactory);

				// Initialize message source for this context.
                //初始化国际化组件
				initMessageSource();

				// Initialize event multicaster for this context.
                //初始化事件派发器
				initApplicationEventMulticaster();

				// Initialize other special beans in specific context subclasses.
				onRefresh();

				// Check for listener beans and register them.
                //ApplicationListener
				registerListeners();

				// Instantiate all remaining (non-lazy-init) singletons.
				finishBeanFactoryInitialization(beanFactory);

				// Last step: publish corresponding event.
				finishRefresh();
			}

			catch (BeansException ex) {
				if (logger.isWarnEnabled()) {
					logger.warn("Exception encountered during context initialization - " +
							"cancelling refresh attempt: " + ex);
				}

				// Destroy already created singletons to avoid dangling resources.
				destroyBeans();

				// Reset 'active' flag.
				cancelRefresh(ex);

				// Propagate exception to caller.
				throw ex;
			}

			finally {
				// Reset common introspection caches in Spring's core, since we
				// might not ever need metadata for singleton beans anymore...
				resetCommonCaches();
			}
		}
```


finishBeanFactoryInitialization


```java
首先获取所有的bean，
    for循环
    	获取bean定义信息RootBeanDefinition
    	不是工厂bean 单例 不是懒加载
    	getBean(beanName)
    		dogetBean(beanname)
    		从缓存中获取
    		private final Map<String, Object> singletonObjects = new ConcurrentHashMap<String, Object>(256);
			标记当前bean已经被创建
            获取Bean的定义信息
            如果依赖其他的bean 那么先创建其他bean
            createBean(beanName, mbd, args);
				让BeanPostProcessor先拦截返回代理对象
                doCreateBean(beanName, mbdToUse, args);创建Bean
                    InstantiationAwareBeanPostProcessor
                    	postProcessBeforeInstantiation
                	populateBean(beanName, mbd, instanceWrapper);
						postProcessAfterInitialization
                    initializeBean(beanName, exposedObject, mbd);
						执行Aware接口方法
                        postProcessBeforeInitialization
                        执行初始化方法
                        执行后置处理器初始化之后
                        postProcessAfterInitialization
                        注册销毁方法
              将创建的Bean添加到缓存中singletonObjects
```

### spring的bean创建

配置元数据：

​	硬编码  通过beanDefinitionRegister ，beanfactory来进行注册到ioc容器中    

​	注解  @Bean   @Import  @ComponentScan

​	配置文件  配置文件就主要可以配置要使用的构造器，属性

@Import 实际可以导入三种类型  **一般实体类，@ImportSelect，@ImportBeanDefinitionRegistrar**

### bean的作用域

**singleton**：这种 bean 范围是默认的，这种范围确保不管接受到多少个请求，每个容器中只
有一个 bean 的实例，单例的模式由 bean factory 自身来维护。
**prototype**：原形范围与单例范围相反，为每一个 bean 请求提供一个实例。
**request**：在请求 bean 范围内会每一个来自客户端的网络请求创建一个实例，在请求完成以
后，bean 会失效并被垃圾回收器回收。
**Session**：与请求范围类似，确保每个 session 中有一个 bean 的实例，在 session 过期后，
bean 会随之失效。更多面试资料在群619881427免费获取（JVM/并发编程/分布式/微服务/等面试视频学习资料都可以免费获取）
**global-session**：global-session 和 Portlet 应用相关。当你的应用部署在 Portlet 容器中工
作时，它包含很多 portlet。如果你想要声明让所有的 portlet 共用全局的存储变量的话，那么
这全局变量需要存储在 global-session 中。
全局作用域与 Servlet 中的 session 作用域效果相同 

## DI

 DI—Dependency Injection，即**“依赖注入”**：是组件之间依赖关系由容器在运行期决定，形象的说，即由容器动态的将**某个依赖关系**注入到组件之中。 

**依赖注入**明确描述了被注入对象依赖IoC容器**配置**依赖对象。

**构造器注入**（Constructor Injection）：Ioc容器会智能地选择选择和调用合的构造函数以创建依赖的对象。如果被选择的构造函数具有相应的参数，Ioc容器在调用构造函数之前解析注册的依赖关系并自行获得相应参数对象；

  **属性注入**（Property Injection）：如果需要使用到被依赖对象的某个属性，在被依赖对象被创建之后，Ioc容器会自动初始化该属性；

  **方法注入**（Method Injection）：如果被依赖对象需要调用某个方法进行相应的初始化，在该对象创建之后，Ioc容器会自动调用该方法。   

```java
public interface MagicBoss {
   Car getCar(); 
}

<!--①prototype类型的Bean-->
<bean id="car" class="com.smart.injectfun.Car"
    p:brand="红旗CA72" p:price="2000" scope="prototype"/>
<!--②实施方法注入-->
<bean id="magicBoss" class="com.smart.injectfun.MagicBoss" >
    <lookup-method name="getCar" bean="car"/>
</bean>
```

### spring循环依赖解决

解决不了 构造器中循环依赖的问题

- singletonFactories ： 单例对象工厂的cache  beanname-> beanFactory

- earlySingletonObjects ：提前曝光的单例对象的Cache beanname-> beaninstabce 创建过程中就可以获取得到

- singletonObjects：单例对象的cache  beanname->beaninstance

 从cache中获取这个单例的bean，这个缓存就是singletonObjects。如果获取不到，并且对象正在创建中，就再从二级缓存earlySingletonObjects中获取。如果还是获取不到且允许singletonFactories通过getObject()获取，就从三级缓存singletonFactory.getObject()(三级缓存)获取，如果获取到了则：从singletonFactories中移除，并放入earlySingletonObjects中。其实也就是从三级缓存移动到了二级缓存。 



 从上面三级缓存的分析，我们可以知道，Spring解决循环依赖的诀窍就在于singletonFactories这个三级cache。这个cache的类型是ObjectFactory。这里就是解决循环依赖的关键，发生在createBeanInstance之后，也就是说单例对象此时已经被创建出来(调用了构造器)。 

### 自动装配模式

no：这是Spring框架的默认设置，在该设置下自动装配是关闭的，开发者需要自行在bean定义中用标签明确的设置依赖关系。

byName：该选项可以根据bean名称设置依赖关系。当向一个bean中自动装配一个属性时，容器将根据bean的名称自动在在配置文件中查询一个匹配的bean。如果找到的话，就装配这个属性，如果没找到的话就报错。

byType：该选项可以根据bean类型设置依赖关系。当向一个bean中自动装配一个属性时，容器将根据bean的类型自动在在配置文件中查询一个匹配的bean。如果找到的话，就装配这个属性，如果没找到的话就报错。

constructor：造器的自动装配和byType模式类似，但是仅仅适用于与有构造器相同参数的bean，如果在容器中没有找到与构造器参数类型一致的bean，那么将会抛出异常。

autodetect：该模式自动探测使用构造器自动装配或者byType自动装配。首先，首先会尝试找合适的带参数的构造器，如果找到的话就是用构造器自动装配，如果在bean内部没有找到相应的构造器或者是无参构造器，容器就会自动选择byTpe的自动装配方式。

## aop

### aop原理

org.springframework.aop.aspectj.annotation.AnnotationAwareAspectJAutoProxyCreator

添加一个beanpostprocessor     SmartInstantiationAwareBeanPostProcessor

在加载factorybeanpostprocessor和beanpostprocessor之后   完成最后的bean创建

AnnotationAwareAspectJAutoProxyCreator

​	每次bean创建之后 判断是否属于advisedBeans

​	1）、获取当前bean的所有增强器（通知方法）  Object[]  specificInterceptors
  			1、找到候选的所有的增强器（找哪些通知方法是需要切入当前bean方法的）
 			2、获取到能在bean使用的增强器。
  			3、给增强器排序
  		2）、保存当前bean在advisedBeans中；
  		3）、如果当前bean需要增强，创建当前bean的代理对象；
  			1）、获取所有增强器（通知方法）
  			2）、保存到proxyFactory
 			3）、创建代理对象：Spring自动决定
  				JdkDynamicAopProxy(config);jdk动态代理；
  				ObjenesisCglibAopProxy(config);cglib的动态代理；
  		4）、给容器中返回当前组件使用cglib增强了的代理对象；
  		5）、以后容器中获取到的就是这个组件的代理对象，执行目标方法的时候，代理对象就会执行通知方法的流程；



3）、目标方法执行	；
  		容器中保存了组件的代理对象（cglib增强后的对象），这个对象里面保存了详细信息（比如增强器，目标对象，xxx）；
  		1）、CglibAopProxy.intercept();拦截目标方法的执行
  		2）、根据ProxyFactory对象获取将要执行的目标方法拦截器链；
  			List<Object> chain = this.advised.getInterceptorsAndDynamicInterceptionAdvice(method, targetClass);
  			1）、List<Object> interceptorList保存所有拦截器 5
 				一个默认的ExposeInvocationInterceptor 和 4个增强器；
 			2）、遍历所有的增强器，将其转为Interceptor；
  				registry.getInterceptors(advisor);
 			3）、将增强器转为List<MethodInterceptor>；
  				如果是MethodInterceptor，直接加入到集合中
 				如果不是，使用AdvisorAdapter将增强器转为MethodInterceptor；
				转换完成返回MethodInterceptor数组；

  		3）、如果没有拦截器链，直接执行目标方法;
  			拦截器链（每一个通知方法又被包装为方法拦截器，利用MethodInterceptor机制）
  		4）、如果有拦截器链，把需要执行的目标对象，目标方法，
  			拦截器链等信息传入创建一个 CglibMethodInvocation 对象，
  			并调用 Object retVal =  mi.proceed();
  		5）、拦截器链的触发过程;
  			1)、如果没有拦截器执行执行目标方法，或者拦截器的索引和拦截器数组-1大小一样（指定到了最后一个拦截器）执行目标方法；
  			2)、链式获取每一个拦截器，拦截器执行invoke方法，每一个拦截器等待下一个拦截器执行完成返回以后再来执行；
  				拦截器链的机制，保证通知方法与目标方法的执行顺序；

数组：

1 AspectAtferThrowing

2 AspectAtferReturn

3 AspectJAfterAdvice

4 MethodBeforeAdviceInterceptor

## BeanFactory：

是Spring里面最低层的接口，提供了最简单的容器的功能，只提供了实例化对象和拿对象的功能

## ApplicationContext：

应用上下文，继承BeanFactory接口，它是Spring的一各更高级的容器，提供了更多的有用的功能

1) 国际化（MessageSource）

2) 访问资源，如URL和文件（ResourceLoader）

3) 载入多个（有继承关系）上下文 ，使得每一个上下文都专注于一个特定的层次，比如应用的web层  

4) 消息发送、响应机制（ApplicationEventPublisher）

载入多个（有继承关系）上下文 ，使得每一个上下文都专注于一个特定的层次，比如应用的web层。

## BeanFactory 和 ApplicationContext

加载方式

BeanFactroy采用的是**延迟加载**形式来注入Bean的，即只有在使用到某个Bean时(调用getBean())，才对该Bean进行加载实例化。

这样，我们就不能发现一些存在的Spring的配置问题。如果Bean的某一个属性没有注入，

BeanFacotry加载后，直至第一次使用调用getBean方法才会抛出异常。

ApplicationContext，它是在容器启动时，**一次性创建**了所有的Bean。这样，在容器启动时，我们就可以发现Spring中存在的配置错误，这样有利于检查所依赖属性是否注入。 ApplicationContext启动后预载入所有的单实例Bean，通过预载入单实例bean ,确保当你需要的时候，你就不用等待，因为它们已经创建好了。

相对于基本的BeanFactory，ApplicationContext 唯一的不足是占用内存空间。当应用程序配置Bean较多时，程序启动较慢。

创建方式

BeanFactory通常以编程的方式被创建，ApplicationContext还能以声明的方式创建，如使用ContextLoader。

注册方式

BeanFactory和ApplicationContext都支持BeanPostProcessor、BeanFactoryPostProcessor的使用，但两者之间的区别是：BeanFactory需要手动注册，而ApplicationContext则是自动注册。

## 什么是Spring IOC 容器？

控制反转即IoC (Inversion of Control)，它把传统上由程序代码直接操控的对象的调用权交给容器，通过容器来实现对象组件的装配和管理。所谓的“控制反转”概念就是对组件对象控制权的转移，从程序代码本身转移到了外部容器。

Spring IOC 负责创建对象，管理对象（通过依赖注入（DI），装配对象，配置对象，并且管理这些对象的整个生命周期。

### 配置元数据

基于 XML 的配置
基于注解的配置
基于 Java 的配置 

## Spring如何处理线程并发问题？

在一般情况下，只有无状态的Bean才可以在多线程环境下共享，在Spring中，绝大部分Bean都可以声明为singleton作用域，因为Spring对一些Bean中非线程安全状态采用ThreadLocal进行处理，解决线程安全问题。

ThreadLocal和线程同步机制都是为了解决多线程中相同变量的访问冲突问题。同步机制采用了“时间换空间”的方式，仅提供一份变量，不同的线程在访问前需要获取锁，没获得锁的线程则需要排队。而ThreadLocal采用了“空间换时间”的方式。

ThreadLocal会为每一个线程提供一个独立的变量副本，从而隔离了多个线程对数据的访问冲突。因为每一个线程都拥有自己的变量副本，从而也就没有必要对该变量进行同步了。ThreadLocal提供了线程安全的共享对象，在编写多线程代码时，可以把不安全的变量封装进ThreadLocal。

## 说一下 spring 的事务隔离？

spring 有五大隔离级别，默认值为 ISOLATION_DEFAULT（使用数据库的设置），其他四个隔离级别和数据库的隔离级别一致：

ISOLATION_DEFAULT：用底层数据库的设置隔离级别，数据库设置的是什么我就用什么；

ISOLATION_READ_UNCOMMITTED：未提交读，最低隔离级别、事务未提交前，就可被其他事务读取（会出现幻读、脏读、不可重复读）；

ISOLATION_READ_COMMITTED：提交读，一个事务提交后才能被其他事务读取到（会造成幻读、不可重复读），SQL server 的默认级别；

ISOLATION_REPEATABLE_READ：可重复读，保证多次读取同一个数据时，其值都和事务开始时候的内容是一致，禁止读取到别的事务未提交的数据（会造成幻读），MySQL 的默认级别；

ISOLATION_SERIALIZABLE：序列化，代价最高最可靠的隔离级别，该隔离级别能防止脏读、不可重复读、幻读。



脏读 ：表示一个事务能够读取另一个事务中还未提交的数据。比如，某个事务尝试插入记录 A，此时该事务还未提交，然后另一个事务尝试读取到了记录 A。

不可重复读 ：是指在一个事务内，多次读同一数据。

幻读 ：指同一个事务内多次查询返回的结果集不一样。比如同一个事务 A 第一次查询时候有 n 条记录，但是第二次同等条件下查询却有 n+1 条记录，这就好像产生了幻觉。发生幻读的原因也是另外一个事务新增或者删除或者修改了第一个事务结果集里面的数据，同一个记录的数据内容被修改了，所有数据行的记录就变多或者变少了。

## AOP（拦截器）



## spring设计模式

### 创建者模式

简单工厂：beanfactory  根据传入的参数 动态的决定要创建的产品类

工厂方法: factoryBean  实现了BeanFactory接口的bean叫做Factory的bean

- mybatis的sqlsessionFactory就是工厂方法的实现

单例模式：Bean默认为单例模式。

- 三重缓存

### 结构性模式

适配器模式：springmvc的HandlerAdapter  使得handle拓展也变得容易 加适配器就行了

装饰者设计模式： 可以动态地给对象增加些额外的属性或行为。相比于使用继承，装饰者模式更加灵活。Spring 中配置DataSource的时候，DataSource可能是不同的数据库和数据源。我们能否根据客户的需求在少修改原有类的代码下切换不同的数据源？这个时候据需要用到装饰者模式。

代理模式：Spring的AOP功能用到了JDK的动态代理和CGLIB字节码生成技术；

### 行为方法模式

模板方法：用来解决代码重复的问题。比如. RestTemplate, JmsTemplate, JpaTemplate。

观察者模式：定义对象键一种一对多的依赖关系，当一个对象的状态发生改变时，所有依赖于它的对象都会得到通知被制动更新，如Spring中listener的实现–ApplicationListener。 

 责任链模式： 责任链模式就是很多对象由每个对象对其下家的引用而连接起来形成一条链，请求在这条链上传递，知道链上的某个对象处理此请求，或者每个对象都可以处理请求，并传给"下家",知道最终链上每个对象都处理完。 filterchain

策略设计模式
**Spring 框架的资源访问接口就是基于策略设计模式实现的**。该接口提供了更强的资源访问能力，Spring框架本身大量使用了Resource接口来访问底层资源。Resource接口本身没有提供访问任何底层资源的实现逻辑，针对不同的额底层资源，Spring将会提供不同的Resource实现类，不同的实现类负责不同的资源访问类型。

Spring 为 Resource 接口提供了如下实现类： 
UrlResource：访问网络资源的实现类。
ClassPathResource：访问类加载路径里资源的实现类。
FileSystemResource：访问文件系统里资源的实现类。
ServletContextResource：访问相对于 ServletContext 路径里的资源的实现类.
InputStreamResource：访问输入流资源的实现类。
ByteArrayResource：访问字节数组资源的实现类。 
这些 Resource 实现类，针对不同的的底层资源，提供了相应的资源访问逻辑，并提供便捷的包装，以利于客户端程序的资源访问。

## springmvc

### mvc过程

1.进入DispatcherServlet的doservice之后 

2.遍历handlermappings数组，把requeat传进去之后从handlermapping中找到对应的处理器

​	2.1 首先会获取对应的handler

​	2.2 获取之后 遍历Interceptor 如果路径匹配 就加入到HandlerExecutionChain中

3.根据handler找到对应的handlerAdapter

4.执行拦截器的prehandle方法
5.执行controller 返回modelAndView
6.执行posthandler
7.根据配置的viewResolver和locale去解析视图
8.调用拦截器的afterCompletion
9.响应用户

```java
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
```



## springboot 

### 自动配置

```java
@SpringBootApplication注解
    @EnableAutoConfiguration
    @Import(AutoConfigurationImportSelector.class)
    public final class SpringFactoriesLoader
        public static final String FACTORIES_RESOURCE_LOCATION = "META-INF/spring.factories";
加载每个jar包下边的这个文件
    org.springframework.boot.autoconfigure.EnableAutoConfiguration=后边的为自动配置类
    每一个配置类都是一个Conguration  带有各种condition  全部为true才生效
    
@Configuration
@AutoConfigureOrder(Ordered.HIGHEST_PRECEDENCE)
@ConditionalOnClass(ServletRequest.class)
@ConditionalOnWebApplication(type = Type.SERVLET)
@EnableConfigurationProperties(ServerProperties.class)
@Import({ ServletWebServerFactoryAutoConfiguration.BeanPostProcessorsRegistrar.class,
		ServletWebServerFactoryConfiguration.EmbeddedTomcat.class,
		ServletWebServerFactoryConfiguration.EmbeddedJetty.class,
		ServletWebServerFactoryConfiguration.EmbeddedUndertow.class })
public class ServletWebServerFactoryAutoConfiguration {
    
@ConfigurationProperties(prefix = "server", ignoreUnknownFields = true)
public class ServerProperties {
    
//properties  中可以填写的属性 就来自这里
    @EnableConfigurationProperties注解：开启配置属性
        诸多的XxxxAutoConfiguration自动配置类，就是Spring容器的JavaConfig形式，作用就是为Spring 容器导入bean，而所有导入的bean所需要的属性都通过xxxxProperties的bean来获得。
        
        
        
        Spring Boot启动的时候会通过@EnableAutoConfiguration注解找到META-INF/spring.factories配置文件中的所有自动配置类，并对其进行加载，而这些自动配置类都是以AutoConfiguration结尾来命名的，它实际上就是一个JavaConfig形式的Spring容器配置类，它能通过以Properties结尾命名的类中取得在全局配置文件中配置的属性如：server.port，而XxxxProperties类是通过@ConfigurationProperties注解与全局配置文件中对应的属性进行绑定的。
```

### starter

starter是一组方便的依赖关系描述符，可以获取所需的spring相关技术，无需复制依赖描述符。

也叫场景启动器

#### 自定义starter

1.首先建立一个空工程，然后引入自定义的configueration工程

2.对于自定义工程的要求

​	2.1 继承自spring-boot-starter-parent

​	2.2 依赖spring-boot-starter       spring-boot-configuration-processor

​	2.3 编写配置类   @Configuration 	 @EnableConfigurationProperties(HelloProperties.class)   定义一个实体类映射信息properties

​	2.4 创建META-INF/spring.factores 

```java
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
com.atguigu.starter.HelloServiceAutoConfiguration(对应的配置类)
```



```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.atguigu.starter</groupId>
    <artifactId>atguigu-spring-boot-starter</artifactId>
    <version>1.0-SNAPSHOT</version>

    <!--启动器-->
    <dependencies>

        <!--引入自动配置模块-->
        <dependency>
            <groupId>com.atguigu.starter</groupId>
            <artifactId>atguigu-spring-boot-starter-autoconfigurer</artifactId>
            <version>0.0.1-SNAPSHOT</version>
        </dependency>
    </dependencies>

</project>
```

