---
layout:     post
title:      NvWaPatch 
date:       2019-08-01
author:     robertyzli
header-img: Resources/Posts/liyizhen_blog_ffi_hot_fix_bg.jpg
catalog: true
tags:
    - ffi/js/swift
    - 热修复
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

> 本文主要介绍自己利用业余时间完成的基于ffi的热修复项目，目前只是串联完主链路，可以为大家对动态化方案提供一些思路。

###  一.开源缘由  
1.精力有限，目前没有时间进一步丰富完善功能。  

2.为大家对动态化方案提供一种思路，或者作为业余项目学习。  

3.该项目本意想扩展到Swift，有兴趣的同学可以进一步探索下。  

4.[**<font style="color:#FF005D">项目源码地址</font>**](https://github.com/scutliyizhen/NvWaPatch)  

###  二.技术价值  
1.动态替换原生代码，线上修复bug。  

2.动态增加原生模块，线上增加新功能。  

3.可与其他跨平台框架结合，增强动态能力。  

###  三.关于审核  
**<font style="color:#0F7290">1.特定类名检查:</font>**主要是针对一些公众比较熟知的热修复开源库比如JSPatch。  

**<font style="color:#0F7290">2.特定API检查：</font>**苹果对消息转发、方法替换API进行检查，这些机制都属于系统API，所以苹果比较容易校验。  

**<font style="color:#0F7290">3.实现原理不同:</font>**女娲热修复机制脚本DSL与JSPatch保持一致（后续准备模块化，容易扩展）；而底层设计与JSPatch完全不同，未使用消息转发机制，而是直接替换调用接口地址，绕过苹果消息转发机制，那么苹果在Runtime机制下则很难察觉。  

**<font style="color:#0F7290">4.底层架构设计：</font>** （JSEngine+DSLParse+Mehtod+HookEngine）重新设计，不在苹果特定检查类名列表中。  

**<font style="color:#0F7290">5.审核案例：</font>**业界有企业使用该机制实现的热修复供企业内部使用，审核可用，故不开源。  

###  四.效果演示
<table>
    <thead>
        <tr>
            <th>效果演示图</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td><img src="/Resources/Posts/liyizhen_blog_ffi_hot_fix_NvWaPatch.gif"/></td>
        </tr>
    </tbody>
</table> 






