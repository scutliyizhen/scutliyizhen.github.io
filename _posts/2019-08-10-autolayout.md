---
layout:     post
title:      从数学角度来看IOS中的AutoLayout布局
date:       2019-08-10
author:     robertyzli
header-img: Resources/Posts/liyzhen_blog_autolayout_bg.png
catalog: true
tags:
    - UIKit
    - AutoLayout
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

<table>
    <tbody>
        <tr>
            <td><img src="/Resources/Posts/uotlayout/1.png"/></td>
            <td><img src="/Resources/Posts/uotlayout/2.png"/></td>
            <td><img src="/Resources/Posts/uotlayout/3.png"/></td>
            <td><img src="/Resources/Posts/uotlayout/4.png"/></td>
            <td><img src="/Resources/Posts/uotlayout/5.png"/></td>
            <td><img src="/Resources/Posts/uotlayout/6.png"/></td>
            <td><img src="/Resources/Posts/uotlayout/7.png"/></td>
            <td><img src="/Resources/Posts/uotlayout/8.png"/></td>
            <td><img src="/Resources/Posts/uotlayout/9.png"/></td>
            <td><img src="/Resources/Posts/uotlayout/10.png"/></td>
            <td><img src="/Resources/Posts/uotlayout/11.png"/></td>
            <td><img src="/Resources/Posts/uotlayout/12.png"/></td>
            <td><img src="/Resources/Posts/uotlayout/13.png"/></td>
            <td><img src="/Resources/Posts/uotlayout/14.png"/></td>
            <td><img src="/Resources/Posts/uotlayout/15.png"/></td>
            <td><img src="/Resources/Posts/uotlayout/16.png"/></td>
        </tr>
    </tbody>
</table>

