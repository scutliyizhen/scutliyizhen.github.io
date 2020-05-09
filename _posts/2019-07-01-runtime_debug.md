---
layout:     post
title:      runtime调试
date:       2019-07-02
author:     robertyzli
header-img: Resources/Posts/liyzhen_blog_runtime_debug_bg.jpg
catalog: true
tags:
    - objc
    - runtime源码
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

> 本文主要介绍调试runtime源码配置方法，这篇总结是2015年左右在腾讯的时候写的，但是当时记录在有道笔记，因权限问题2017年发布到[**<font style="color:#FF005D">知乎</font>**](https://zhuanlan.zhihu.com/p/27786725)。

> objc-rutime对oc对象进行了封装，运行时就好像是oc对象的小型操作系统，底层对对象进行各种操作，因此对运行时深入了解有助于真正了解oc对象以及oc程序的开发。

###  一.用源码生成动态库(libobjc.A.dylib)

####  1.首先拉取源码
[**<font style="color:#0F7290">rutime技术文档</font>**](https://github.com/huang303513/iOS-RunTime-Practice)
[**<font style="color:#0F7290">苹果开放的源代码</font>**](http://www.opensource.apple.com/tarballs/objc4/)

但是由于xcode与mac的升级导致系统/usr/include以及/usr/local/include目录已经不存在，所以如果你直接在官方下载源码然后make,会发生很多错误，但是如果你不运行可以直接在官网下载供阅读使用。
提供一个可以[**<font style="color:#0F7290">正常编译的runtime源码下载链接</font>**](https://github.com/RetVal/objc-runtime)（也可以直接跟我要源码）。

####  2.用xcode打开rutime源码工程
ps:选择objc-My Mac模式，然后点击run，不要选择debug-objc和mark模式。目前该源码工程已经兼容了xcode7。
<table>
    <tbody>
        <tr>
            <td><img src="/Resources/Posts/runtime/1.png"/></td>
        </tr>
    </tbody>
</table>

####  3.查看生成的libobjc.A.dylib
在objc-My Mac模式下点击run，产生的libobjc.A.dylib动态库在默认目录products下
<table>
    <tbody>
        <tr>
            <td><img src="/Resources/Posts/runtime/2.png"/></td>
        </tr>
    </tbody>
</table>
也可以在xcode->preference->Location->Derived Data 然后点击箭头，就可以定位到products目录，也就是libobjc.A.dylib生成的目录。
<table>
    <tbody>
        <tr>
            <td><img src="/Resources/Posts/runtime/3.png"/></td>
        </tr>
    </tbody>
</table>
当然，libobjc.A.dylib的生成目录也可以修改，xcode->preference->Location->Derived Data->Advanced->Custom(选择绝对路径Absolute修改Products或者Intermdeiates目录)
<table>
    <tbody>
        <tr>
            <td><img src="/Resources/Posts/runtime/4.png"/></td>
        </tr>
    </tbody>
</table>
**<font style="color:#FF005D">注意：这里需要提醒的是，这里的修改会对所有用xcode打开的项目生效，所以不建议修改这两个目录。</font>**

###  二.创建Mac工程,用来调试runtime

**<font style="color:#0F7290">备注:至于为什么不创建ios工程源码调试还未研究，读者可以自己试试</font>**
####  1.打开工程创建向导
<table>
    <tbody>
        <tr>
            <td><img src="/Resources/Posts/runtime/5.png"/></td>
        </tr>
    </tbody>
</table>

####  2.创建mac项目
<table>
    <tbody>
        <tr>
            <td><img src="/Resources/Posts/runtime/6.png"/></td>
        </tr>
    </tbody>
</table>

###  三.用自定义的runtime 动态库(libobjc.A.dylib)替换默认的运行时动态库

####  1.引入自己编译的libobjc.A.dylib动态库
**<font style="color:#0F7290">（1）新建一个Build选项</font>**
<table>
    <tbody>
        <tr>
            <td><img src="/Resources/Posts/runtime/7.png"/></td>
        </tr>
    </tbody>
</table>
**<font style="color:#0F7290">（2）引入动态库</font>**
<table>
    <tbody>
        <tr>
            <td><img src="/Resources/Posts/runtime/9.png"/></td>
        </tr>
    </tbody>
</table>

####  2.替换工程运行时动态库libobjc.A.dylib
**<font style="color:#0F7290">（1）新建新的脚本Build选项</font>**
<table>
    <tbody>
        <tr>
            <td><img src="/Resources/Posts/runtime/10.png"/></td>
        </tr>
    </tbody>
</table>

**<font style="color:#0F7290">（2）添加运行脚本</font>**（替换运行时动态库为源码生成的动态库，注意这里只对本工程起作用）

> install_name_tool -change /usr/lib/libobjc.A.dylib /Users/robertyzli/Desktop/ObjcTest/products/Debug/libobjc.A.dylib /Users/robertyzli/Desktop/ObjcTest/products/Debug/MacTest.app/Contents/MacOS/MacTest  

要根据自己mac中生成的libobjc.A.dylib运行时动态库生成的目录、以及项目运行文件TestMac的目录替换上述命令中的目录。
<table>
    <tbody>
        <tr>
            <td><img src="/Resources/Posts/runtime/11.png"/></td>
        </tr>
    </tbody>
</table>

配置动态库的路径出现错误，如果修改运行脚本时，需要注意以下几方面   
**<font style="color:#18191B">（1）修改完运行脚本，需要关闭xcode重新启动</font>**  
**<font style="color:#18191B">（2）重新启动xcode后，执行clean</font>**  
**<font style="color:#18191B">（3）执行Build</font>**  
**<font style="color:#18191B">（4）执行run</font>**  
最好按照这个步骤，否则修改后的脚本不起作用。  
这里用的是绝对路径，当然也可以使用相对路径（@rpath,@loader_path, @executable_path）。   

####  3.Debug TestMac 逐步调试进入runtime 源码
<table>
    <tbody>
        <tr>
            <td><img src="/Resources/Posts/runtime/12.png"/></td>
            <td><img src="/Resources/Posts/runtime/13.png"/></td>
        </tr>
    </tbody>
</table>

> 小伙伴们到这里就可以阅读、调试runtime源码啦.......是不是很幸福，有助于我们对oc 以及swift更深入的理解，以及解决一些复杂问题。







