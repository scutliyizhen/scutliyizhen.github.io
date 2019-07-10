---
layout:     post
title:      跨平台架构演进(梳理中......)  
date:       2019-07-01
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

###   一.早期Hybrid架构 
  
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

#### （一）功能点概况      
<font style="color:#0F7290;font-weight:bold;">&ensp;&ensp;&ensp;&ensp;1.主要包含的功能点</font>
&ensp;&ensp;&ensp;&ensp;通信、Hybrid API、Event、URL配置、调试工具、UniversalLink、容器UI。    
<font style="color:#0F7290;font-weight:bold;">&ensp;&ensp;&ensp;&ensp;2.不包含的功能模块</font>
&ensp;&ensp;&ensp;&ensp;离线、监控。 

#### （二）面临的问题主要有      
<font style="color:#0F7290;font-weight:bold;">&ensp;&ensp;&ensp;&ensp;1.功能模块划分不清晰</font>
&ensp;&ensp;&ensp;&ensp;以上所有的功能点都放在WebAppKit基础库中，并且在基础库中没有合理的划分模块。    
<font style="color:#0F7290;font-weight:bold;">&ensp;&ensp;&ensp;&ensp;2.视图容器逻辑不清晰</font>
&ensp;&ensp;&ensp;&ensp;各种UI组件初始化、WebView（WK/UI）配置以及回调、导航控制逻辑等等都放在Web视图容器中，各种逻辑通过不断的添加属性来区分，当时该文件有1000多行的代码，对于后续扩展监控、离线、注入脚本则会带来很大的困难，维护很困难。       
<font style="color:#0F7290;font-weight:bold;">&ensp;&ensp;&ensp;&ensp;3.接口不可控问题</font>
&ensp;&ensp;&ensp;&ensp;比如在视图中WebView完全暴露，而子类可以在任意时机操作WebView，引入不可控因素。     
<font style="color:#0F7290;font-weight:bold;">&ensp;&ensp;&ensp;&ensp;4.WebKit可扩展性、耦合性问题：</font>
&ensp;&ensp;&ensp;&ensp;业务侧只能使用继承的方式使用基础WebController，不支持组合的方式，并且不能单独使用WebView。比如业务同学单独使用WebView做运营广告位，只能使用野路子的方式，操作WebController中的View。      
<font style="color:#0F7290;font-weight:bold;">&ensp;&ensp;&ensp;&ensp;5.PG(原生API)设计问题</font>
&ensp;&ensp;&ensp;&ensp;耦合度高、线程安全、基础库依赖、运行时问题（比如PG生命周期、所属范围等）、可维护性差等问题。   

###  二.跨平台架构    
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
- <font style="color:#0F7290">PG历史包袱</font> 
平台兼容性问题、缺少规范、数据不一致、文档描述与实现不一致等各种问题严重影响跨平台效率。

###  三.架构演进   

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
            <td><img src="/Resources/Posts/liyizhen_blog_cross_platform_new.jpg"/></td>
            <td><img src="/Resources/Posts/liyizhen_blog_cross_platform_pg_core.jpg"/></td>
            <td>新的架构演进,Hybrid、PGCore、头部容器等是Swift版本</td>
        </tr>
    </tbody>
</table>


####  1.Hybrid容器    
- <font style="color:#0F7290">UI层</font>WebView（WK/UI）、组件（加载loading、异常错误、进度条等）、Controller、WebView接口扩展。     
- <font style="color:#0F7290">Engine层</font>分为WKEngine、UIEngine，主要处理Web容器与离线、监控、注入脚本、路由、PGCore等模块的交换核心逻辑。   
- <font style="color:#0F7290">注入脚本模块</font>区分WKWebView注入脚本、UIWebView注入脚本（与离线技术方案相似，涉及业务敏感性所以不透露太多）。   
- <font style="color:#0F7290">Data层</font>处理WebUA与Cookie的问题。  

####  2.Weex容器    

####  3.头部容器   

####  4.离线模块 

####  5.EventBus   

###  四.PG设计演进  

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

####  主要解决以下问题     
- <font style="color:#0F7290">1.基础库依赖问题</font>    
- <font style="color:#0F7290">2.可复用性问题</font>    
- <font style="color:#0F7290">3.运行时安全问题</font>    
- <font style="color:#0F7290">4.API与框架稳定性问题</font>   
- <font style="color:#0F7290">5.线程安全问题</font>   
- <font style="color:#0F7290">6.可维护性问题</font>   

> 这是第一篇技术文章，后续计划先将已经阅读的Weex源码进行剖析分享给大家，例如从前端VM（虚拟Dom，Vue Dom）到客户端虚拟Dom（RenderObject）的整个渲染过程分析、线程模型设计、架构分析、性能优化等。   
