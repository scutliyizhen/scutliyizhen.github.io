---
layout:     post
title:      从数学角度来看iOS中的AutoLayout布局
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

>  本问主要介绍对AutoLayout的深入理解，2015年写于QQ空间中。 

###  一.传统布局与autoLayout布局区别
####  1.frame 与 autoresizing 概念
<table>
    <tbody>
        <tr>
            <td><img src="/Resources/Posts/auotlayout/1.png"/></td>
        </tr>
    </tbody>
</table>
**<font style="color:#0F7290">（1）frame布局</font>**  
对于UIView采用frame布局的方式是大家最熟悉也是最直接容易理解的，比如：  
UIView* v1 = [[UIView alloc] init];  
v1.frame = CGRectMake(100,100,70,65);  
如上图所示。这段代码吧v1的位置以及长宽都给出了定义值，则系统会根据这个逻辑点绘制视图v1在其父视图中的位置。  
**<font style="color:#0F7290">（2）autoresizing</font>**      
@property(nonatomic)UIViewAutoresizing autoresizingMask;   
typedefNS_OPTIONS(NSUInteger, UIViewAutoresizing)  
{
     UIViewAutoresizingNone =0,//控件相对于父视图坐标值不变  
    UIViewAutoresizingFlexibleLeftMargin =1 << 0,//到屏幕左边的距离随着父视图的宽度按比例改变；  
    UIViewAutoresizingFlexibleWidth =1 << 1,//控件的宽度随着父视图的宽度按比例改变；  
    UIViewAutoresizingFlexibleRightMargin =1 << 2,//到屏幕右边的距离随着父视图的宽度按比例改变；  
    UIViewAutoresizingFlexibleTopMargin =1 << 3,//到屏幕上边的距离随着父视图的高度按比例改变；   
    UIViewAutoresizingFlexibleHeight =1 << 4,//控件的高度随着父视图的高度按比例改变； 
    UIViewAutoresizingFlexibleBottomMargin =1 << 5//到屏幕底边的距离随着父视图的高度按比例改变；  
};   
这个属性均相对于父视图。  

举例如下:  
vi.super.frame = CGRectMake(0,0,320,320);  
v1.frame = CGRectMake(100,100,70,65);  
v1. autoresizingMask = UIViewAutoresizingFlexibleWidth | UIViewAutoresizingFlexibleLeftMargin;  

如果v1的父视图宽度变为480时，那v1的frame是多少？  
比例ratio = 480/320;  
所以v1.frame会变为[150,100,105,65]   

所以这个属性的设置就解决了父view大小发生变化时，子视图可以灵活改变相对父视图间距、大小的问题。 既然autoresizingMask  
可以解决子视图随父视图变化的问题，那为什么还要有autoLayout自动布局呢？他们又有什么区别？  
v1.frame=[100,100,70,65]  v2.frame = [190,100,70,65];   
 
首先先来考虑 两个很简单的问题：（以图一为例）  
1). v2在v1的右边，且相距20，那如果仅仅改变v1的x坐标（其他条件均不变）由100变为120，还要v2保持与v1之前的相对位置，也就是v1.frame=[120,100,70,65],v2是否可以通过autoresizingMask自动改变为v2.frame=[210,100,50,50](假设v2的宽度高度均为50)呢？   

答案肯定是否定的，autoresizingMask肯定不能解决这个问题，因为这个属性解决的是v1与父视图的相对情况，而不能解决兄弟、以及叔侄View等之间的相对关系。  

2).还是以vi.super.frame = CGRectMake(0,0,320,320)为例，当v1父视图的宽度由320变为480时，设置vi.autoresizingMask=UIViewAutoresizingFlexibleLeftMargin(其他条件不变)，是否可以通过设置v2的autoresizingMask属性来自动改变v2继续保持与在v1右边距离20处的位置呢？  
 分析：父视图的宽度变为480以后，v1的x坐标变为150，我们希望v2的x坐标应该变为150+70+20=230。  
v2起始x坐标为 190,目标x坐标为230， 那么230/190=1.21.    
但是仅通过设置v2.autoResizingMask=UIViewAutoresizingFlexibleLeftMargin，显然是不能实现的，因为如果这样设置v2的x坐标将会由190变为285，即285/190=480/320=1.5。
明显仅仅通过设置这个属性是不能满足要求的。     

如果设备发生旋转、适配ipad等，并且保证视图原来之间的相对关系，则以上的方法都是无法解决的。如果要做这些适配，在autoLayout未出来之前需要编写大量的代码，并且花费大量的调试适配时间。  

为了解决以上问题，AutoLayout诞生了。   

####  2.autoLayout概念
从英语单词字面的意思理解就是“自动布局”。那怎么才算是“自动”布局呢？那以前的frame,autoresizingMask到底算不算“自动”布局？我个人的理解是:   
1).frame肯定不是“自动”布局，因为 View的坐标与长宽都是写死的，当父视图或者屏幕发生变化时，view在其父视图中的相对位置或者相对大小都不会跟随发生变化，此时当其父view或者兄弟view发生变化时，其之前的相对关系不能很好的保留，不能 达到我们想要的布局效果。    
2).autoresizingMask，应该可以说是半自动化得布局方式，之所以说是“半”自动化，因为这种布局方式在一定程度上解决了view跟随其父view的变化而自动布局。比如当父view的宽度增大的时候，子view只要设置UIViewAutoresizingFlexibleWidth，子view就能按照比例跟随父view放缩。但是它的功能是很受限制的，它不能解决屏幕发生偏转、兄弟视图之间相对关系保持不变的布局。因此说autoresizingMask只能说是半自动化的布局，不过应该说“半”自动化都算不上。它的功能与autoLayout是远远不能提并论的。    
3).Autolayout是iOS6引入的新特性,随着iOS设备尺寸逐渐碎片化，纯粹的hard code方式UI布局将会走向死角，而autoresizing方式也有其局限性，因此无论如何autolayout都将成为IOS开发中 UI布局的重要方式。    
AutoLayout给开发者提供了一种全新的界面布局方式，自动布局是一个强大的、灵活的描述性系统。它描述了视图和他们的内容是如何关联，他们和他们所占据的窗口或者父视图之间如何关联，视图与兄弟视图等之间是如何关联。这种布局方式完全是通过视图之间的“联系”来描述视图的布局，通过解析这些关联的关系来布局每一个视图，这样当一个视图发生变化时，只要关联关系规则不发生变化，那么其他视图也会跟随发生变化，继续与这个发生变化的视图保持原来的关系，这就达到了自动布局的效果。    
个人对AuatoLayout的理解，其本质是开发者描述了视图之间的“关系”，则自动布局系统根据这些“关系”计算视图的frame。因此，最终视图的位置与大小还是由frame来决定，只不过是视图的frame不再是由开发者用硬编码的方式写死，而是根据关系动态被计算。   
AutoLayout  是一个强大的描述性系统，它的特点主要有：   
- **<font style="color:#0F7290">（1）声明性的</font>**   
也就是开发者描述界面时不需要关心规则是如何实现的，只要描述界面的布局即可，视图的frame由AutoLayout来计算。  
- **<font style="color:#0F7290">（2）集中性的</font>**  
开发者不管是在IB创建布局，还是用代码去创建界面布局。AutoLayout规则倾向于迁移到简单的关系上来，这样便于调试和检查。  
- **<font style="color:#0F7290">（3）描述性和相关性</font>**   
开发者需要描述项在屏幕上是如何关联的，忘记尺寸和位置硬编码的布局方式，重要的是视图之间的“关系”，“关系”是自动布局AutoLayout的核心。  
- **<font style="color:#0F7290">（4）动态的</font>**   
应用的界面 会在响应用户或者源自应用的改变时而更新。  
- **<font style="color:#0F7290">（5）本地化的</font>**    
《AutoLayout开发秘籍》中对这个特性的描述是：使用AutoLayout可以征服世界，它在维护界面完整性时，适应不同的单词或词组长度。  
**<font style="color:#0F7290">（6）表达性的</font>**  
使用AutoLayout可以表达比 旧的spring-strut更多更复杂的关系。自动布局不仅仅可以“吸附这条边”、“沿着坐标轴改变尺寸大小”，它还可以表示一个视图与另一个视图的关联方式，不仅仅只是视图与父视图的关系。   
**<font style="color:#0F7290">（7）增量式的</font>**   
开发者可以根据自己的时间表随时使用AutoLayout, 因为自动布局开发者可以在项目中完全使用spring-strut,  或者完全使用自动布局，或者是两者的混合使用。 

####  3.对比优缺点   
**<font style="color:#0F7290">AutoLayout布局缺点：</font>**  
（1）.Frame布局简单易学，容易掌握。然而AutoLayout布局的具有一定的学习难度，初学者不容易掌握。   
（2）.Frame布局代码量要比自动布局代码量少，由于AutoLayout采用约束布局，要想更新布局一般要移除旧的约束，重新安装新的约束；并且由于每个约束都有占有一定的代码量，所以自动布局代码量要要大点。  
**<font style="color:#0F7290">AutoLayout布局优点：</font>**  
（1）.自动布局是一个很强大的布局描述系统，它能够将视图与其父视图、试图与其具有相同祖先的视图的关系很好的描述出来，根据关系布局视图，因此布局可以自动发生变化。Frame属于硬编码布局，布局不能够自动改变。随着屏幕尺寸的改变、屏幕旋转等都给UI布局带来了更大的挑战。  
（2）.使得布局更加简单，提高开发者的开发效率。  
（3）.AutoLayout有着更加强大的功能，除了自动布局之外，AutoLayout还与很多优秀的变成API接口兼容，比如动画，动画效果等，它的强大功能是Frame无法比拟的。  

###  二.数学抽象AutoLayout
<table>
    <tbody>
        <tr>
            <td><img src="/Resources/Posts/auotlayout/2.png"/></td>
        </tr>
    </tbody>
</table>

####  1.举例传统布局原理(v1,v2,v3的父视图为v)  
如图一，V1布局如下：  
v1.frame = CGRectMake(100,100,70,65)，系统会根据此框架将v1视图绘制到屏幕上，位于其父视图坐标（100,100）逻辑点处，大小为（70.65）。  
 
####  2.举例autoLayout解释原理  
使用VL语言描述视图的约束：  
(1).[v addConstraints:[NSLayoutConstrain constraintsWithVisualFormat:@“H:|-100-[v1(70)]-20-[v2(v1)]-(>=0)-|” options:NSLayoutFormatAlignAllTop metrics:nil views:NSDictioaryOfVariableBindings(v1,v2)]];      
(2). [v addConstraint:[NSLayoutConstraint constrintWithItem:v2 attribute:NSLayoutAttributeHeight relatedBy:NSLayoutRelationEqual toItem:v1 attribute:NSLayoutAttributeHeight multiplayier:1.0 constant:0]];      
(3).[v addConstraints:[NSLayoutConstrain constraintsWithVisualFormat:@"V:|-100-[v1(65)]-20-[v3(v1)]-(>=0)-|" options:NSLayoutFormatAlignAllLeading metrics:nil views:NSDictioaryOfVariableBindings(v1,v3)]];       
(4).[v addConstraint:[NSLayoutConstraint constrintWithItem:v3 attribute:NSLayoutAttributeTrailling relatedBy:NSLayoutRelationEqual toItem:v2 attribute:NSLayoutAttributeTrailling multiplayier:1.0 constant:0]];    
 
####  3.通过举例抽象出自动布局数学公式  
注：视图的frame参数描述如下：   
x坐标：v.x   
y坐标：v.y  
宽度：v.width  
高度：v.height  

由以上三个视图约束，v1,v2,v3视图Frame的描述如下：  
v1视图：    
v1.x = 100;由（1）中的“H:|-100-”，v1视图水平方向上距离其父视图100逻辑点处  
v1.y = 100;由（3）中的“V:|-100-”，v1视图垂直方向上距离其父视图100逻辑点处  
v1.width = 70;由（1）中的[v1(70)]  
v1.height = 65;由（3）中的[v1(65)]  

v2视图：  
v2.x = v1.x + v1.width + 20;由（1）中“-20-[v3”  
v2.y = v1.y;由（1）中NSLayoutFormatAlignAllTop  
v2.width = v1.width;由(1)中[v2(v1)]  
v2.height = v1.height;由（2）得  

v3视图：  
v3.x = v1.x;由（3）中NSLayoutFormatAlignAllLeading  
v3.y = v1.y + v1.height + 20;由（3）中“-20-[v3”  
v3.3.x + v3.width = v2.x + v2.width;由（4）得  
v3.height = v1.height;由（3）中[v3(v1)]  

将以上视图的Frame参数抽象成数学公式如下：  
注：参数描述如下  
v1.x 表示成x1;  
v2.x表示成x2;  
v1.y表示成y1;  
v1.width 表示成m1;   
v1.height表示成n1;  
其他视图的Frame参数依次如上描述。  
<table>
    <tbody>
        <tr>
            <td><img src="/Resources/Posts/auotlayout/3.jpeg"/></td>
        </tr>
    </tbody>
</table>
将以上等式变形为：  
<table>
    <tbody>
        <tr>
            <td><img src="/Resources/Posts/auotlayout/4.jpeg"/></td>
        </tr>
    </tbody>
</table>
此时，以上方程组，大家肯定很熟悉了，也就是《线性代数》中的线性方程组，现在将以上线性方程组抽象为：  
<table>
    <tbody>
        <tr>
            <td><img src="/Resources/Posts/auotlayout/5.jpeg"/></td>
        </tr>
    </tbody>
</table>
上图表示“等式”方程组，那么是否还可以继续抽象？也就是说上述方程组能否完全表示未知元素之间与已知元素之间的关系，显然还不全面，因为还有（<,>,<=,>=）不等关系，因此将“=”等号抽象为关系"R",在数学上关系R也就包括了“=”,"<",">","<=",">="等关系。上述线程方程组变形为：（实质上，AutoLayout中所有的约束确实都是用数学关系式y R ax + b描述）    
<table>
    <tbody>
        <tr>
            <td><img src="/Resources/Posts/auotlayout/6.jpeg"/></td>
        </tr>
    </tbody>
</table>现在已经将自动布局一步步抽象为数学公式，那么对视图的布局其实就是对线性方程组的求解。因此，AutoLayout自动布局系统的对视图布局的过程实质上就是对线性方程组进行求解的过程。 
线性方程组解的情况有三种，实质上也对应着自动布局对视图的三种布局方案:  
**<font style="color:#0F7290">（1）唯一解：所有未知数由方程解出唯一解</font>**   
充分约束：给一个视图添加的约束必须是充分的，才能正确布局一个视图。    
**<font style="color:#0F7290">（2）多个解：未知数不能求解出准确的唯一解</font>**即未知数可能存在多个或者无限个解满足线性方程组。    
欠约束：给视图所添加的约束不能够充分的表达视图的准确位置，在这种情况下自动布局会随意给视图一个布局方案，也就是自动布局中视图不能够正确布局或者视图丢失的情况。     
**<font style="color:#0F7290">（3）无解：不存在解满足线性方程组</font>**  
冲突约束：给视图添加的约束表达视图布局出现了冲突，比如同时满足同一个视图宽度即为100又为200，这是不可能存在的。此时程序会出现崩溃。通过以上描述，将AutoLayout系统的作用描述如图所示（在第三部分会给出自动布局描述系统在计算机上的工作原理）：   
<table>
    <tbody>
        <tr>
            <td><img src="/Resources/Posts/auotlayout/7.jpeg"/></td>
        </tr>
    </tbody>
</table>

###   三. 自动布局运行原理
####  1.AutoLayout的由来
AutoLayout是苹果公司于2012年首次曝光于ios6系统中，自动布局出现的主要目的就是取代之前基于spring和strut的autosizing的布局系统，自动布局系统是一个描述性系统，用来构建视图与父视图、视图与其他视图之间的关系。因此，在AutoLayout中最重要的就是“关系”。   
AutoLayout的前身是Cassowary约束解析工具包。根据Cassowary维基百科(https://en.wikipedia.org/wiki/Cassowary_(software))给出的简介如下：  
Cassowary was developed by Greg Badros and Alan Borning, and was optimized for user interface applications.    
Cassowary是由华盛顿大学的Greg Badros 和 Alan Borning博士为了优化用户界面布局应用程序而开发的。   
Cassowary is an incremental constraint solving toolkit that efficiently solves systems of linear equalities and inequalities. Constraints may be either requirements or preferences. Client code specifies the constraints to be maintained, and the solver updates the constrained variables to have values that satisfy the constraints.   
Cassowary是一种递增式约束解析工具包，它用来有效的解析线性相等与线性不等系统。约束可能是一种需求或者是一种偏好。客户端代码指定维护的约束，解析器更新约束变量，并且给出这些变量的值，用来满足约束。（翻译很菜.....）   
具体的关于递增式、约束需求还是偏好这些概念可以在论文《The Cassowary linear arithmetic constraint solving algorithm》    （见附件）中得到解释，这篇论文主要是讲解Cassowary对线性相等与线性不等关系系统解析的优化。  
对该论文中的内容解释，来自http://cassowary.readthedocs.org/en/latest/topics/theory.html中的一段文字总结的很好，如下：
Constraint solving systems are an algorithmic approach to solving Linear Programming problems. A linear programming problem is a mathematical problem where you have a set of non- negative, real valued variables (x[1], x[2], ... x[n]), and a series of linear constraints (i.e, no exponential terms) on those variables. These constraints are expressed as a set of equations of the form:    
a[1]x[1] + ... + a[n]x[n] = b,  
a[1]x[1] + ... + a[n]x[n] <= b, or  
a[1]x[1] + ... + a[n]x[n] >= b,   
Given these contraints, the problem is to find the values of x[i] that minimizes or maximizes the value of an objective function:   
c + d[1]x[1] + ... + d[n]x[n]   
Cassowary is an algorithm designed to solve linear programming problems of this type. Published in 1997, it now forms the basis fo the UI layout tools in OS X Lion, and iOS 6+ (the approach known asAuto Layout). The Cassowary algorithm (and this implementation of it) provides the tools to describe a set of constraints, and then find an optimal solution for that set of constraints.    
简单描述就是：约束解析系统是针对已经给定的一组约束，给变量找出可能的值，使得目标函数最大或者最小。而 Cassowary是对约束解析系统的优化，   Cassowary算法为描述一组约束提供了工具，并且为这组约束找出最优的解决方案。   

####  2.开源Cassowary的使用
使用步骤不讨论，可以在http://cassowary.readthedocs.org/en/latest/topics/theory.html阅读卡开发doc,这里仅仅是采用开发文档中的例子来说明自动布局原理：   
代码如下（python）：  
 from cassowary import SimplexSolver,  
 Variablesolver = SimplexSolver()  
 left = Variable('left')  
 middle = Variable('middle')  
 right = Variable('right')  
 solver.add_constraint(middle == (left + right) / 2)  
 solver.add_constraint(right == left + 10)  
 solver.add_constraint(right <= 100)   
solver.add_constraint(left >= 0)   
运行结果如下：  
 >>> left.value90.0  
 >>> middle.value95.0  
 >>> right.value 100.0   

 3.通过Cassowary验证AutoLayout的运行原理   
 针对第2点的使用做如下解释来阐述AutoLayout的运行原理。  
（1）.第2点钟的代码描述是一维度的布局（水平），其他维度的同样也是。  
（2）.从运行结果与代码给的一组约束来看，满足该约束的有多个结果，Cassowary给出了一个可能的最优解。因此，在自动布局中，如果是欠约束，那么视图的布局则是随意的，或者是出现视图丢失的情况。  
（3）.AutoLayout是根据给视图添加各个维度的约束来布局的，因此自动布局框架其实是对Cassowary的一种封装。开发者对视图添加的各种约束，提交给自动布局系统之后，由AutoLayout自动布局系统为添加的一组约束寻找满足这组约束的最优方案（位置或者大小）。  

综上所得，AutoLayout 其实是使用Cassowary算法根据为视图添加的约束，给视图提供一个最优的frame解决方案，从而达到自动布局的目标。  

 






