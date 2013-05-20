---
layout: post
title: "mobile client service 架构"
date: 2013-05-20 21:07
comments: true
categories: 
---
## 架构图
-------
![Alt text](https://raw.github.com/chunlong/chunlong.github.com/master/images/mcs_overall_runtime.png)

##  结构理解

*	mcs是手机端的入口,调用各种后台服务,处于某种考虑 在后台服务和mcs之间加了一层wfs.wfs本身是一个 ice节点.ice是一个rpc框架.wfs里面封装的对后台的服务调用包括好几大类
	基本都是一些rpc调用或者是直接的dao调用.之所以要加入这样一层wfs,是因为不希望这些远程调用服务启动的失败或者dao查询过大直接搞死mcs.当然由于后期代码越来越多,参与者的增加,这个约定没有一直执行下去,也有mcs直接连后台的服务.所以mcs里面也有直接调用rpc和dao的后台服务.
	
## 模块依赖关系
-----------
![Alt text](https://raw.github.com/chunlong/chunlong.github.com/master/images/mcs-dependency.png)

*	mcs采用Srping MVC实现。全局只存在一个MVC的Controller实例RestServerController（继承关系 MethodController-->RestServerController-->AbstractRestController)。Spring的MVC Dispatcher将所有请求都发送的到此Controller后，再有mcs自行实现的method2Class mapping完成从“method名称”到某个具体ApiCommand的映射，并最终完成执行。
	ApiCommand的基类AbstractApiCommand中实现了类似于拦截器的功能以完成如校验等功能。
	
*	RestServerController 主要工作,接口名称到具体接口执行类的映射 一个map结构,依据几口名称获取对应的接口执行类;登陆校验;用户状态校验;

*	AbstractRestController 主要工作,验证基本的参数是否完全,包括sig(唯一标示一个请求,缓存结构),app信息,sessionKey等; validateExtRequiredParams方法 (在RestServerController 实现)又做了一遍登陆校验..不明白.

*	validateRequestFrequency 方法 根据sig保存 查询结果,5s之内有重复查询进来 则直接返回结果不在查询.
	
*	全文就只有这几个controller ,具体的接口实现在command里面,所有command继承自AbstractApiCommand(基本是一个废弃的抽象类)继承自ApiCommandSupport实现了IApiCommand,在RestServerController 里获取正确的command子类实例,用父类IApiCommend的引用持有该bean,根据多态原理调用execute

*	ApiCommandSupport实现了IApiCommand的惟一的方法execute,该方法是所有请求的入口.在该方法内部调用execute--->executeMethod(ApiCommandSupport)--->executeWithStringParamsOnly(ApiCommandSupport)---->onExecute(根据多态调用具体command接口的实现) 
	在RestServerController 里获取正确的command子类实例,用父类IApiCommend的引用持有该bean,根据多态原理调用execute
	对应的实现方法.
	
*   总结来说就是 在 RestServerController方法 handleApiRequest里,IApiCommand (持有具体commandA的实例)调用execute (实际调用ApiCommandSupport的execute 因为commandA并没有实现这个方法,所以这个方法成了所有接口调用的入口),最终根据多态调用commandA
    的onExecute方法.

*   先写这么多.说实话,这个东西真没啥....	
	
	


