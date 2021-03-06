---
layout:     post
title:      51信用卡跨平台架构演进 
date:       2020-05-04
author:     robertyzli
header-img: Resources/Posts/liyizhen_blog_cross_platform_bg.jpg
catalog: true
tags:
    - Hybrid/Weex/Swift
    - 跨平台
    - 架构演进 
---

<style>
  table {
      width: 100%; /*表格宽度*/
      border-collapse: collapse; /*使用单一线条的边框*/
      empty-cells: show; /*单元格无内容依旧绘制边框*/
  }
	
  table th,td {
    height: 35px; /*统一每一行的默认高度*/
  }
	
  table th {
      font-weight: bold; /*加粗*/
      text-align: center !important; /*内容居中，加上 !important 避免被 Markdown 样式覆盖*/
      white-space: nowrap; /*表头内容强制在一行显示*/
      font-size:16px;font-family:"Times New Roman", Times, serif !important;
      background: #ECF2F9; /*背景色*/
      color:#0F7290;
  }
  
   table td {
      text-align: center !important; /*内容居中，加上 !important 避免被 Markdown 样式覆盖*/
      font-size:14px;font-family:"Times New Roman", Times, serif !important;
  }
	
  /* 隔行变色 */
  table tbody tr:nth-child(2n) {
      background: #F4F7FB; 
  }
  /* 悬浮变色 */
  /*table tr:hover {
      background: #B2B2B2; 
  }*/
	
  /* 首列不换行 */
  table td:nth-child(1) {
      white-space: nowrap; 
  }
  /* 指定列宽度 */
  /*table th:nth-of-type(2) {
    	width: 200px;
     white-space: nowrap;
  }*/
  </style>  
   
> 本文主要介绍51信用卡跨平台架构的演进情况，对混合开发模式存在的一些共性问题给出解决方案。   

&ensp;&ensp;&ensp;&ensp;自2015年Facebook开源首个跨平台UI框架<font style="color:#0F7290">ReactNative(RN)</font>，到2016年阿里开源<font style="color:#0F7290">Weex</font>，2017年Google推出<font style="color:#0F7290">Flutter</font>跨平台UI框架。跨平台解决方案在移动端正处于蓬勃发展的状态，而目前大部分企业基本都是原生与跨平台的混合开发模式，无论是业务需要还是移动平台技术的差异特性，都很难做到完全跨平台。然而，混合开发模式下的开发会面临更多的问题。

&ensp;&ensp;&ensp;&ensp;2017年我进入51信用卡，当时客户端与前端使用的是Hybrid混合开发模式。2017年年中开始负责Hybrid，2018年年中开始接手Weex负责整个跨平台（Hybrid与Weex），主要负责解决Weex框架源码问题以及基础能力建设。**<font style="color:#FF005D">2018年跨平台支持团队在100人+</font>**（前端团队80人左右，客户端团队50人左右），支持4个App团队（**<font style="color:#FF005D">51信用卡App注册用户量7000多W</font>**）。在51的三年基本上就是处于不断填坑状态，开始接触WebKit的时候Hybrid本身并没有架构的概念，很多代码基本都是以功能逻辑融合在一起，每当新增需求都很困难担心改动点是否全面有遗漏；而当Weex跨平台接入之后又依赖Hybrid，出现了运行时安全、耦合依赖等问题，本文会详细阐述遇到的实际问题以及是如何优化架构解决问题。

###  一.目标体现
**<font style="color:#FF005D">总体目标：</font>**跨平台层作为前端与Native的中间混合层，主要目标是为Hybrid/Weex（或者其他跨平台方案）**<font style="color:#0F7290">提供更好的服务能力或者互动能力</font>**（比如获取地理位置信息或者设置容器导航标题与按钮等等）。     

**<font style="color:#FF005D">目标特点</font>**：**<font style="color:#0F7290">高扩展、高复用、高内聚低耦合、高性能（待优化）、方便易用、稳定安全</font>**  

- <font style="color:#0F7290">1.高扩展</font> 
比如灵活扩展Weex内部的WebView调用PG方法，以及Weex调用PG方法、或者业务自定义浏览器；或者业务侧覆盖基础侧提供的默认PG方法。
- <font style="color:#0F7290">2.高复用</font> 
比如Hybrid沉淀的PG方法，Weex等其他跨平台方案可以无缝支持。
- <font style="color:#0F7290">3.高内聚低耦合</font> 
比如对webview容器修改的PG方法以及相关逻辑都放在Hybrid PG分类容器中，不需要修改主体Hybrid容器从而导致主体容器代码量冗杂庞大。
- <font style="color:#0F7290">4.方便易用</font> 
比如业界基本都是采用Module的方式业务若需扩展自定义需要创建Module类比较笨重，而我们的PG则只是一个简单的函数方法。
- <font style="color:#0F7290">5.稳定安全</font> 
PG监控、访问权限控制。
- <font style="color:#0F7290">6.高性能（待优化）</font> 
线程模型优化（任务数目不可控、任务拥塞）、任务优先级队列、接口单点优化等。

###   二.早期架构  
####  （一）Hybrid架构   
<table>
    <thead>
        <tr>
            <th>Hybrid架构</th>
            <th>通信逻辑架构</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td><img src="/Resources/Posts/liyizhen_blog_cross_platform_old.png"/></td>
            <td><img src="/Resources/Posts/liyizhen_blog_cross_platform_old_pg.png"/></td>
        </tr>
    </tbody>
</table>

####  （二）跨平台架构
<table>
    <thead>
        <tr>
            <th>总体架构设计</th>
            <th>通信逻辑架构</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td><img src="/Resources/Posts/liyizhen_blog_cross_platform_weex_hybrid.png"/></td>
            <td><img src="/Resources/Posts/liyizhen_blog_cross_platform_weex_hybrid_pg.png"/></td>
        </tr>
    </tbody>
</table>

#### （三）面临问题   
- <font style="color:#0F7290">1.功能模块划分不清晰</font> 
Hybrid功能点主要有双核浏览器、Cookie管理、WebUA管理、头部扩展（标题、按钮等样式自定义扩展很困难以及不满足沉浸式头部等等）、加载提示（进度条、菊花，并且双WebView进度设置方式不同）、PG（H5与Native交互API，比如调用原生getLocation获取地理位置信息）执行核心逻辑、URLScheme跳转（比如跳转到AppStore、拉起微信支付、打电话等等）等等。这些功能点都放在WebAppKit基础库中，并且没有划分清晰的模块逻辑都融合在一起导致后续扩展离线优化、注入脚本（在dom创建时机注入，注入过早导致无法执行，过晚可能错过时机）、通信（H5页面之间通信、H5与Native之间通信）、WebGL Crash处理等功能非常的困难。
- <font style="color:#0F7290">2.基础库依赖问题</font> 
由于PG核心逻辑高耦合Hybrid导致跨平台方案Weex必须依赖Hybrid（因为Weex需要扩展支持PG方法调用），任何需要扩展PG的基础库（比如登录、埋点等）、业务都会依赖Hybrid。高度耦合依赖导致每次发版效率很低，基础库的依赖呈现网状状态。
- <font style="color:#0F7290">3.视图容器逻辑不清晰</font> 
比如，加载逻辑（加载进度条或者菊花未抽象UI组件）、WebView配置逻辑（双核浏览器逻辑耦合在一起）、导航控制逻辑（标题更新、返回按钮、分享按钮等逻辑，并且缺乏灵活扩展）、WebUA等等都放在Web视图容器中，各种逻辑通过不断的添加属性作为条件判断来区分逻辑，当时该文件有1000多行的代码，这种逻辑聚集在一起犹如滚雪球越滚越大逻辑越来越复杂，一个技术小点的改动都需要反复确认风险点、改动点覆盖、影响范围等等。<font style="color:#22797D">虽然，每个小技术点不是技术难题，但是当这些小点杂乱无章的聚集在一起，只是满足当时业务需求而未考虑后期的演进（防止过渡设计）时，久而久之就会演变成</font>**<font style="color:#FF005D">麻乱系统</font>**<font style="color:#22797D">导致变更成本越来越高（</font>**<font style="color:#FF005D">比如苹果对UIWebView废弃，若逻辑包袱很重再加上WKWebView存在的坑点，推动适配的成本将会变的很高</font>**<font style="color:#22797D">），甚至最后导致不可维护，也许一个核心的模块最后成为团队都不想负责的一个模块</font> 。
- <font style="color:#0F7290">4.接口不可控问题</font> 
基础hybrid在视图中直接暴露WebView浏览器，从而子类可以在任意时机操作WebView，引入不可控因素。比如业务继承基础暴露WebView，业务自定义加载url时机，导致基础在处理h5加载时机（比如监控或者处理url公共参数）时很容易遗漏等等，需求变更或者做技术演进风险与成本都很高。
- <font style="color:#0F7290">5.线程安全问题</font> 
PG底层逻辑层默认使用子线程，需要读取上层容器URL，直接调用WebView获取URL接口等等，潜在了很多不安全因素。
- <font style="color:#0F7290">6.运行时安全问题</font> 
在PG执行逻辑中Weex视图容器对象直接转换为Web视图容器使用，比如获取容器当前加载的URL，则Weex容器必须要提供一个与Web容器一样的获取URL的接口否则出现Crash，同理只要Web容器增加属性或者方法Weex也要同样增加，潜在了很严重的不安全因素。
- <font style="color:#0F7290">7.PG所属范围问题</font> 
比如打开A页面再快速打开B页面，然后快速返回，若在B页面调用了设置容器的方法那么B页面返回后该PG还会继续执行，则最后导致在A页面上生效。
- <font style="color:#0F7290">8.PG一致性问题</font> 
因在PG方法中可以自由操作视图容器等（比如设置头部标题）沉淀的hybrid PG方法无法在Weex上复用（因为更新头部标题的PG方法内部访问的是Web容器而不是Weex视图容器），有很多类似的API不能做到一致性从而导致Weex在一定程度上很难做到可降级H5页面。
- <font style="color:#0F7290">9.可扩展性差</font> 
因PG核心逻辑强耦合Web视图容器，若让WebView支持PG方法调用则必须依赖Web视图容器。比如，业务同学单独使用WebView做运营广告位，只能使用野路子的方式，操作WebController中的View。并且，Weex组件WebView无法扩展使用PG方法。 
- <font style="color:#0F7290">10.稳定安全问题</font> 
缺少对PG使用监控手段（比如PG调用、白名单、脚本、性能监控等等），以及敏感PG（比如获取用户信息GetUserInfo返回token）的权限访问。
- <font style="color:#0F7290">11.PG历史包袱问题</font> 
平台兼容性问题（同一个方法在iOS、Android上表现不一致）、缺少规范（比如参数、返回值、命名、版本等等）、实现不一致（比如管家、51人品都需要存储数据到本地的接口SaveValue。iOS在每个App上单独实现功能大致表现一致，管家使用YYModel存储，而人品贷App使用UserDefault存储；而Android则在基础库中实现，使用xml方式存储）、文档描述与实现不一致等各种问题严重影响前端与客户端开发效率。

###  三.架构演进   
####  （一）总体架构
**<font style="color:#0F7290">1.总体架构图</font>**
<table>
    <thead>
        <tr>
            <th>跨平台架构图</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td><img src="/Resources/Posts/liyizhen_blog_cross_platform_new.jpg"/></td>
        </tr>
    </tbody>
</table>

**<font style="color:#0F7290">2.架构描述</font>**    
**<font style="color:#0F7290">（1）H5业务架构</font>**   
**<font style="color:#18191B">描述表达式:</font>**头部容器 + TNCrossplatform（适配跨平台调用原生API）+ TNHybrid+TNSuperSpeed（离线）+ TNPGLib(原生API) + TNEventBus(通信总线) + 监控  
**<font style="color:#0F7290">（2）Weex业务架构</font>**  
**<font style="color:#18191B">描述表达式:</font>**头部容器 + TNCrossplatform（适配跨平台调用原生API）+ TNWeex+TNSuperSpeed（离线）+ TNPGLib（原生API）+ TNEventBus(通信总线) + 监控 + WeexSDK（官方）  

**<font style="color:#0F7290">3.设计思想</font>**     
**<font style="color:#0F7290">（1）模块化</font>**  
公共逻辑下沉、模块化解耦,  
**<font style="color:#18191B">应用层：</font>**TNHybrid（双核浏览器、Cookie管理、WebUA管理等）、TNWeex（组件/Module、配置、SDKBugFix等）、TNPGLib（各个基础库定义的PG方法分类集合）、其他基础库、业务     
**<font style="color:#18191B">基础层：</font>**TNSuperSpeed（离线化，负责H5与Weex离线资源）、TNEventBus(原生、H5、Weex之间通信)、TNHeader（头部容器）、TNFireEye（监控）   
**<font style="color:#18191B">核心层：</font>**TNPGCore（PG执行核心逻辑）   
**<font style="color:#FF005D">模块化原则主要解决功能模块划分不清晰、视图容器逻辑不清晰、接口不可控等问题</font>**。   
**<font style="color:#0F7290">（2）容器化</font>**    
无论哪种跨平台方案（H5，Weex，RN，Flutter）基本遵循三个原则,   
**<font style="color:#18191B">基于原生容器：</font>**UIViewController，也就各种跨平台UI最终都会绘制到视图容器的View上。  
**<font style="color:#18191B">双向通信能力：</font>**无论哪种跨平台方案基本都具备与原生双向通信能力。  
**<font style="color:#18191B">混合开发模式：</font>**也就是无论哪种跨平台方案都必然与原生共同存在，也就是混合开发模式。必然也就存在混合视图栈管理、混合视图通信、共享数据、共享头部、共享底部TabBar等混合开发问题。  
基于以上原则，可以将各种平台方案进行容器化抽象，比如视图容器、头部容器、浏览器容器（双核浏览器）、Weex实例容器（WeexInstance）等。H5/Weex可以通过PG调用，PGCore中的Dispatch将调用动作派发到对应的容器层来操作容器。   
**<font style="color:#FF005D">容器化原则主要解决基础库依赖、可扩展性差、运行时安全、PG一致性等问题</font>**。      
**<font style="color:#0F7290">（3）平坦化</font>**  
**<font style="color:#18191B">逻辑下沉：</font>**公共逻辑下沉，可以应用到各个平台方案，包括原生，h5，跨平台方案（Weex，RN，Flutter），这部分逻辑不应该依赖任何与特定平台技术相关的逻辑（一般只包括系统接口与工程Base基础库）。     
**<font style="color:#18191B">单项依赖：</font>** 作为最底层逻辑模块（PGCore）应该尽量保持独立性，平行模块之间不应该出现互相依赖行为，该模块只可以被上层模块依赖（头部容器，通信模块，Hybrid，Weex等基础库，或者业务）。    
**<font style="color:#FF005D">平坦化原则主要解决基础库依赖、可扩展性差、线程安全、运行时安全、PG所属范围问题、PG一致性等问题</font>**。    
**<font style="color:#0F7290">（4）立体化</font>**    
**<font style="color:#18191B">联通性</font>**，通过事件总线TNEventBus将原生、H5、Weex联通，实现混合开发模式下页面之间的通信能力。  
**<font style="color:#18191B">稳定性</font>**，通过监控告警TNFireEye对PG调用、H5/Weex加载性能、脚本运行、PG权限白名单、页面加载等异常情况进行监控，敏感重要的监控做到实时告警通知，将告警通知关联到相关负责人，实现监控告警闭环。      
**<font style="color:#18191B">开发效率</font>**，组件、PG、技术方案文档规范化，提供调试工具、PG历史包袱收敛（**<font style="color:#0F7290">18年我曾耗费近6个月的时间推动前端、客户端、产品收敛完遗留历史包袱</font>**）等提高业务团队开发效率。   
**<font style="color:#FF005D">立体化选择主要解决稳定安全问题、开发效率等问题</font>**。  

####  （二）PG架构
**<font style="color:#0F7290">1.总体架构图</font>**
<table>
    <thead>
        <tr>
            <th>PGCore逻辑架构</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td><img src="/Resources/Posts/liyizhen_blog_cross_platform_pg_core.jpg"/></td>
        </tr>
    </tbody>
</table>  

**<font style="color:#0F7290">2.架构描述</font>**       
**<font style="color:#0F7290">（1）PGCore</font>**    
**<font style="color:#18191B">描述表达式:</font>**Instance+Service+Plugin(Method)+Dispatcher+Bridge
**<font style="color:#18191B">Instance：</font>**统领全文的作用，每一个页面对应一个Instance，内部封装Service、Dispatcher、Bridge、PluginMananger等。  
**<font style="color:#18191B">Service：</font>**主要是负责将Bridge传递的方法描述抽象表达成对应的Plugin,并且交给Plugin管理器进行管理（该管理器不对外暴露），以及启动PG执行。  
**<font style="color:#18191B">Bridge：</font>**负责接收跨平台层传递过来的方法描述，以及结果反馈到跨平台层。  
**<font style="color:#18191B">Plugin：</font>**负责决定方法执行完成后是否通过Dispatcher向容器层进行派发调用，以及将方法结果反馈到跨平台层。而内部Method是真正的根据方法描述生成的方法体，业务侧自定义的PG方法也就是在Method扩展中定义方法体，Method中负责对方法定义规则校验、执行情况检查、方法体执行、结果返回封装等等。  
**<font style="color:#18191B">Disaptcher：</font>**负责将PG调用向跨平台层进行派发，并且可以定义派发规则，比如，基础库可以提供默认的PG方法实现（路由u51DeepLink该方法依赖的是51自研路由），管家App使用的基础库提供的默认路由PG，而小蓝本App团队使用的是第三方路由JRoute，业务侧需要自定义实现路由u51DeepLink来覆盖基础库默认提供的U51DeepLink。那么，你可能会想是不是可以在基础库中实现两种方式然后通过bool值来区分，这显然是不合理的，作为基础库不能依赖业务实现。   
**<font style="color:#0F7290">（2）CrossPlatform</font>**  
**<font style="color:#18191B">描述表达式:</font>**Bridge(Hybrid(UI+WK)+Weex+Flutter)+通信扩展（Hybrid(UI+WK)+Weex+Flutter）   
跨平台层主要是作为对PGCore适配跨平台框架的中间层，比如浏览器WK/UI、Weex、Flutter通信方式各不相同，需要在该层适配。那么，你可能想是否可以将该层直接搬迁到PGCore层，当然不可以，那样的话PGCore就会依赖Hybrid、Weex、Flutter，而其他基础库若想自定义PG方法就会依赖PGCore，间接就会依赖各个跨平台基础库，导致基础库依赖成网状复杂化。  
**<font style="color:#0F7290">（3）Header</font>**    
**<font style="color:#18191B">描述表达式:</font>**Container+Implmentation+Router(popn,**<font style="color:#FF005D">基于双向链表解决连续push/pop/present/dismiss问题</font>**)+ Elements(StatusBar+HeaderStyle+Navigator(LBtns+MidTitle+RBtns))  
**<font style="color:#18191B">Container：</font>**头部容器总领全文，作为UIViewController的关联属性，每个视图容器均具默认提供该容器，内部封装了头部样式、混合栈管理、状态栏等操作逻辑。  
**<font style="color:#18191B">Implementation：</font>**作为头部实现的定义类，这里主要是考虑到头部有系统默认导航，以及业务自定义导航可以灵活扩展，比如沉浸式头部，内部封装Elements、Route。  
**<font style="color:#18191B">Elements：</font>**内部封装了状态栏操作、导航样式操作、导航元素操作（自定义左侧按钮、中间标题、右侧按钮等复杂操作）。   
**<font style="color:#18191B">Route：</font>**混合视图堆栈管理，这里需要**<font style="color:#FF005D">重点提及连续Push/Pop/Present/Dismiss问题</font>**（比如A页面Push出B页面，在B页面关闭的时候Push出C页面，会偶现C页面不能出来的问题，一般支付场景中会遇到），当带动画连续操作时就会出现偶现页面不能正常出现或者页面出现顺序被打乱的场景，再加上App或者一些第三方库比如QMUIKit为了防止连续Push/Pop导致Crash而增加了Push/Pop锁同样也导致了以上问题的出现。为了解决该问题，我们提出了使用双向链表构造任务队列的方式解决混合视图栈问题。    

**<font style="color:#0F7290">3.PG使用举例</font>**
<table>
    <thead>
        <tr>
            <th>PG调用逻辑交互</th>
            <th>PG调用代码示例</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td><img src="/Resources/Posts/liyizhen_blog_cross_platform_pg_new_logic.jpeg"/></td>
            <td><img src="/Resources/Posts/liyizhen_blog_cross_platform_pg_new_code.jpeg"/></td>
        </tr>
    </tbody>
</table>  

#### （三）监控告警
**<font style="color:#0F7290">1.监控大盘</font>**
<table>
    <thead>
        <tr>
            <th>Hybrid大盘</th>
            <th>Weex大盘</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td><img src="/Resources/Posts/liyizhen_blog_cross_platform_hybrid_market.png"/></td>
            <td><img src="/Resources/Posts/liyizhen_blog_cross_platform_weex_market.png"/></td>
        </tr>
    </tbody>
</table>  

**<font style="color:#0F7290">2.监控告警</font>**
<table>
    <thead>
        <tr>
            <th>告警大盘</th>
            <th>告警通知</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td><img src="/Resources/Posts/liyizhen_blog_cross_platform_pg_monitor_market.png"/></td>
            <td><img src="/Resources/Posts/liyizhen_blog_cross_platform_pg_warnning.png"/></td>
        </tr>
    </tbody>
</table>  

###  四.PG设计方案

####  （一）设计方案  
**<font style="color:#18191B"> 1.通信协议</font>**   
前端与客户端约定协议规则：callBackid、methodName、Args    
**<font style="color:#18191B">2.方法调用</font>**   
前端方法调用：pg.setNavigationBarightBtns(r1,r2)  
**<font style="color:#18191B">3.生成方法ID</font>**   
前端生成方法体描述，并且随机生成id，前端采用hash表管理。  
**<font style="color:#18191B">4.通信方案</font>**   
跨平台端与客户端通信，将方法抽象描述传递给客户端。  
（1）UIWebView使用kVO获取JSContext，然后注入Block回调作为JS调用客户端方法。  
（2）WKWebView使用userContentController。  
（3）Weex扩展BridModule即可。  
（4）Flutter使用FlutterMethodChannel即可。     
**<font style="color:#18191B">5.生成方法体</font>**    
客户端根据方法体描述生成plugin与方法体   
（1）plugin：负责将结果反馈给跨平台层、以及将执行动作Dispatch到容器层。    
（2）方法体：业务开发只需要定义方法，在犯法中不可以直接访问容器。    
**<font style="color:#18191B">6.方法执行检测</font>**    
（1）同步方法      
（2）异步方法    
（3）执行动作是否进一步传递到容器层        
**<font style="color:#18191B">7.结果返回 </font>**      
（1）直接返回     
（2）异步返回      
（3）容器层动作执行完成后返回，由plugin进行向跨平台层返回结果，回调给客户端。      

#### （二）主要特点
**<font style="color:#18191B">1.API一致性</font>**    
各跨平台、以及多核浏览器可使用一致化API，包括对容器的处理。  
**<font style="color:#18191B">2.灵活扩展</font>**   
可灵活接入各跨平台技术方案，且沉淀的API不受影响。比如，对浏览器的操作包括hybrid容器，业务自定义webview、以及weex中的webview。  
**<font style="color:#18191B">3.高内聚、低耦合</font>**   
所有方法体功能性代码均只限制在该类扩展中方法类文件中，比如对webview操作相关代码也只能在该方法体的分类文件中，不会将大量的功能性代码放入到主体文件中，增强维护性。  
**<font style="color:#18191B">4.轻量无状态</font>**   
以方法体的方式进行描述，而非业界Module的方式（不会造成大量的Module独享常驻App），方法体的生命周期，仅在方法调用开始到结果返回，无状态化保证了同一个方法多次调用的安全性。   
**<font style="color:#18191B">5.稳定安全</font>**    
所有API均是无状态化，并且所属领域只在当前页面，保证了极限状态下堆API的使用各个页面之间互相不影响。并且对方法进行监控、以及方法调用权限管理等。    

#### （三）待优化点  
**<font style="color:#18191B">1.线程模型优化 </font>**  
（1）线程数不可控：目前使用gcd执行方法体，若瞬时有大量方法体执行（比如H5埋点）会导致系统执行过多线程。     
（2）线程任务拥塞。    
（3）线程模型：使用线程池？还是与Weex线程模型一致常驻线程？（若支持同层渲染，UI绘制优先级会更高，逻辑线程优先级低，包括如何平衡复杂度）。   
**<font style="color:#18191B">2.任务优先队列：</font>**
比如，当前页面上的任务优先执行，其他页面任务执行优先级自动降低等。  
**<font style="color:#18191B">3.接口单点优化</font>**
比如业务从原生端数据库读取数据、获取图片数据等方法需要根据每个方法的职能进行优化。

### 五.数据量化
以H5/Weex离线方案TNSuperSpeed架构设计为例数据量化，用客观数据类衡量架构设计对开发效率与稳定性的重要性。   
**<font style="color:#18191B">问题背景：</font>**因基础库设计不合理代码混乱污染业务代码（金融业务）<font style="color:#0F7290">多个产品遭遇AppStore审核问题被拒甚至被下架。CEO新项目小蓝本无法使用原始基础库，抽离关键基础库Swift化，最大限度差异化代码保证审核</font>，**<font style="color:#FF005D">CEO催的比较紧急</font>**。   
**<font style="color:#18191B">架构描述：</font>**应用接口层+资源加载层+资源下载层+资源存储层。   
**<font style="color:#18191B">代码行数：</font>**离线基础库4300行，UIWebView扩展逻辑1197行。   
**<font style="color:#18191B">逻辑复杂：</font>**ui扩展逻辑功能需与wk保持一致，业务无感知切换浏览器；离线涉及并发下载队列、资源持久化与缓存、数据一致性、资源清理、请求拦截、极限条件下容错处理等等细节，逻辑复杂。  
**<font style="color:#18191B">团队协作：</font>**4个同学（其中一名同学刚入职对Swfit不熟悉，只有我自己熟悉离线与Hybrid），分层难易程度协作分工。  
**<font style="color:#18191B">开发效率：</font>**5000+行代码，从OC版本转换到Swfit版本一周（5个工作日）完成提交到业务团队。  
**<font style="color:#18191B">质量保证：</font>**未占用测试资源，所有内部H5业务均使用离线上线后未出现问题。    


     
