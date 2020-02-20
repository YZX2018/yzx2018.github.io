---
layout: post
title: "spring源码之Bean的创建过程与Spring AOP原理"
date: 2019-07-13
tags: java
---

#Bean的创建过程
#####spring注解版单实例Bean的创建是容器启动的时候调用getBean(beanName)创建，然后保存到IOC容器中；多实例Bean每次都会getBean(beanName)创建新的实例
调用的方法是
**org.springframework.context.support.AbstractApplicationContext#refresh**
&emsp;**org.springframework.context.support.AbstractApplicationContext#finishBeanFactoryInitialization**
&emsp;&emsp;**org.springframework.beans.factory.config.ConfigurableListableBeanFactory#preInstantiateSingletons**
&emsp;&emsp;&emsp;**org.springframework.beans.factory.support.AbstractBeanFactory#getBean(java.lang.String)**
&emsp;&emsp;&emsp;&emsp;**org.springframework.beans.factory.support.AbstractBeanFactory#doGetBean**

#####getBean(java.lang.String)方法调用了doGetBean(name, null, null, false)处理Bean的创建
我们来看看doGetBean()做了哪些处理，**这里主要分析单实例Bean的创建过程**。
```
protected <T> T doGetBean(final String name, @Nullable final Class<T> requiredType,
			@Nullable final Object[] args, boolean typeCheckOnly) throws BeansException {

		final String beanName = transformedBeanName(name);
		Object bean;

		// 1.先获取缓存中保存的单实例Bean。如果能获取到说明这个Bean之前被创建过（所有创建过的单实例Bean都会被缓存起来），如果bean的定义信息scope是多例，就不会被缓存
		Object sharedInstance = getSingleton(beanName);
		if (sharedInstance != null && args == null) {
			......
             // 如果缓存中有,判断这个Bean是否是FactoryBean，如果不是FactoryBean，直接返回sharedInstance
			bean = getObjectForBeanInstance(sharedInstance, name, beanName, null);
		}
              // 2.如果缓存中不存在对应Bean的实例
		else {
			// 判断这个bean是否正在创建中
			if (isPrototypeCurrentlyInCreation(beanName)) {
				throw new BeanCurrentlyInCreationException(beanName);
			}

		......忽略部份代码
                        
			if (!typeCheckOnly) {
                // 3.标记当前bean已经被创建（或即将创建）
				markBeanAsCreated(beanName);
			}

			try {
                              //4.获取Bean的定义信息
				final RootBeanDefinition mbd = getMergedLocalBeanDefinition(beanName);
				checkMergedBeanDefinition(mbd, beanName, args);

				// 5.获取当前Bean所依赖的其他Bean
				String[] dependsOn = mbd.getDependsOn();
				if (dependsOn != null) {
					for (String dep : dependsOn) {
						if (isDependent(beanName, dep)) {
							throw new BeanCreationException(mbd.getResourceDescription(), beanName,
									"Circular depends-on relationship between '" + beanName + "' and '" + dep + "'");
						}
						registerDependentBean(dep, beanName);
						try {
                         // 6.如果当前Bean依赖了其他Bean，通过getBean()先把所依赖的Bean先创建出来
							getBean(dep);
						}
						catch (NoSuchBeanDefinitionException ex) {
							throw new BeanCreationException(mbd.getResourceDescription(), beanName,
									"'" + beanName + "' depends on missing bean '" + dep + "'", ex);
						}
					}
				}

				// 7.如果是单实例Bean
				if (mbd.isSingleton()) {
                     // 创建Bean实例对象并添加到IOC容器中
					sharedInstance = getSingleton(beanName, () -> {
						try {
                                  //8.单实例Bean的创建
							return createBean(beanName, mbd, args);
						}
						catch (BeansException ex) {
							destroySingleton(beanName);
							throw ex;
						}
					});
                                      // 判断当前Bean是不是FactoryBean，不是就直接返回sharedInstance(原来的Bean实例)
					bean = getObjectForBeanInstance(sharedInstance, name, beanName, mbd);
				}
                           
......忽略部份代码
	}
```
上**面代码 8.单实例Bean的创建，getSingleton方法内调用了createBean(beanName, mbd, args)是创建Bean的流程   分析createBean(beanName, mbd, args) **
```
protected Object createBean(String beanName, RootBeanDefinition mbd, @Nullable Object[] args)
			throws BeanCreationException {

		if (logger.isTraceEnabled()) {
			logger.trace("Creating instance of bean '" + beanName + "'");
		}
		RootBeanDefinition mbdToUse = mbd;

		
		......忽略部份代码

		try {
           //让BeanPostProcessors(后置处理器)有机会返回代理而不是目标bean实例
	    // 判断容器中是否有InstantiationAwareBeanPostProcessor的实现(比如开启AOP时就会调用)，如果有则执行InstantiationAwareBeanPostProcessor
          // InstantiationAwareBeanPostProcessors会先触发：postProcessBeforeInstantiation()；
          //如果有返回值(一般不会有返回值)：再触发postProcessAfterInitialization()
			Object bean = resolveBeforeInstantiation(beanName, mbdToUse);
			if (bean != null) {
				return bean;
			}
		}
		catch (Throwable ex) {
			throw new BeanCreationException(mbdToUse.getResourceDescription(), beanName,
					"BeanPostProcessor before instantiation of bean failed", ex);
		}

		try {
             // 如果前面的InstantiationAwareBeanPostProcessor没有返回代理对象，调用doCreateBean创建
			Object beanInstance = doCreateBean(beanName, mbdToUse, args);
			if (logger.isTraceEnabled()) {
				logger.trace("Finished creating instance of bean '" + beanName + "'");
			}
			return beanInstance;
		}
		......
	}
```
最终调用doCreateBean(beanName, mbdToUse, args)方法创建完成Bean的创建
```
protected Object doCreateBean(final String beanName, final RootBeanDefinition mbd, final @Nullable Object[] args)
			throws BeanCreationException {

		// Instantiate the bean.
              //BeanWrapper  存储对象的实例
		BeanWrapper instanceWrapper = null;
		if (mbd.isSingleton()) {
			instanceWrapper = this.factoryBeanInstanceCache.remove(beanName);
		}
		if (instanceWrapper == null) {
                        // 1.创建Bean实例(利用工厂方法或者对象的构造器创建出Bean实例)
			instanceWrapper = createBeanInstance(beanName, mbd, args);
		}
		......

		// Allow post-processors to modify the merged bean definition.
		synchronized (mbd.postProcessingLock) {
			if (!mbd.postProcessed) {
				try {
             //  2.调用MergedBeanDefinitionPostProcessor(后置处理器)的postProcessMergedBeanDefinition方法
					applyMergedBeanDefinitionPostProcessors(mbd, beanType, beanName);
				}
				catch (Throwable ex) {
					throw new BeanCreationException(mbd.getResourceDescription(), beanName,
							"Post-processing of merged bean definition failed", ex);
				}
				mbd.postProcessed = true;
			}
		}

		......

		// Initialize the bean instance.   初始化bean实例
		Object exposedObject = bean;
		try {
  // 3.给bean的属性赋值
      // 1)赋值之前会从容器中获取InstantiationAwareBeanPostProcessor(后置处理器)执行  postProcessProperties()
     // 2)再执行postProcessPropertyValues() (应用Bean属性的值,即可以对属性值进行修改(这个时候属性值还未被设置(还未调用set赋值)，但是我们可以修改原本该设置进去的属性值))
     // 3)最后属性利用setter方法等进行赋值 applyPropertyValues(beanName, mbd, bw, pvs)
			populateBean(beanName, mbd, instanceWrapper);
            // 4.Bean初始化
                //1) invokeAwareMethods(beanName, bean);执行xxxAware接口的方法
                // 2)applyBeanPostProcessorsBeforeInitialization(wrappedBean, beanName) 执行后置处理器初始化的前置方法(bean初始化之前调用)  BeanPostProcessor.postProcessBeforeInitialization（）
                // 3)执行初始化方法 
                          //1)如果该Bean实现了InitializingBean接口；执行接口规定的初始化    
                         // 2)如果指定了自定义初始化方法( @Bean(initMethod = "init"))，执行自定义初始化方法
                //4.applyBeanPostProcessorsAfterInitialization执行后置处理器初始化的后置方法(bean初始化之后调用)BeanPostProcessor.postProcessAfterInitialization()		
exposedObject = initializeBean(beanName, exposedObject, mbd)
		}
		catch (Throwable ex) {
			if (ex instanceof BeanCreationException && beanName.equals(((BeanCreationException) ex).getBeanName())) {
				throw (BeanCreationException) ex;
			}
			else {
				throw new BeanCreationException(
						mbd.getResourceDescription(), beanName, "Initialization of bean failed", ex);
			}
		}

		......
		// Register bean as disposable.
		try {
                       // 注册Bean的销毁方法
			registerDisposableBeanIfNecessary(beanName, bean, mbd);
		}
		catch (BeanDefinitionValidationException ex) {
			throw new BeanCreationException(
					mbd.getResourceDescription(), beanName, "Invalid destruction signature", ex);
		}

		return exposedObject;
	}
```
createBean创建完成之后返回Bean实例对象然后继续走getSingleton方法将Bean实例缓存到容器的singletonObjects的Map中（调用addSingleton()）
```
if (mbd.isSingleton()) {
                     // createBean创建Bean实例对象，getSingleton将其缓存到singletonObjects的Map中
					sharedInstance = getSingleton(beanName, () -> {
						try {
                                  //8.单实例Bean的创建
							return createBean(beanName, mbd, args);
						}
						catch (BeansException ex) {
							destroySingleton(beanName);
							throw ex;
						}
					});
                    // 判断当前Bean是不是FactoryBean，不是就直接返回sharedInstance(原来的Bean实例)
					bean = getObjectForBeanInstance(sharedInstance, name, beanName, mbd);
				}
```
总结单实例Bean的创建过程
>#####getBean(beanName) 创建对象
>#####&emsp;1.doGetBean(name, null, null, false)
>&emsp;&emsp;1)先获取缓存中保存的单实例Bean。如果能获取到说明这个Bean之前被创建过，直接返回回（所有创建过的单实例Bean都会被缓存起来，如果bean的定义信息scope是多例，就不会被缓存）
>&emsp;&emsp;2)缓存中获取不到，开始Bean的创建对象流程
>&emsp;&emsp;3)标记当前bean已经被创建（或即将创建）
>&emsp;&emsp;4)获取Bean的定义信息(依赖、属性、作用域等Bean的信息都存在BeanDefinition)
>&emsp;&emsp;5)从定义信息中getDependsOn获取当前Bean所依赖的其他Bean,如果有调用getBean()把依赖的Bean先创建出来(**注意，这个依赖是指@DependsOn指定的其他bean，当前bean的属性依赖**)
>&emsp;&emsp;6)createBean(beanName, mbd, args)  创建当前Bean的实例对象
>&emsp;&emsp;&emsp;&emsp;1、调用Object bean = resolveBeforeInstantiation(beanName, mbdToUse); 让BeanPostProcessors(后置处理器)有机会返回代理而不是目标bean实例，判断容器中是否有InstantiationAwareBeanPostProcessor的实现(比如开启AOP时就会调用)，如果有InstantiationAwareBeanPostProcessors会先触发：postProcessBeforeInstantiation()；如果有返回值(一般不会有返回值)：再触发postProcessAfterInitialization()
>&emsp;&emsp;&emsp;&emsp;2、如果前面的InstantiationAwareBeanPostProcessor没有返回代理对象，调用doCreateBean创建
>&emsp;&emsp;&emsp;&emsp;3、Object beanInstance = doCreateBean(beanName, mbdToUse, args) 创建Bean实例对象
>&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;1.instanceWrapper = createBeanInstance(beanName, mbd, args); 创建Bean实例(利用工厂方法或者对象的构造器创建出Bean实例)
>&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;2.applyMergedBeanDefinitionPostProcessors(mbd, beanType, beanName);从容器中获取MergedBeanDefinitionPostProcessor(后置处理器)并调用postProcessMergedBeanDefinition方法
>&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;3.populateBean(beanName, mbd, instanceWrapper); 给Bean属性赋值
>&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;1/赋值之前会从容器中获取InstantiationAwareBeanPostProcessor(后置处理器)
>&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;2/执行InstantiationAwareBeanPostProcessor的postProcessProperties()进行处理
>&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;3/再执行postProcessPropertyValues() (应用Bean属性的值,即可以对属性值进行修改(这个时候属性值还未被设置(还未调用set赋值)，但是我们可以修改原本该设置进去的属性值))
>&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;4/最后属性利用setter方法进行赋值 applyPropertyValues(beanName, mbd, bw, pvs)
>&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;4.initializeBean(beanName, exposedObject, mbd); 对Bean进行初始化处理
>&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;1/invokeAwareMethods(beanName, bean);判断当前Bean是否实现了xxxAware的接口，如果是就执行xxxAware接口的方法(比如ApplicationContextAware:setApplicationContext())
>&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;2/applyBeanPostProcessorsBeforeInitialization(wrappedBean, beanName) 后置处理器的前置处理BeanPostProcessor.postProcessBeforeInitialization() (bean初始化之前调用)
>&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;3/执行初始化方法，首先判断：如果该Bean实现了InitializingBean接口,执行接口规定的初始化 。然后判断：如果指定了自定义初始化方法( @Bean(initMethod = "init"))，执行自定义初始化init方法
>&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;4/applyBeanPostProcessorsAfterInitialization(wrappedBean, beanName)执行后置处理器后置方法BeanPostProcessor.postProcessAfterInitialization()(bean初始化之后调用)
>&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;5.registerDisposableBeanIfNecessary(beanName, bean, mbd); 注册Bean的销毁方法
>&emsp;&emsp;7)Bean创建完成后，执行getSingleton方法将创建的Bean添加到IOC容器中(缓存到singletonObjects(Map集合)中，IOC容器有很多个map集合，这是其中一个)

---
#####Bean生命周期流程图
![Bean生命周期流程图](https://upload-images.jianshu.io/upload_images/14890912-faa093376610044b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

---
#spring aop的原理分析
AOP：面向切面的编程，在方法运行时，动态将方法切入到其他代码中运行
切面（@Aspect）、切入点（@Pointcut）通知(@Before、@After、@AfterReturning、@AfterThrowing、@Around)

spring注解方式开启aop功能只需要在配置类上加上@EnableAspectJAutoProxy注解
```
@Configuration
@EnableAspectJAutoProxy
public class DemoApplication {
	public static void main(String[] args) {
    AnnotationConfigApplicationContext applicationContextionContext = new AnnotationConfigApplicationContext(DemoApplication.class);
}
```


AOP原理主要通过@EnableAspectJAutoProxy注解来实现
#####分析@EnableAspectJAutoProxy
点进@EnableAspectJAutoProxy中，发现@Import(AspectJAutoProxyRegistrar.class)，这个注解作用是把AspectJAutoProxyRegistrar注册到容器中
```
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Import(AspectJAutoProxyRegistrar.class)  // 把AspectJAutoProxyRegistrar导入到容器中
public @interface EnableAspectJAutoProxy {
...... 忽略部份代码
}
```
#####1、进入AspectJAutoProxyRegistrar类
&emsp;&emsp; 1.这个类实现了ImportBeanDefinitionRegistrar，可通过BeanDefinitionRegistry  registry手动注册组件到容器中
&emsp;&emsp; 2.AopConfigUtils.registerAspectJAnnotationAutoProxyCreatorIfNecessary(registry) 手动给容器注册一个AspectJAnnotationAutoProxyCreator组件
```
class AspectJAutoProxyRegistrar implements ImportBeanDefinitionRegistrar {

	/**
	 * Register, escalate, and configure the AspectJ auto proxy creator based on the value
	 * of the @{@link EnableAspectJAutoProxy#proxyTargetClass()} attribute on the importing
	 * {@code @Configuration} class.
	 */
	@Override
	public void registerBeanDefinitions(
			AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {
                // 给容器注册一个AspectJAnnotationAutoProxyCreator类
		AopConfigUtils.registerAspectJAnnotationAutoProxyCreatorIfNecessary(registry);
         ...... 忽略部份代码
}

```
#####2、进入上面的registerAspectJAnnotationAutoProxyCreatorIfNecessary()方法，这个方法最终包装到registerOrEscalateApcAsRequired()方法来处理
&emsp;&emsp; 1.注册一个beanName(bean的id)为internalAutoProxyCreator的AnnotationAwareAspectJAutoProxyCreator类到容器中
```
@Nullable
	private static BeanDefinition registerOrEscalateApcAsRequired(
			Class<?> cls, BeanDefinitionRegistry registry, @Nullable Object source) {
          ......忽略部份代码
		RootBeanDefinition beanDefinition = new RootBeanDefinition(cls);
		beanDefinition.setSource(source);
		beanDefinition.getPropertyValues().add("order", Ordered.HIGHEST_PRECEDENCE);
		beanDefinition.setRole(BeanDefinition.ROLE_INFRASTRUCTURE);
    // 传入的cls为AnnotationAwareAspectJAutoProxyCreator.class
 // public static final String AUTO_PROXY_CREATOR_BEAN_NAME ="org.springframework.aop.config.internalAutoProxyCreator";
        // 注册一个beanName(bean的id)为internalAutoProxyCreator的AnnotationAwareAspectJAutoProxyCreator类到容器中
		registry.registerBeanDefinition(AUTO_PROXY_CREATOR_BEAN_NAME, beanDefinition);
		return beanDefinition;
	}
```
#####3、最终将AnnotationAwareAspectJAutoProxyCreator类的定义信息注册到容器中(beanDefinitionMap(Map集合)中)，这个类是后置处理器的实现类(后置处理器会在bean初始化前后做一些处理)。
#####4、之前讲spring启动原理的时候提到过后置处理器的实现会在刷新容器的时候会调用registerBeanPostProcessors(beanFactory)，找出这些后置处理器定义信息调用getBean方法(上面已经讲过bean的创建流程)进行Bean创建并初始化注册到容器中然后(保存到容器的beanPostProcessors(Map集合)中)
```
public static void registerBeanPostProcessors(
			ConfigurableListableBeanFactory beanFactory, AbstractApplicationContext applicationContext) {

		......忽略部份代码
          //    AnnotationAwareAspectJAutoProxyCreator的父类实现了Ordered接口，所以处理的逻辑在这里注册
		// Next, register the BeanPostProcessors that implement Ordered.
                // 注册实现了Ordered接口的BeanPostProcessor
		List<BeanPostProcessor> orderedPostProcessors = new ArrayList<>();
		for (String ppName : orderedPostProcessorNames) {
                        // 通过getBean创建并初始化Bean对象
			BeanPostProcessor pp = beanFactory.getBean(ppName, BeanPostProcessor.class);
			orderedPostProcessors.add(pp);
			if (pp instanceof MergedBeanDefinitionPostProcessor) {
				internalPostProcessors.add(pp);
			}
		}
		sortPostProcessors(orderedPostProcessors, beanFactory);
             //   然后保存到容器的beanPostProcessors的位置中
		registerBeanPostProcessors(beanFactory, orderedPostProcessors);
	......忽略部份代码
	}
```
**到这里AnnotationAwareAspectJAutoProxyCreator就创建完成了，它属于InstantiationAwareBeanPostProcessor类型的BeanPostProcessor**


#####spring aop注解使用AOP功能只需要编写一个切面类@Aspect，然后指定@Pointcut需要代理的包范围，再编写 @Before等通知方法，例如下面代码
```
 */
@Component
@Aspect
public class MyAspectJ {
    //抽取公共的切入点表达式
    //1、本类引用
    //2、其他的切面引用
    @Pointcut("execution(public * com.example.demo.aop..*.*(..))")
    public void pointCut(){};

    //@Before在目标方法之前切入；切入点表达式（指定在哪个方法切入）
    @Before("pointCut()")
    public void before(JoinPoint joinPoint){
        Object[] args = joinPoint.getArgs();
        System.out.println(""+joinPoint.getSignature().getName()+"运行。。。@Before:参数列表是：{"+ Arrays.asList(args)+"}");
    }

    @After("pointCut()")
    public void after(JoinPoint joinPoint){
        System.out.println(""+joinPoint.getSignature().getName()+"结束。。。@After");
    }

    //JoinPoint一定要出现在参数表的第一位
    @AfterReturning(value="pointCut()",returning="result")
    public void afterReturning(JoinPoint joinPoint,Object result){
        System.out.println(""+joinPoint.getSignature().getName()+"正常返回。。。@AfterReturning:运行结果：{"+result+"}");
    }

    @AfterThrowing(value="pointCut()",throwing="exception")
    public void afterThrowing(JoinPoint joinPoint,Exception exception){
        System.out.println(""+joinPoint.getSignature().getName()+"异常。。。异常信息：{"+exception+"}");
    }
}
```
// 需要增强的目标方法类(这个类必须在com.example.demo.aop下)
```
// vp需要增强的目标方法
package com.example.demo.aop;
@Component
public class MyAopBean {
    public int vp(){
        System.out.println("运行vp方法");
        return 1;
    }
}
```
####这里主要分析切面类(MyAspectJ)和需要增强的目标方法类(MyAopBean)的创建实例对象的过程

#MyAspectJ创建实例对象过程
MyAspectJ的实例化过程与上面分析的单实例Bean实例过程一致，不同的是开启AOP切面类(MyAspectJ)在
6.1、调用Object bean = resolveBeforeInstantiation(beanName, mbdToUse)会执行
AbstractAutoProxyCreator.postProcessBeforeInstantiation()，把MyAspectJ切面类增加到advisedBeans(this.advisedBeans.put(cacheKey, Boolean.FALSE))
```
@Nullable
	protected Object resolveBeforeInstantiation(String beanName, RootBeanDefinition mbd) {
		Object bean = null;
	......
// 因为AOP的AnnotationAwareAspectJAutoProxyCreator属于InstantiationAwareBeanPostProcessor类型的后置处理器
//所以会进行applyBeanPostProcessorsBeforeInstantiation(targetType, beanName)方法
			if (!mbd.isSynthetic() && hasInstantiationAwareBeanPostProcessors()) {
				Class<?> targetType = determineTargetType(beanName, mbd);
				if (targetType != null) {
					bean = applyBeanPostProcessorsBeforeInstantiation(targetType, beanName);
					if (bean != null) {
						bean = applyBeanPostProcessorsAfterInitialization(bean, beanName);
					}
	......
	}
```
applyBeanPostProcessorsBeforeInstantiation(targetType, beanName)方法的ibp.postProcessBeforeInstantiation(beanClass, beanName)执行的就是AbstractAutoProxyCreator.postProcessBeforeInstantiation()
```
@Override
	public Object postProcessBeforeInstantiation(Class<?> beanClass, String beanName) {
		Object cacheKey = getCacheKey(beanClass, beanName);

		if (!StringUtils.hasLength(beanName) || !this.targetSourcedBeans.contains(beanName)) {
			if (this.advisedBeans.containsKey(cacheKey)) {
				return null;
			}
// isInfrastructureClass(beanClass)会判断当前bean是否是基础类型的Advice、Pointcut、Advisor、AopInfrastructureBean或者是否是切面(@Aspect)
//如果是就会把bean添加到advisedBeans集合中
// 因为MyAspectJ属于@Aspect，所以会被添加到advisedBeans中
			if (isInfrastructureClass(beanClass) || shouldSkip(beanClass, beanName)) {
				this.advisedBeans.put(cacheKey, Boolean.FALSE);
				return null;
			}
		}
		......

		return null;
	}
```
#####切面类实例化的其他步骤与普通Bean的流程一样
#####切面类(MyAspectJ)的实例化过程不同点在于执行到了Object bean = resolveBeforeInstantiation(beanName, mbdToUse)时该Bean会被保存到advisedBeans中(保存了AOP的增强信息，后面的目标方法创建代理对象需要到这里找到增强的通知方法)
---
#目标方法类(MyAopBean)的创建过程
在6.3.4.4调用initializeBean(beanName, exposedObject, mbd)的applyBeanPostProcessorsAfterInitialization(wrappedBean, beanName)执行后置处理器的后置方法时，会执行
AbstractAutoProxyCreator.postProcessAfterInitialization()方法
```
	@Override
	public Object postProcessAfterInitialization(@Nullable Object bean, String beanName) {
		if (bean != null) {
			Object cacheKey = getCacheKey(bean.getClass(), beanName);
			if (this.earlyProxyReferences.remove(cacheKey) != bean) {
                        // 如果需要就进行包装，这个方法处理创建代理对象流程
				return wrapIfNecessary(bean, beanName, cacheKey);
			}
		}
		return bean;
	}
```
进入wrapIfNecessary(bean, beanName, cacheKey)方法
```
protected Object wrapIfNecessary(Object bean, String beanName, Object cacheKey) {
		if (StringUtils.hasLength(beanName) && this.targetSourcedBeans.contains(beanName)) {
			return bean;
		}
		if (Boolean.FALSE.equals(this.advisedBeans.get(cacheKey))) {
			return bean;
		}
		if (isInfrastructureClass(bean.getClass()) || shouldSkip(bean.getClass(), beanName)) {
			this.advisedBeans.put(cacheKey, Boolean.FALSE);
			return bean;
		}

		// 如果有当前Bean所要的增强(通知方法)，就创建代理对象。我们在MyAspectJ切面类上定义了4个增强方法(通知方法)，这里会get到，看下图
		Object[] specificInterceptors = getAdvicesAndAdvisorsForBean(bean.getClass(), beanName, null);
		// 如果获取到通知方法，就创建代理对象
                if (specificInterceptors != DO_NOT_PROXY) {
                   // 把当前目标方法类放入advisedBeans中，true表示做了增强处理
			this.advisedBeans.put(cacheKey, Boolean.TRUE);
                  // 开始创建代理对象
                  // spring会自动决定使用JdkDynamicAopProxy(config) jdk动态代理
                  //或者ObjenesisCglibAopProxy(config) cglib的动态代理来创建
			Object proxy = createProxy(
					bean.getClass(), beanName, specificInterceptors, new SingletonTargetSource(bean));
			this.proxyTypes.put(cacheKey, proxy.getClass());
                      // 返回代理对象
			return proxy;
		}

		this.advisedBeans.put(cacheKey, Boolean.FALSE);
		return bean;
	}
```
1 2 3 4是我们自己定义切面类(MyAspectJ)上的增强方法
![getAdvicesAndAdvisorsForBean获取到的通知方法](https://upload-images.jianshu.io/upload_images/14890912-70c64881470b0c1f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#####目标方法类(MyAopBean)的创建过程在6.3.4.4调用initializeBean(beanName, exposedObject, mbd)的applyBeanPostProcessorsAfterInitialization(wrappedBean, beanName)执行后置处理器的后置方法时，会执行AbstractAutoProxyCreator.postProcessAfterInitialization()方法创建代理对象并返回。
这里就把目标类的Bean(代理对象)创建好了，以后获取这个Bean时都是获取代理对象。

---
#MyAopBean创建好后，分析目标方法vp()是如何执行的(是通过责任链模式调用来执行的)
把断点打到了vp()方法，Step into进入了
org.springframework.aop.framework.CglibAopProxy.DynamicAdvisedInterceptor#intercept
说明这个intercept拦截目标方法的执行，做了一些处理
```
public Object intercept(Object proxy, Method method, Object[] args, MethodProxy methodProxy) throws Throwable {
			......
				List<Object> chain = this.advised.getInterceptorsAndDynamicInterceptionAdvice(method, targetClass);
				Object retVal;
				// 如果没有增强方法，就执行这里(这里是通过反射直接调用目标方法)
				if (chain.isEmpty() && Modifier.isPublic(method.getModifiers())) {
					Object[] argsToUse = AopProxyUtils.adaptArgumentsIfNecessary(method, args);
					retVal = methodProxy.invoke(target, argsToUse);
				}
				else {
					// chain的list有增强的方法，就创建方法的代理并执行proceed()
					retVal = new CglibMethodInvocation(proxy, target, method, args, targetClass, chain, methodProxy).proceed();
				}
				retVal = processReturnType(proxy, target, method, retVal);
				return retVal;
			}
			finally {
				if (target != null && !targetSource.isStatic()) {
					targetSource.releaseTarget(target);
				}
				if (setProxyContext) {
					// Restore old proxy.
					AopContext.setCurrentProxy(oldProxy);
				}
			}
		}
```
#####1.调用List<Object> chain = this.advised.getInterceptorsAndDynamicInterceptionAdvice(method, targetClass)获取将要执行的目标方法拦截器链(就是增强方法(通知))
 &emsp;&emsp;1.1)获取并遍历所有的Advisor(增强器),将其转为Interceptor并返回。
这里获取到5个增强，一个默认的ExposeInvocationInterceptor 和 4个自定义的aop通知，如下图
![image.png](https://upload-images.jianshu.io/upload_images/14890912-e4857d6e3720d4cf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
#####2.获取到拦截器链后调用retVal = new CglibMethodInvocation(proxy, target, method, args, targetClass, chain, methodProxy).proceed();创建CglibMethodInvocation对象并调用proceed()触发拦截器链(org.springframework.aop.framework.ReflectiveMethodInvocation#proceed)
#####3.分析proceed方法的执行
```
public Object proceed() throws Throwable {
		//  currentInterceptorIndex 默认为-1，每执行一次proceed()都会+1，没有拦截器(通知方法)或者执行最后一个拦截器执行proceed()方法时，都会执行目标方法
        //this.interceptorsAndDynamicMethodMatchers是一个list集合，存储了上面获取到的5个拦截器(通知方法)
		if (this.currentInterceptorIndex == this.interceptorsAndDynamicMethodMatchers.size() - 1) {
                        // invokeJoinpoint是执行目标方法
			return invokeJoinpoint();
		}
           // 获取++this.currentInterceptorIndex坐标的拦截器
		Object interceptorOrInterceptionAdvice =
				this.interceptorsAndDynamicMethodMatchers.get(++this.currentInterceptorIndex);
		}
            ......忽略部份代码
		else {
			// It's an interceptor, so we just invoke it: The pointcut will have
			// been evaluated statically before this object was constructed.
			return ((MethodInterceptor) interceptorOrInterceptionAdvice).invoke(this);
		}
	}
```
 &emsp;&emsp;3.1)第一次进入proceed方法，currentInterceptorIndex为-1，this.interceptorsAndDynamicMethodMatchers.get(++this.currentInterceptorIndex);获取坐标为0的通知方法(ExposeInvocationInterceptor  默认的增强器)
![第一次调用proceed](https://upload-images.jianshu.io/upload_images/14890912-b929d42832be20de.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
 &emsp;&emsp;3.2)然后走到((MethodInterceptor) interceptorOrInterceptionAdvice).invoke(this)，将增强器转成MethodInterceptor执行invoke(this)，执行的是ExposeInvocationInterceptor.invoke()，发现调用的还是mi.proceed()方法(上面传参是this，所以这方法还是ReflectiveMethodInvocation#proceed)
![ExposeInvocationInterceptor.invoke()](https://upload-images.jianshu.io/upload_images/14890912-d85d1dc67d509f04.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
 &emsp;&emsp;3.3)又回到ReflectiveMethodInvocation#proceed方法，此时currentInterceptorIndex为0，不等于(5-1)，会继续执行this.interceptorsAndDynamicMethodMatchers.get(++this.currentInterceptorIndex);获取坐标为**1**的通知方法(AspectJAfterThrowingAdvice(异常通知 @AfterThrowing))
![第二次调用proceed](https://upload-images.jianshu.io/upload_images/14890912-62ea5d6522de67e1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
 &emsp;&emsp;3.4)然后又走到((MethodInterceptor) interceptorOrInterceptionAdvice).invoke(this)，这次执行的是异常通知的invoke，AspectJAfterThrowingAdvice#invoke()，它又会执行mi.proceed()
![AspectJAfterThrowingAdvice.invoke](https://upload-images.jianshu.io/upload_images/14890912-6566093c640239b7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
 &emsp;&emsp;3.5)AspectJAfterThrowingAdvice#invoke()调用mi.proceed()回到ReflectiveMethodInvocation#proceed方法此时currentInterceptorIndex为1，不等于(5-1)，会继续执行this.interceptorsAndDynamicMethodMatchers.get(++this.currentInterceptorIndex);获取坐标为2的通知方法(AfterReturningAdviceInterceptor (返回通知 @AfterReturning 目标方法正常返回之后调用))
![第三次调用proceed](https://upload-images.jianshu.io/upload_images/14890912-46a6216b17b28364.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
 &emsp;&emsp;3.6)然后又走到((MethodInterceptor) interceptorOrInterceptionAdvice).invoke(this)，这次执行的是返回通知的invoke，AfterReturningAdviceInterceptor#invoke，它又会先执行mi.proceed()
![AfterReturningAdviceInterceptor.invoke](https://upload-images.jianshu.io/upload_images/14890912-cd0f0289397577ac.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
3.7)AfterReturningAdviceInterceptor#invoke()调用mi.proceed()回到ReflectiveMethodInvocation#proceed方法此时currentInterceptorIndex为2，不等于(5-1)，会继续执行this.interceptorsAndDynamicMethodMatchers.get(++this.currentInterceptorIndex);获取坐标为3的通知方法(AspectJAfterAdvice(后置通知 @After 在目标方法运行结束之后运行之后调用，不管是否有异常))
![第四次调用proceed](https://upload-images.jianshu.io/upload_images/14890912-a6a769f0de175183.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
3.8)然后又走到((MethodInterceptor) interceptorOrInterceptionAdvice).invoke(this)，这次执行的是后置通知的invoke，AspectJAfterAdvice#invoke，它又会先执行mi.proceed()
![AspectJAfterAdvice.invoke](https://upload-images.jianshu.io/upload_images/14890912-84e6b3921f098599.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
3.9)AspectJAfterAdvice#invoke()调用mi.proceed()回到ReflectiveMethodInvocation#proceed方法此时currentInterceptorIndex为3，不等于(5-1)，会继续执行this.interceptorsAndDynamicMethodMatchers.get(++this.currentInterceptorIndex);获取坐标为4的通知方法(MethodBeforAdivceInterceptor(前置通知 @Before 在目标方法运行之前调用))
![advice.before](https://upload-images.jianshu.io/upload_images/14890912-889c2683e7128a73.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
3.10)然后又走到((MethodInterceptor) interceptorOrInterceptionAdvice).invoke(this)，这次执行的是前置通知的invoke，MethodBeforeAdviceInterceptor#invoke，它会先调用this.advice.before(mi.getMethod(), mi.getArguments(), mi.getThis())执行前置通知的方法(@Before)
![MethodBeforAdivceInterceptor#invoke](https://upload-images.jianshu.io/upload_images/14890912-164de97664d25d85.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![前置通知](https://upload-images.jianshu.io/upload_images/14890912-48b08fa3c2943364.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
3.11)执行完前置通知的方法后，会再次调用mi.proceed()，此时currentInterceptorIndex为4，等于(5-1)，所以会执行invokeJoinpoint()(这个方法就是执行目标方法的，通过this.methodProxy.invoke(this.target, this.arguments))
![第五次调用proceed](https://upload-images.jianshu.io/upload_images/14890912-c89fe90505ee4eb4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![目标方法(打印出运行vp方法)](https://upload-images.jianshu.io/upload_images/14890912-272a266c2088fea8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
3.12)执行完ruturn invokeJoinpoint()(目标方法之后)，MethodBeforeAdviceInterceptor.invoke()就return了,然后回到调用AspectJAfterAdvice.invoke((回到上一级调用MethodBeforeAdviceInterceptor的方法))，这时会调用invokeAdviceMethod(getJoinPointMatch(), null, null)，执行后置方法(不管是否目标方法有异常都会执行到)
![AspectJAfterAdvice.invoke](https://upload-images.jianshu.io/upload_images/14890912-919fcc163d0bef75.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/14890912-9f05e414625b630f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

3.13)执行完AspectJAfterAdvice.invoke()之后，这时目标方法抛出异常了(int i = 1/0;)，AfterReturningAdviceInterceptor#invoke就不会执行this.advice.afterReturning(retVal, mi.getMethod(), mi.getArguments(), mi.getThis())了(如果正常返回就会执行this.advice.afterReturning来调用返回通知)，它会把异常抛给上一层(AspectJAfterThrowingAdvice)，这时回到
AspectJAfterThrowingAdvice#invoke(回到上一级调用AfterReturning的方法)，会调用invokeAdviceMethod(getJoinPointMatch(), null, ex);执行异常通知方法，然后throw ex抛出异常给上一层(ExposeInvocationInterceptor)
![image.png](https://upload-images.jianshu.io/upload_images/14890912-7f3c1b45169a17d7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![异常通知](https://upload-images.jianshu.io/upload_images/14890912-81e39fa1ef47719d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
3.14)执行完异常通知方法，会回到最早的ExposeInvocationInterceptor增强器方法中，执行完finally之后，就会抛出异常给jvm
![image.png](https://upload-images.jianshu.io/upload_images/14890912-a15a84996ea0510e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/14890912-cf468ba529be34ca.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
#####AOP目标方法执行原理分析完毕，使用链式调用，从坐标为0的增强器一直调到最后一个增强器，然后最后一个增强器执行完后，返回上一个增强器继续执行操作...直到返回到第一个增强器执行完成，增强了的目标方法就执行完成
#####目标方法(MyAopBean.vp)执行的流程图
![image.png](https://upload-images.jianshu.io/upload_images/14890912-f1e655eaaac81c2f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)











AOP启动总结：
spring容器启动时调用refresh()刷新容器(具体spring容器启动的流程请看[https://www.jianshu.com/writer#/notebooks/31229867/notes/54723874/preview](https://www.jianshu.com/writer#/notebooks/31229867/notes/54723874/preview)
)

* **refresh()# invokeBeanFactoryPostProcessors(beanFactory)，会将spring能感知到的(加了注解的)所有Bean的定义信息保存到容器中。（AnnotationAwareAspectJAutoProxyCreator、MyAspectJ、MyAopBean的定义信息都是这时候注册到容器的）**
* **refresh()# registerBeanPostProcessors(beanFactory)会创建并初始化后置处理器并注册到容器中，因为AnnotationAwareAspectJAutoProxyCreator属于InstantiationAwareBeanPostProcessor类型的后置处理器，所以在这时会创建AnnotationAwareAspectJAutoProxyCreator的实例注册到容器中**
* **refresh()# finishBeanFactoryInitialization(beanFactory) 会初始化(创建Bean的实例)所有剩下的单实例Bean，MyAspectJ类和MyAopBean类在这时会创建Bean实例并保存到容器中**