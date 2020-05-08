---
layout:     post
title:      图片选择器 
date:       2019-06-26
author:     robertyzli
header-img: Resources/Posts/liyizhen_blog_image_selector_bg.jpg
catalog: true
tags:
    - OC
    - 图片选择器
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

> 本文主要介绍作者在腾讯时开发的一个图片选择器框架，主要用于腾讯手游宝、玩家派（项目已死）、游点赞（项目已死）、WeGame（项目已死），很悲剧啊，从手Q转岗到游戏平台部两年项目组做的项目没有一个成功的，哎...，但是，该图片选择器在性能、稳定性上都已经上线验证过，交互效果支持高仿Instangram图片选择器。

### 一.交互效果
<table>
    <thead>
        <tr>
            <th>效果图一</th>
            <th>效果图二</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td><img src="/Resources/Posts/liyizhen_blog_image_selector_demo1.gif"/></td>
            <td><img src="/Resources/Posts/liyizhen_blog_image_selector_demo2.gif"/></td>
        </tr>
    </tbody>
</table>

### 二.优化方案
**<font style="color:#0F7290">1.性能优化</font>**   
控制层方案：首次进入未发生滚动时，同步请求高清缩略图；发生滚动时，异步请求缩略图,Scrollvelocity(滚动速度)>1000时, 缩略图模式为FastImage(否则为高清)，滚动速度越快，请求的缩略图分辨率越低；滚动静止时，同步请求高清缩略图。  
UI层方案：高清图模式：主线程绘制；FastImage：异步绘制(YYAsyncLayer)。  
优化效果：前FPS在24~48；后FPS在53~57；  微信FPS在46-56之间；手Q FPS在45-58之间。  

**<font style="color:#0F7290">2.内存优化</font>**    
优化方案：缩略图个数>阀值(目前定为800)，异步释放缩略图至常规数目(目前定为500)，按曝光次数从低到高依次释放；收到内存警告时，异步释放掉除当前屏幕上的其他所有缩略图。   
优化结果：滑动内存消耗稳定在20M左右。  

**<font style="color:#0F7290">3.效果对比</font>**   
测试机型：6s 系统:ios 9.3.1 相册数据: 13734照片+97视频  
（1）首次进入选图界面 优化前：1428.8ms 优化后：348.83ms 提高4.1倍。  
（2）滑动列表性能：55-58 FPS  
（3）所有照片滑动出内存占用 优化前：105.MIB 优化后：70.04MIB 降低35.06%   

### 三.架构设计
**<font style="color:#18191B">1.分层与解藕：</font>**数据源层、逻辑层、UI层分离，实现充分解耦。  

**<font style="color:#18191B">2.扩展与复用：</font>**   支持数据层、逻辑交互层、视图层的可复用与可扩展；从九宫格Cell小粒度到整个图片选择器大粒度的不同程度扩展。  

**<font style="color:#18191B">3.屏蔽框架差异：</font>**底层支持iOS7.0的AssetsLibrary 与 iOS8.0以上支持的PhotoKit   框架，接口实现统一，屏蔽系统差异性，更加方便逻辑层与UI层开发。  

**<font style="color:#18191B">4.复杂逻辑简单化：</font>**选择器不同相册的选中与预览状态采用专门的DataSource来管理组织，因此可以支持不同相册的反复选中与预览状态标记，当再次切换至该相册时状态的恢复等复杂逻辑。   

### 四.其他描述
因作者本人不太喜欢写文章，平时项目中遇到的很多疑难问题解决方案也没有来得及梳理，趁本次求职就把之前写的文章梳理到个人博客中，原文最早发表在[**<font style="color:#FF005D">知乎</font>**](https://zhuanlan.zhihu.com/p/35030850),更早应该在我的印象笔记，但因权限问题所以临时性发表在知乎，[**<font style="color:#FF005D">git源码地址</font>**](https://github.com/scutliyizhen/LYZMediaFilesSelector)。 









