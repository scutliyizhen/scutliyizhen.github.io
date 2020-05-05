---
layout:     post
title:      跨平台架构演进 
date:       2029-05-04
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

&ensp;&ensp;&ensp;&ensp;2017年我进入51信用卡，当时客户端与前端使用的是Hybrid混合开发模式。在51的两年基本上就是处于不断填坑状态，开始接触WebKit的时候Hybrid本身并没有架构的概念，很多代码基本都是以功能逻辑融合在一起，每当新增需求都很困难担心改动点是否全面有遗漏；而当Weex跨平台接入之后又依赖Hybrid，出现了运行时安全、耦合依赖等问题，本文会详细阐述遇到的实际问题以及是如何优化架构解决问题。

###  一.目标与体现
**<font style="color:#FF005D">总体目标：</font>**跨平台层作为前端与Native的中间混合层，主要目标是为Hybrid/Weex（或者其他跨平台方案）**<font style="color:#FF005D">提供更好的服务能力（或者互动能力）</font>**。
**<font style="color:#FF005D">目标体现：高扩展、高复用、高内聚低耦合、高性能（待优化）、方便易用、稳定安全</font>**

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

###   二.早期Hybrid架构   
<table>
    <thead>
        <tr>
            <th>Hybrid架构图</th>
            <th>混合开发通信逻辑架构</th>
            <th>备注</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td><img src="/Resources/Posts/liyizhen_blog_cross_platform_old.png"/></td>
            <td><img src="/Resources/Posts/liyizhen_blog_cross_platform_old_pg.png"/></td>
            <td>架构图是我2017年刚进公司的时候在公司的wiki上翻到的，对应的基础库代码也就是早期的WebAppKit。</td>
        </tr>
    </tbody>
</table>

#### 面临的问题主要有   
- <font style="color:#0F7290">1.功能模块划分不清晰</font> 
比如，以上所有的功能点都放在WebAppKit基础库中，并且在基础库中没有合理的划分模块。
- <font style="color:#0F7290">2.视图容器逻辑不清晰</font> 
比如，各种UI组件初始化、WebView（WK/UI）配置以及回调、导航控制逻辑等等都放在Web视图容器中，各种逻辑通过不断的添加属性来区分，当时该文件有1000多行的代码，对于后续扩展监控、离线、注入脚本则会带来很大的困难，导致维护成本很高。
- <font style="color:#0F7290">3.接口不可控问题</font> 
比如，在视图中WebView完全暴露，而子类可以在任意时机操作WebView，引入不可控因素。
- <font style="color:#0F7290">4.WebKit可扩展性、耦合性问题</font> 
比如，业务侧只能使用继承的方式使用基础WebController，不支持组合的方式，并且不能单独使用WebView。比如业务同学单独使用WebView做运营广告位，只能使用野路子的方式，操作WebController中的View。 
- <font style="color:#0F7290">5.PG(原生API)设计问题</font> 
比如，耦合度高、线程安全、基础库依赖、运行时问题（比如PG生命周期、所属范围等）、可维护性差等问题。 
  
###  三.早期跨平台架构    
<table>
    <thead>
        <tr>
            <th>跨平台架构图</th>
            <th>混合开发通信逻辑架构</th>
            <th>备注</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td><img src="/Resources/Posts/liyizhen_blog_cross_platform_weex_hybrid.png"/></td>
            <td><img src="/Resources/Posts/liyizhen_blog_cross_platform_weex_hybrid_pg.png"/></td>
            <td>Weex平台接入后，直接依赖WebKit，导致潜在了更多的问题</td>
        </tr>
    </tbody>
</table>

####  面临的问题主要有  
- <font style="color:#0F7290">1.基础库依赖问题</font> 
比如，Weex依赖Webkit。
- <font style="color:#0F7290">2.运行时安全问题</font> 
比如，由于PG可以随意访问容器UI元素，而接入Weex后，导致沉淀的PG运行处于不安全状态，Weex视图强制转换成Web视图对象。
- <font style="color:#0F7290">3.维护困难</font> 
比如，由于Weex视图对象强制转换成Web视图对象使用，比如每次Web视图修改都要考虑Weex视图做相应修改，很不安全。
- <font style="color:#0F7290">4.质量问题</font> 
缺少必要的监控手段。
- <font style="color:#0F7290">5.PG历史包袱</font> 
平台兼容性问题、缺少规范、数据不一致、文档描述与实现不一致等各种问题严重影响跨平台效率。

###  四.架构演进   
<table>
    <thead>
        <tr>
            <th>跨平台架构图</th>
            <th>备注</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td><img src="/Resources/Posts/liyizhen_blog_cross_platform_new.jpg"/></td>
            <td>新的架构演进,Hybrid、PGCore、头部容器等是Swift版本</td>
        </tr>
    </tbody>
</table>

###  五.PG设计演进  
<table>
    <thead>
        <tr>
            <th>PGCore逻辑架构</th>
            <th>备注</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td><img src="/Resources/Posts/liyizhen_blog_cross_platform_pg_core.jpg"/></td>
            <td>新的架构演进,Hybrid、PGCore、头部容器等是Swift版本</td>
        </tr>
    </tbody>
</table>  

**举例（H5设置导航右侧按钮）**
<table>
    <thead>
        <tr>
            <th>旧方案使用方式</th>
            <th>新方案使用方式</th>
            <th>备注</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td><img src="/Resources/Posts/liyizhen_blog_cross_platform_pg_old.jpg"/></td>
            <td><img src="/Resources/Posts/liyizhen_blog_cross_platform_pg_new.jpg"/></td>
            <td>老方案问题点：Weex不能直接使用该方法（意味着无法降级）；若扩展其他跨平台方案则需要重新各自实现（比如再扩展Flutter）；基础侧提供默认设置方式，不同业务团队无法自定义扩展。</td>
        </tr>
    </tbody>
</table>  

#### （一）解决问题 
- <font style="color:#0F7290">1.基础库依赖问题</font>    
- <font style="color:#0F7290">2.可复用性问题</font>    
- <font style="color:#0F7290">3.运行时安全问题</font>    
- <font style="color:#0F7290">4.API与框架稳定性问题</font>   
- <font style="color:#0F7290">5.线程安全问题</font>   
- <font style="color:#0F7290">6.可维护性问题</font> 

####  （二）设计方案  
#####  1.通信协议
callBackid、methodName、Args
#####  2.前端方法调用  
pg.setNavigationBarightBtns(r1,r2)
#####  3.前端生成方法体描述，并且随机生成id
前端采用hash表管理。
#####  4.前端与客户端通信，将方法抽象描述传递给客户端。
#####  5.客户端根据方法体描述生成plugin与方法体。 
（1）plugin：负责将结果反馈给跨平台层、以及将执行动作Dispatch到容器层。  
（2）方法体：业务开发只需要定义方法，在犯法中不可以直接访问容器。  
#####  6.方法执行与方法检测
（1）同步方法     
（2）异步方法   
（3）执行动作是否进一步传递到容器层   
#####  7.结果返回  
（1）直接返回  
（2）异步返回   
（3）容器层动作执行完成后返回，由plugin进行向跨平台层返回结果，回调给客户端。   

#### （三）主要特点
#####  1.API一致性
各跨平台、以及多核浏览器可使用一致化API，包括对容器的处理。
#####  2.灵活扩展
可灵活接入各跨平台技术方案，且沉淀的API不受影响。比如，对浏览器的操作包括hybrid容器，业务自定义webview、以及weex中的webview。
#####  3.高内聚、低耦合
所有方法体功能性代码均只限制在该类扩展中方法类文件中，比如对webview操作相关代码也只能在该方法体的分类文件中，不会将大量的功能性代码放入到主体文件中，增强维护性。
#####  4.轻量无状态
以方法体的方式进行描述，而非业界Module的方式（不会造成大量的Module独享常驻App），方法体的生命周期，仅在方法调用开始到结果返回，无状态化保证了同一个方法多次调用的安全性。
#####  5.稳定安全
所有API均是无状态化，并且所属领域只在当前页面，保证了极限状态下堆API的使用各个页面之间互相不影响。并且对方法进行监控、以及方法调用权限管理等。   

#### （四）待优化点  
#####  1.线程模型优化    
（1）线程数不可控：目前使用gcd执行方法体，若瞬时有大量方法体执行（比如H5埋点）会导致系统执行过多线程。     
（2）线程任务拥塞。    
（3）线程模型：使用线程池？还是与Weex线程模型一致常驻线程？（若支持同层渲染，UI绘制优先级会更高，逻辑线程优先级低，包括如何平衡复杂度）。   
#####  2.任务优先队列   
比如，当前页面上的任务优先执行，其他页面任务执行优先级自动降低等。  
#####  3.接口单点优化  
比如业务从原生端数据库读取数据、获取图片数据等方法需要根据每个方法的职能进行优化。
     
