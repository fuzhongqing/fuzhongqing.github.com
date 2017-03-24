---
title: 在Android中尝试dagger2
layout: post
guid: urn:uuid:b8dfd7b5-4c18-4699-9c64-60a7958170d1
tags:
  - android
  - dagger2
---
<br>

## 概览
许多Android app有错综复杂的依赖关系，一个功能模块依赖另一个稍微底层的功能模块，
而这个底层的模块往往又依赖于其他的模块。比如：Twitter API客户端使用Retrofit网络库。 
要使用此库，可能还需要添加解析库，如Gson，另外，实现认证或高速缓存的类可能需要依赖
Shared preference 或其他常见存储类，这样，如果需要使用Twitter API需要首先实例化整个依赖链上必须的类。

Dagger2 的目的之一就是为解决此类问题。官方给Dagger2的定义是：

> Dagger 2 is a compile-time evolution approach to dependency injection. 
Taking the approach started in Dagger 1.x to its ultimate conclusion, 
Dagger 2.x eliminates all reflection, 
and improves code clarity by removing the traditional ObjectGraph/Injector 
in favor of user-specified `@Component` interfaces.

首先，在使用Dagger2之前，应该了解先关于 `依赖注入 dependency injection`的概念,可以在[这个博客](http://antonioleiva.com/dependency-injection-android-dagger-part-1/)
中了解。
简单地说，我们平时的应用场景中常常出现这种情况：
![ioc_0](/media/files/dagger2/ioc_a.gif)

结合上面Twitter的例子，如果放纵这种依赖的产生，出现的大量的依赖使得结构不够清晰[?]
![ioc_1](/media/files/dagger2/ioc_b.gif)

所以依赖注入的解决方案是这样的
![ioc_2](/media/files/dagger2/ioc_c.gif)
简单说就是把所有的依赖统一管理，注入到需要他的类里面的

## 关于 JSR-330

Dagger2中使用到了JSR-330,这是一个注解的集合，他的主要作用是

## 优势
这里有一些为什么使用`dependency injection`去解决模块间的依赖关系的原因，以及在Android平台下
使用`Dagger2`框架的原因。

首先我们应该了解使用 Di的好处:

+ 由于我们的依赖都交由第三方管理，所以我们可以轻易的**重用这些组件**
+ 当将抽象对象作为协作者注入时，**我们可以只改变对象对接口的实现，而不必在我们的代码库中做很多变化**，因为对象的实例驻留在一个孤立的,解耦的地方。
+ Di可以轻易的注入一个组件当中,所以也可以注入到Mock implementations中，这样我们可以**轻易地对每个模块做测试**


## 参考
- [tasting-dagger-2-on-android](https://fernandocejas.com/2015/04/11/tasting-dagger-2-on-android/)
- [dagger2 Github](https://github.com/google/dagger)
- [Dependency Injection with Dagger 2](https://github.com/codepath/android_guides/wiki/Dependency-Injection-with-Dagger-2)