---
layout: post
title: "springboot启动类springApplication与注解解析"
date: 2018-07-23
tags: java
---

springboot的main方法3种启动方式

第一种
![image.png](https://upload-images.jianshu.io/upload_images/14890912-e4516f4e4f18b86c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

第二种
![image.png](https://upload-images.jianshu.io/upload_images/14890912-e215f25a79918ff9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/14890912-7e75bd5cb7281695.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
可以看到tomcat的启动端口为54155

第三种，使用bulider模式
![image.png](https://upload-images.jianshu.io/upload_images/14890912-ee44202aaa070bb3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

@SpringBootApplication注解和springApplication.run方法运行main方法来启动spring容器或者tomcat容器。那么springboot启动的原理是什么?

以上三种方法都是使用springApplication类的run方法来启动的

![image](https://upload-images.jianshu.io/upload_images/14890912-d2381c6ff36744bb?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

springApplication.run方法返回的是

ConfigurableApplicationContext接口，它是springframework的注解，启动打印一下它的实现类的谁?

![image](https://upload-images.jianshu.io/upload_images/14890912-3aaf32581b6810fc?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这个实现类是AnnotationConfigServletWebServerApplicationContext

用中式英文翻译理解成是一个注解配置的ServletWeb的服务application上下文，就是使用注解配置的方式启动web服务。
![image](https://upload-images.jianshu.io/upload_images/14890912-9033992b92f41435?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

从run方法的源码可以看出，是先创建一个ApplicationContext(刚才启动得到是的

AnnotationConfigServletWebServerApplicationContext)

再进行注册和启动容器上下文的

既然是通过这个类来实现启动的，底层用的也是springframework注解的启动方式，我们可以直接通过这个实现类来启动web服务

![image](https://upload-images.jianshu.io/upload_images/14890912-75767b08e76a785f?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

进行运行main方法，

![image](https://upload-images.jianshu.io/upload_images/14890912-01fe254062568813?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

启动web容器成功。这个其实还需要依赖@SpringBootApplication里面的

@EnableAutoConfiguration注解，这个注解能将初始化web容器的类加载并注册到spring容量中

为什么是实现类是一个ServletWeb的类?是因为我在maven依赖下加了springboot的web包，

![image](https://upload-images.jianshu.io/upload_images/14890912-cca91aade37c446c?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

WebApplicationType.deduceFromClasspath()方法里面

![image](https://upload-images.jianshu.io/upload_images/14890912-741e529c263f86cc?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

启动只会判断使用哪个容器，加了web包会使用默认的tomcat容

![image](https://upload-images.jianshu.io/upload_images/14890912-5c3bf57824cf3dcf?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

所以启动的就是web容器。

如果我把这个包去掉，或者用springboot的方法设置成不启动web，只启动spring容器，看看它是实现类是哪个？

![image](https://upload-images.jianshu.io/upload_images/14890912-0bbcb75d9be37a35?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![image](https://upload-images.jianshu.io/upload_images/14890912-5b85a6b82f39c417?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

AnnotationConfigApplicationContext类，这个spring的一个注解方式启动的类，可以结合@ComponentScan和@Configuration注解来启动spring IOC容器

使用注解方式启动spring IOC容器的方式

![image](https://upload-images.jianshu.io/upload_images/14890912-e44225f847d2e016?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我们创建一个Hello类，看看能不能通过getBean方式得到实例

![image](https://upload-images.jianshu.io/upload_images/14890912-90e672ec9d464851?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

启动容器，看到日志打印了hello方法执行的值，说明启动spring容器成功

![image](https://upload-images.jianshu.io/upload_images/14890912-dbcff6010134f281?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这种方式启动spring与 new ClassPathXmlApplicationContext("..xml")类通过xml配置文件启动spring的方式一样。

我们来看看@SpringBootApplication注解
@SpringBootApplication

![](https://upload-images.jianshu.io/upload_images/14890912-a31273ee4fb3b0f9?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我们可以看到@SpringBootApplication主要由

<pre style="margin: 0px; padding: 0px; max-width: 100%; box-sizing: border-box !important; overflow-wrap: break-word !important; font-style: normal; font-variant-ligatures: normal; font-variant-caps: normal; font-weight: 400; letter-spacing: 0.544px; orphans: 2; text-align: justify; text-indent: 0px; text-transform: none; widows: 2; word-spacing: 0px; -webkit-text-stroke-width: 0px; text-decoration-style: initial; text-decoration-color: initial; background-color: rgb(255, 255, 255); color: rgb(0, 0, 0); font-family: Consolas; font-size: 12pt;">@SpringBootConfiguration</pre>
<pre style="margin: 0px; padding: 0px; max-width: 100%; box-sizing: border-box !important; overflow-wrap: break-word !important; font-style: normal; font-variant-ligatures: normal; font-variant-caps: normal; font-weight: 400; letter-spacing: 0.544px; orphans: 2; text-align: justify; text-indent: 0px; text-transform: none; widows: 2; word-spacing: 0px; -webkit-text-stroke-width: 0px; text-decoration-style: initial; text-decoration-color: initial; background-color: rgb(255, 255, 255); color: rgb(0, 0, 0); font-family: Consolas; font-size: 12pt;">@EnableAutoConfiguration</pre>
@ComponentScan

三个注解组合。

那么这三个注解都做了什么？

@SpringBootConfiguration注解，一层层点到最后，发现其实就是我们熟悉的spring的framework包下的注解@Configuartion

![image](https://upload-images.jianshu.io/upload_images/14890912-c152673a2bdc7f17?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

@Configuartion是@Component的派生注解，如果不熟识的话，我们可以看看官网解析

[图片上传中...(image-dfc8e-1567504392180-2)]

@Service、@Repository、@Configuartion等注解都是@Component的派生，可以理解成@Component是父类，@Configuartion是子类。

其实这些注解的作用都是，就是spring扫描类时会对加上这些注解的类自动装配到Spring容器中进行管理。只是注解分多个名称来标注，阅读代码时更好的理解代码的作用，我们可以看看

ClassPathScanningCandidateComponentProvider类

![image](https://upload-images.jianshu.io/upload_images/14890912-d08db2ed762bde8d?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![image](https://upload-images.jianshu.io/upload_images/14890912-0180bce0e3cd4440?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

扫描的就是@Component注解，和它派生出来的注解(子类注解)

与xml中配置<bean id="" class=""/>作用相同

@EnableAutoConfiguration注解，能够激活spring spi，会读取所有jar包下

META-INF/spring.factories文件的配置的类,在spring启动时，Spring容器会对这些配置类进行处理。(类似于java的spi) ,spring.factories所有类都会被spring加载(不管是否需要用到)并注册到spring容器中，例如需要启动web容器如tomcat时，就需要用到

ServletWebServerFactoryAutoConfiguration类，那么这个类就需要注册到spring容器中进行初始化，通过spring.factories扫描到这个类

![image](https://upload-images.jianshu.io/upload_images/14890912-f4db4d717cb24d35?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

并且激活自动装配的以@Enable开头注解 如：

@EnableAspectJAutoProxy

@EnableAsync

@ComponentScan注解

指定要扫描的包及其子包下的类，默认扫描当前类的同级包及其子包，作用：比如某个类上有@Component，还需要@ComponentScan注解来指定扫描这个包的类，spring才会去处理这个类上的注解

与xml的<context:component-scan base-package="" />作用相同

springApplication.run方法主要是对META-INF/spring.factories的类进行加载，加载spring的监听器。选择容器并对容器初始化，进行组件的扫描，加载和注册，最后返回容器的上下文。

springboot选择容器时有3种类型容器判断

   WebApplicationType.NONE : 非 Web 类型

   WebApplicationType.REACTIVE : Spring WebFlux

WebApplicationType.SERVLET : Spring MVC(ServletWeb容器(tomcat jetty等))

springboot的注解基本都是对springframework注解的封装，

@ComponentScan是指定要扫描的包

@SpringBootConfiguration是注解方式将bean注册到spring容器到

@EnableAutoConfiguration可以理解成是激活spring spi，容器启动过程，如果想要将bean注册到spring容器中，必须要扫描到类的路径，但是springboot默认是扫描main方法的同及包，不会去扫描jar包，想要初始化jar包下的bean，就要用到这个spring spi机制，会读取所有jar包下META-INF/spring.factories配置文件，并加载配置文件上的类，注册到spring容器中。