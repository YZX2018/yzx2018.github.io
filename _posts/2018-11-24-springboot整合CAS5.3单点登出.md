---
layout: post
title: "springboot整合CAS5.3单点登出"
date: 2018-11-24
tags: java
---

上编讲到[springbott整合cas5.3单点登录](https://www.jianshu.com/p/f2facc4d1c3a)

关于单点登出
参考官网地址 
https://apereo.github.io/cas/5.3.x/installation/Configuration-Properties.html#logout 
官网共给出了以下几个属性值：

#配置单点登出
#配置允许登出后跳转到指定页面
cas.logout.followServiceRedirects=false
#跳转到指定页面需要的参数名为 service
cas.logout.redirectParameter=service
#登出后需要跳转到的地址,如果配置该参数,service将无效。
cas.logout.redirectUrl=https://www.taobao.com
#在退出时是否需要 确认退出提示   true弹出确认提示框  false直接退出
cas.logout.confirmLogout=true
#是否移除子系统的票据
cas.logout.removeDescendantTickets=true
#禁用单点登出,默认是false不禁止
#cas.slo.disabled=true
#默认异步通知客户端,清除session
#cas.slo.asynchronous=true


cas 默认登出后默认会跳转到CASServer的登出页，若想跳转到其它资源，可在/logout的URL后面加上service=jumpurl，例如：https://server.cas.com:8443/cas/logout?service=https://www.github.com 
但默认servcie跳转不会生效，需要在 cas服务端的application.properties添加cas.logout.followServiceRedirects=true 
这个参数也不一定非要叫 service, 可以通过cas.logout.redirectParameter 来修改它。 
另外,默认退出的时候没有任何提示,直接就退出了,若想要有弹出提示,需要添加as.logout.confirmLogout=true 
再另外,有一个cas.logout.redirectUrl的属性,可以配置默认登出之后跳转到的连接,若 配置该属性,service参数将无效。就算传了service参数,也是走的该页面，所以我们不需要配置此参数。 
如果配置了cas.slo.disabled=true 将禁用单点登出。调用登出将无效。

服务端配置
application.properties添加以下属性
#配置允许登出后跳转到指定页面
cas.logout.followServiceRedirects=true
#跳转到指定页面需要的参数名为 service
cas.logout.redirectParameter=service
#在退出时是否需要 确认一下  true确认 false直接退出
cas.logout.confirmLogout=true
#是否移除子系统的票据
cas.logout.removeDescendantTickets=true