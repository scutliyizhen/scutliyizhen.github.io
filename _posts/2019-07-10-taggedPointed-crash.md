---
layout:     post
title:      标签指针(TaggedPonted  in arm64 of iOS)在RN项目中引起Crash 
date:       2019-07-10
author:     robertyzli
header-img: Resources/Posts/liyzhen_blog_taggedPointed_bg.jpg
catalog: true
tags:
    - OC/RN
    - Crash/TaggedPonted
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

>  本文主要介绍有关标签指针的一些知识，该问题是2016年作者在腾讯尝试RN的时候遇到的一个crash问题，文章最初发布在有道笔记，因权限问题于2017.07.09发布到知乎。

###  一.问题背景

<font style="color:#0F7290">“存在即合理”。凡是存在的，都是合乎规律的。任何新事物的产生总要的它的道理；任何新事物的发展总是有着取代旧事物的能力。在React.js 2015大会上，Facebook公布了基于React.js的React Native，该框架适用于android与ios两大平台。因此，带着好奇心，对该框架进行了尝试。</font>
根据[**<font style="color:#0F7290">React Native配置创建了RN工程</font>**](https://zhuanlan.zhihu.com/p/27777402)，项目非常顺利的跑起来。但是，技术的出现应该要应用到我们自己的项目中，可以参照[**<font style="color:#0F7290">React Native集成现有项目配置自己的项目</font>**](https://zhuanlan.zhihu.com/p/27777402)。开始我先配置了手游宝项目(Sybplatform)，一切配置结束后，激动人心的时刻到了，终于可以真正的将该技术应用到自己的项目中了。当我用xcode启动手游宝项目以后，进入到React Native加载页面时，问题出现了，如下图所示：
<table>
    <tbody>
        <tr>
            <td><img src="/Resources/Posts/TaggedPonted/1.png"/></td>
        </tr>
    </tbody>
</table>

###  二.问题分析
####  1.初步分析
当看到EXC_BAD_ACCESS时，这种crash大家都很熟悉了，一定是非法方位了某块内存。
objc_msgsend运行时方法，都很熟悉了，就是给对象jsonObject发送JSONKitSelector消息，该参数是0和error。消息JSONKitSelector的返回值是NSString对象。首先看JSONKitSelector是什么消息，如下图所示
<table>
    <tbody>
        <tr>
            <td><img src="/Resources/Posts/TaggedPonted/2.png"/></td>
        </tr>
    </tbody>
</table>
从上图可以知道JSONKitSelector=@"JSONSStringWithOptions:erro:",jsonObject为NSArray对象，因此上述表达式可以表达为: [jsonObject JSONStringWithO hasptions:0 error:&error](此处error是输出参数，因此采用的是指针的指针类型) 。
我们知道该Crash为非法访问了内存，并且知道 [jsonObject JSONStringWithOptions:0 error:&error]，因此重点看jsonObject是否已经被释放，接收消息对象释放，造成非法访问内存。但是从上图p jsonObject来看，该对象没有被释放。**<font style="color:#FF005D">那问题到底出在哪里了呢？</font>**必须要深入到JSONSStringWithOptions:erro:内部去看到底发生了什么，**<font style="color:#FF005D">在工程全文搜索该方法，很不幸没有搜到实现该方法的代码？</font>**。
由于ReactNative对不了解，又无法进入到该方法内部。那怎么定位问题？我们知道初始化的工程AwesomeProjec是可以正常运行的，故打开AwesomeProjec工程，在该代码处打断点，发现并没有调用objc_msgsend方法，说明NSArray的category中没有添加JSONSStringWithOptions:erro:方法。
因此，google了此方法，**<font style="color:#FF005D">发现该方法是JSONkit中的方法</font>**。下载源码JSONKit,然后进入函数内部，如下图所示：   
<table>
    <tbody>
        <tr>
            <td><img src="/Resources/Posts/TaggedPonted/3.png"/></td>
        </tr>
    </tbody>
</table>

####  2.深入分析
由上图可以知道发生crash的地方出现在了void * keyObjectISA = \*(void\*\*)key[idx].首先最先想到的就是idx索引是否超出范围，经过验证idx索引并未超出范围，继续查证如下图所示：
<table>
    <tbody>
        <tr>
            <td><img src="/Resources/Posts/TaggedPonted/4.png"/></td>
        </tr>
    </tbody>
</table>
分析过程：keys[1024]元素是指针类型的数据，p keys[idx] 与p (void*)[idx]的作用相同，都是打印keys[0]地址值所指向的对象；但是，当执行p *(void**)keys[idx](也就是 p *(void**)keys[0])控制台输出了**<font style="color:#FF005D">“无法读取内存”</font>**的log；当执行p *(void**)keys[1]控制太正常输出了地址；当执行p*(void**)keys[2]时，控制台输出了"无法读取内存"的log。因此，从以上控制台输出可以看出当对象为**<font style="color:#FF005D">"NSTaggedPointerString"</font>**时就会出现“无法读取内存”的问题，这也就是发生的crash的问题所在。

####  3.定位问题
既然已经知道“NSTaggedPointerString”引起的Crash，并且知道_NSCFString并没有造成crash,这两种数据类型有什么区别呢？
**<font style="color:#0F7290">（1）iOS内存分区</font>**
<table>
    <tbody>
        <tr>
            <td><img src="/Resources/Posts/TaggedPonted/5.png"/></td>
        </tr>
    </tbody>
</table>
栈区：系统自动分配，回收也有系统来完成，速度比较快，不会产生内存碎片。
堆区：由程序员负责分配和释放，速度比较慢，容易产生内存碎片。
全局区（静态区）：全局变量和静态变量存放的地方，由系统负责释放。
常量区：存放常量字符串，程序结束后由系统释放。
代码区：存放程序代码的地方。

**<font style="color:#0F7290">（2）深入理解NSString</font>**
NSString是一个类簇，一个公开的抽象类，当我们创建NSString的时候都会创建一个私有子类的实例(大家调试的时候一定碰到过_NCFString, **<font style="color:#FF005D">\_NSCFString,NSTaggedPointerString</font>**
 ),由于是私有子类，因此在官方文档是查不到关于这三个私有子类的相关信息的，下面简单来简单介绍下它们:
**<font style="color:#FF005D">\_NSCFConstantString</font>** :字符串常量存放在常量区，相同内容的\_\_NSCFConstantString的对象地址是相同的，可以理解成字符串常量是一种单例。即使对这种对象进行release操作，retainCount也不会有任何变化。
字符串常量一般是通过字面值@"\.\.\.", CFSTR(""),stringWithString方法产生的。
​\_NSCFString:是对象运行时产生的一种NSString的子类,存放在堆区，通过SStingWithFormat等方法产生的。
**<font style="color:#FF005D">最后一个NSTaggedPonterString是什么呢</font>**？

**<font style="color:#0F7290">（3）理解标签指针（NSTaggedPointer）</font>**
苹果公司在2013年9月发布了iphone5s,该设备最受关注的就是首次采用了64位架构的A7双核处理器，64位处理器的出现极大的提高了处理速度。苹果公司为了进一步节省能存与提高执行效率，推出了TaggedPointer的概念。据了解引入TaggedPointer以后对于64位程序来讲，内存节省了将近一般，内存访问速度提升了3被，创建销毁速度提升了100倍。下面我们来简单了解下什么是TaggedPointer。
在认识TaggedPointer之前，先来看一下**<font style="color:#FF005D">32位机器内存的访问原理</font>**？，如下图所示：
<table>
    <tbody>
        <tr>
            <td><img src="/Resources/Posts/TaggedPonted/6.png"/></td>
        </tr>
    </tbody>
</table>
在未引入NSTaggedPointer之前，由32位机器扩展为64位机器后，对象的地址也直接扩展为64位，指针指向的数据内容也随之扩展为原来的2倍，如下图所示：
<table>
    <tbody>
        <tr>
            <td><img src="/Resources/Posts/TaggedPonted/7.png"/></td>
        </tr>
    </tbody>
</table>
根据上图所示，我们知道64位的处理器处理一个NSNumber对象时，首先将8个字节的地址读入指令寄存器，然后根据地址找到对象所在的地方，将NSNumber对象数据（若大于64位，则分批）读到指令寄存器。最后，将读取到寄存器的内容放到逻辑运算单元（AU）参与运算。在一定程度上讲，的确是加快了运算速度，因为每次处理的数据内容增加了。
但是，对于一般的程序，比如一个整型数据，4个字节所代表的数据(2^31=2147483648，另外1位作为符号位)足够用了，当扩展为8个字节以后，只有几位是有数据的，其余位均是0。为了充分利用数据为0的空间，苹果公司引入了NSTaggedPointer数据，数据模型如下图所示：
<table>
    <tbody>
        <tr>
            <td><img src="/Resources/Posts/TaggedPonted/8.png"/></td>
        </tr>
    </tbody>
</table>
根据模型图所示，可以看到一个NSNumber对象数据（该数据不超过2^59-1,至于为什么不是2^63-1后面会介绍），用一个8字节就可以表示了，在未引入NSTaggedPointer之前需要用16位（8字节地址+8字节数据）来表示。关于NSTaggedPointer构造，如下图所示：
<table>
    <tbody>
        <tr>
            <td><img src="/Resources/Posts/TaggedPonted/9.png"/></td>
        </tr>
    </tbody>
</table>
根据上图所述，NSNumber类型数据3用一个8字节64位空间来存储的话，末尾最后一位1是NSTaggedPointer的标记位，从末尾倒数第二位至倒数第四位代表类型（比如上图整形数据用101=5，但实际上苹果公司应该改变了策略，但是这不影响我们对该数据类型的理解），下面我们来看下实际运行的结果，如下图所示：
<table>
    <tbody>
        <tr>
            <td><img src="/Resources/Posts/TaggedPonted/10.png"/></td>
        </tr>
    </tbody>
</table>
比如上图中的a=1真实表示为:0x0000000000000001, b=2真实表示为:0x0000000000000002，从这两个数据来看最后一位并不是标记位，而是真实的数据。再来看下字符串i,k和m，这两个字符串都是NSTaggedPointerString类型的数据，分别是@"b",@"c", 它们在内存中的表示分别为：0xa000000000000611,0xa000000000000621,0xa000000000000631这三个字符串最高位都是a,最后一位都是1，对于NSTaggedPointerString来说这应该是标记位；再来看他们的ASII码，61(0110 0001)=2^6+2^5+2^0=97, 62(0110 0010)=2^6+2^5+2^1=98, 63(0110 0011)=2^6+2^5+2^1+2^0=99。 **<font style="color:#FF005D">综上所述，NSTaggedPointer类型数据（较小的数据）用一个8字节的空间就可以存储了，不过较大的数据还是会按照以前的存储方式存储</font>**，比如g=1000000000,运行结果如上图。

###  三.解决方案
 根据上述分析，我们NSTaggedPointerString已经不是一个地址了，而是实实在在的数据，既然知道NSTaggedPointer类型数据是在64位机器上苹果公司做的优化，对此我们先关闭arm64编译，看看能不能解决问题：

####  1.关闭arm64
<table>
    <tbody>
        <tr>
            <td><img src="/Resources/Posts/TaggedPonted/10.png"/></td>
        </tr>
    </tbody>
</table>
运行结果如下图所示（从图中我们可以看到已经没有NSTaggpointerString数据了）：
<table>
    <tbody>
        <tr>
            <td><img src="/Resources/Posts/TaggedPonted/12.png"/></td>
        </tr>
    </tbody>
</table>
关闭断点调试，将代码跑起来，如下图所示：
<table>
    <tbody>
        <tr>
            <td><img src="/Resources/Posts/TaggedPonted/13.png"/></td>
        </tr>
    </tbody>
</table>

####  2.尝试方案
上述解决方案由于太暴力，显然不是解决办法。再来分析下出问题的代码:void*keyObjectISA=\*(void\*\*)keys[idx] 并且void* keys[1024]。分析如下（arm64位下编译运行）：
**<font style="color:#FF005D">首先</font>**，数组keys每个单元（每个单元为8个字节）存放的都是地址值。keys[idx]取出来的数据是一个8字节的数据A，然后对A做强制数据类型转换(void\*\*),也就是说A是一个间接指针（地址的地址），对A做一次间接访问。
**<font style="color:#FF005D">其次</font>**，根据前面的分析我们知道NSTaggedPointerString已经不是地址值了，因此对A做内存间接访问，必定会出现Crash。如果对A不做间接访问，直接是void\* keyObjectISA=(void\*)kes[idx],是否可以呢？
<table>
    <tbody>
        <tr>
            <td><img src="/Resources/Posts/TaggedPonted/14.png"/></td>
            <td><img src="/Resources/Posts/TaggedPonted/15.png"/></td>
        </tr>
    </tbody>
</table>
走到这一步，是不是问题解决了呢？部分读者可能以为程序正常运行，则认为应该是解决问题了。**<font style="color:#FF005D">但是，我想说还没有，因为你没有真正理解keyObjectISA=*(void**)keys[idx] 并且void* keys[1024]</font>**。

###  3.问题解决
再深入分析keyObjectISA=\*(void\*\*)keys[idx] 并且void* keys[1024]。
首先来分析，针对_NSCFString类型的字符串，这种字符串的存储方式与32位机器上是一样的。那么keys[id]取出的数据A是字符串的地址，那么(void\*\*)A可以这样理解(void\*)((void\*)A),因为B=(void\*)A很容易理解就是一个指向任意数据类型的地址，也就是B指向一个对象；故C=(void\*)B应该是把对象强制转换为一个void类型的指针， 即C是一个地址值，因此对C进行间接访问"\*C "(也就是\*(void\*\*)keys[idx]),其实就是访问(void\*)keys[idx]指针指向的字符串对象的第一个字节，把这第一个字节当作指向另一个对象的指针。估计大家都被绕晕了，就是把\*(void\*\*)keys[idx]理解为访问字符串对象所在内存的头8个字节的内容。
那么一个对象的头8个字节是什么数据呢？如图所示：
<table>
    <tbody>
        <tr>
            <td><img src="/Resources/Posts/TaggedPonted/16.png"/></td>
            <td><img src="/Resources/Posts/TaggedPonted/17.png"/></td>
        </tr>
    </tbody>
</table>
从上图可以知道，oc中的任何一个对象在内存中最先存储的是ISA的一个指针，这个ISA其实就是该类的元类（在这里对该概念不做详细叙述，读者可以自行查阅oc对象模型，深入理解）。
因此,**<font style="color:#FF005D">keyObjectISA=*(void**)keys[idx],其实就是获取字符串对象的isa指针</font>**，那么针对arm64位下由于引入了标签指针NSTaggedPointer，苹果公司只保证内部API对对象的访问遵守标签这阵规则，故对象的访问尽量要使用苹果提供的API，不要随意使用c或者c++对内存直接访问。为此，**<font style="color:#FF005D">苹果给大家提供了运行时方法objct_getClass获取对象的ISA</font>**。解决方案如下：
<table>
    <tbody>
        <tr>
            <td><img src="/Resources/Posts/TaggedPonted/18.png"/></td>
            <td><img src="/Resources/Posts/TaggedPonted/19.png"/></td>
        </tr>
    </tbody>
</table>

> 到此，该问题解决完毕。如果哪里有不对的地方，希望大家给予指正。



