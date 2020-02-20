---
layout: post
title: "@SpringBootApplication注解分析"
date: 2019-07-20
tags: java
---

@SpringBootApplication

![image](https://upload-images.jianshu.io/upload_images/14890912-a31273ee4fb3b0f9?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我们可以看到@SpringBootApplication主要由以下三个注解组合。

####@SpringBootConfiguration

#####@EnableAutoConfiguration

#####@ComponentScan
-------------------------------------------------------------------
#@SpringBootConfiguration注解(指定springboot的主配置类(启动类))
>**@SpringBootConfiguration**标注在某个类上，表示这是一个springboot的配置类
>一层层点到最后，发现其实就是我们熟悉的spring的framework包下的注解@Configuartion
>@Configuartion:配置类上用这个注册-配置类的作用可以充当配置文件，配置类也是容器中的一个组件;@Component
```
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Configuration
public @interface SpringBootConfiguration {
}
```
@Configuartion是@Component的派生注解
@Service、@Repository、@Configuartion等注解都是@Component的派生，可以理解成@Component是父类，@Configuartion是子类。
其实这些注解的作用都是，就是spring扫描类时会对加上这些注解的类自动装配到Spring容器中进行管理。只是注解分多个名称来标注，阅读代码时更好的理解代码的作用，我们可以看看
ClassPathScanningCandidateComponentProvider类
![image](https://upload-images.jianshu.io/upload_images/14890912-d08db2ed762bde8d?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image](https://upload-images.jianshu.io/upload_images/14890912-0180bce0e3cd4440?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
扫描的就是@Component注解，和它派生出来的注解(子类注解)
与xml中配置<bean id="" class=""/>作用相同
-----
#@EnableAutoConfiguration 开启自动配置功能springboot启动类所在包及子包的组件类和自动配置类(META-INF/spring.factories文件下的类)，并将其初始化注入到IOC容器)
```
@AutoConfigurationPackage
@Import(EnableAutoConfigurationImportSelector.class)
public @interface EnableAutoConfiguration {
```
>一.@**AutoConfigurationPackage**：自动配置包，这注解里面通过@Import(AutoConfigurationPackages.Registrar.class)注入了Registrar类，springboot启动时这个类的**registerBeanDefinitions**方法会扫描获取到springboot启动类所在的包及其子包下所有的类,将扫描到的所有组件(加了@Component等spring注解的类)注入spring的IOC容器中
```
static class Registrar implements ImportBeanDefinitionRegistrar, DeterminableImports {

		@Override
		public void registerBeanDefinitions(AnnotationMetadata metadata,
				BeanDefinitionRegistry registry) {
			register(registry, new PackageImport(metadata).getPackageName());
		}
```
>二.@**Import**(EnableAutoConfigurationImportSelector.class)；
>注入的EnableAutoConfigurationImportSelector类，这个Selector选择器类会给容器中导入非常多的自动配置类（xxxAutoConfiguration）
>这个类会调用SpringFactoriesLoader.loadFactoryNames(EnableAutoConfiguration.class,classLoader)方法
>Spring Boot在启动的时候从类路径下的META-INF/spring.factories中获取EnableAutoConfiguration指定的值，将这些值作为自动配置类导入到容器中，自动配置类就生效，帮我们进行自动配置工作(类似于java的spi)

**自动配置：比如之前启动WebMvc需要在xml上写很多配置，现在只要引入springboot的starter-web包,springboot启动时会将spring-boot-autoconfigure-2.1.4.RELEASE.jar包下的META-INF\spring.factories配置文件org.springframework.boot.autoconfigure.EnableAutoConfiguration为key的类加载到IOC容器中，其中就有启动WebMvc需要配置的自动配置类WebMvcAutoConfiguration，这个类初始化就会帮我们配置好webMVC需要的配置**
**又比如启动springboot web时不配置server.port，默认的端口号为8080，这也是通过自动配置类初始化的**

#@ComponentScan注解
>指定要扫描的包及其子包下的类，默认扫描当前类的同级包及其子包，作用：比如某个类上有@Component，还需要@ComponentScan注解来指定扫描这个包的类，spring才会去处理这个类上的注解
>xml的<context:component-scan base-package="" />作用相同