---
layout: post
title: "spring解决单实例bean的循环依赖"
date: 2019-07-16
tags: java
---

singletonObjects一级缓存，存放完整的对象

earlySingletonObjects 二级缓存，存放不完成的对象(只实例化对象，并未赋值)
singletonFactories 三级缓存，存放不完成的对象(只实例化对象，并未赋值)

二级缓存存放的就是三级缓存的对象。如果在singletonFactories找到对应的对象，会把它存放到earlySingletonObjects中，并删除singletonFactories中的对象。
为什么不直接放入二级缓存?
(是为了做扩展)


A依赖B   B依赖A
创建A对象时
1.首先从容器缓存查询(先查询一级缓存，如果一级缓存不存在，并且A对象被标记为正在创建，就会查二、三级缓存)是否有A对象
2.缓存中查询不到A对象，开始实例化A对象(此时并未给属性赋值)，然后将A实例(不完整对象)存放到singletonFactories(三级缓存中)
org.springframework.beans.factory.support.DefaultSingletonBeanRegistry#addSingletonFactory
3.然后对A对象属性进行赋值
org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#populateBean
4.A对象属性中依赖了B对象，当给A对象赋值B时，会从容器中找B对象，此时又会走1 2 3步骤
5.首先从容器缓存查询(先查询一级缓存，如果一级缓存不存在，并且B对象被标记为正在创建，就会查二、三级缓存)是否有B对象
6.容器缓存中查询不到A对象，开始实例化B对象(此时并未给属性赋值)，然后将B实例(不完整对象)存放到singletonFactories(三级缓存中)
7.然后对B对象属性进行赋值
org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#populateBean
8.B对象属性中依赖了A对象,当给B对象赋值A时，会从容器中找A对象，此时的A对象已经被标记为正在创建并存放到三级缓存中，所以可以找到A对象的实例，然后B对象赋值时会引用这个A实例(此时的A实例是不完整的对象)
9.给B赋值完后，此时B对象是一个完成的对象，B对象初始化完成(创建实例并赋值完成)后会存到一级缓存中，并删除二、三级缓存保存的B对象
10.此时又回到A对象的赋值，找到B对象引用，A对象成功给B属性赋值
11.A对象赋值完成后,A对象初始化完成(创建实例并赋值完成)后会存到一级缓存中，并删除二、三级缓存保存的A对象
12.因为B对象的A属性是引用了A对象的地址，此时A对象也赋值完成了，所以B对象引用的A对象也变成了完整的对象(单例对象，引用地址是同一份)
上面都是指A和B单例对象

如果把A和B都设置成spoce("prototype")多例，就无法解决循环依赖问题，因为spring创建对象时只缓存单例对象
  只能不使用spring的依赖注入，通过set方法手动赋值

如果把A对象构造方法引用B，B对象构造方法引用A，也是无法解决循环依赖问题，因为两个对象都无法实例化，更不用说赋值了