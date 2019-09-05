---
layout: post
title: "使用Guava retryer优雅的实现接口重调机制"
date: 2017-11-29
tags: java
---

所有的中间件在设计上,都做了"重试".那问题来了,在我们业务代码中,万一有重试的需求,我们如何优雅实现?是try一下,catch了我再调用,那一旦我要重试个四五次,每次重试间隔是10秒,那代码讲写得不堪入目.技多不压身,给大家分享一下优雅实现的方式
https://www.cnblogs.com/jianzh5/p/6651799.html

