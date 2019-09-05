---
layout: post
title: "JS对象和Java对象转成JSON"
date: 2016-06-20 
tags: java 
---

  一.Json对象与Json字符串的转化


1.jQuery插件支持的转换方式：


　　$.parseJSON( jsonstr ); //jQuery.parseJSON(jsonstr),可以将json字符串转换成json对象


2.浏览器支持的转换方式(Firefox，chrome，opera，safari，ie9，ie8)等浏览器：


　　JSON.stringify(obj)将js对象转为JSON。JSON.parse(string)将字符串转为JSON格式；


var a={"name":"tom","sex":"男","age":"24"}; 
var b='{"name":"Mike","sex":"女","age":"29"}'; 
var aToStr=JSON.stringify(a); 
var bToObj=JSON.parse(b); 
alert(typeof(aToStr));  //string 
alert(typeof(bToObj));//object


 


3.Javascript支持的转换方式： 
eval('(' + jsonstr + ')'); //可以将json字符串转换成json对象,注意需要在json字符外包裹一对小括号 
注：ie8(兼容模式),ie7和ie6也可以使用eval()将字符串转为JSON对象，但不推荐这些方式，这种方式不安全eval会执行json串中的表达式。


4.JSON官方的转换方式： 
http://www.json.org/提供了一个json.js,这样ie8(兼容模式),ie7和ie6就可以支持JSON对象以及其stringify()和parse()方法； 
可以在https://github.com/douglascrockford/JSON-js上获取到这个js，一般现在用json2.js。


 


二、JSON 字符串 与 java 对象的转换


\1. 把java 对象列表转换为json对象数组，并转为字符串


    JSONArray array = JSONArray.fromObject(list);
    String jsonstr = array.toString();


 


\2. 把java对象转换成json对象，并转化为字符串


  JSONObject object = JSONObject.fromObject(user);
  Log4jInit.ysulogger.debug(object.toString());


3.把JSON字符串转换为JAVA 对象数组
  JSONArray json = JSONArray.fromObject(userStr);//userStr是json字符串
  List<User> users= (List<User>)JSONArray.toCollection(json, User.class);

4.把JSON字符串转换为JAVA 对象
　　JSONObject jsonobject = JSONObject.fromObject(jsonStr);
　　User user= (User)JSONObject.toBean(object,User.class);