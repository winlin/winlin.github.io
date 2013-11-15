---
layout: post
title: "The base knowledge of iOS Development"
description: "My collection of base knowledge for iOS"
category: iOS
tags: []
---
{% include JB/setup %}

1. 虚拟内存     
   iPhone OS并不会将易失性内存（如应用程序数据）写到交换文件，因此应用程序可用内存量将受到更多的限制。
    Cocoa Touch提供一种内置机制，可以将内存不足的情况通知给应用程序。
2. nib文件构成   
    File's Owner是所有nib文件中的第一个图标，它标示从磁盘加载nib文件的对象。即，File's Owner是“拥有”此nib文件的对象。  
    First Responder可理解成用户当前正在与之交互的对象。例如，如果用户当前正在textfield中输入数据，则该textfield就是当前的First Responder。   
    First Responder将随着用户与界面的交互而变化。它的IB属性为placeholders，这意味这它属于一个虚拟实例，就不如textfield的string placeholders一样只是临时显示一下。真正的First Responder会被其他对象代替。实际上，任何派生自NSResponder类的对象都可以作为First Responder。而First Responder里面所有的Action就是NSResponder提供的或者自定义的响应函数。   
    MacOS在系统内部会维护一个称为“The Responder Chain”的链表。该列表内容为Responder对象实例，它们会对各种系统事件做出响应。最上面的那个对象就叫做First Responder，它是最先接收到系统事件的对象。如果该对象不处理改事件，系统会将这个事件向下传递，直到找到响应事件的对象，我们可以理解为该事件被该对象截取。  
    The Responder Chain基本结构如下图所示：  
    ![](/images/ios_responder_chain.png)

3. 线程安全     
 	Mutable container is not thread safe  
    "不要在后台线程更新你的UI"。其实，这个说法并不严密。首先需要把“UI更新”这个词做一个说明，它可以有2个层次的理解:  
    首先是绘制，其实是显示。绘制是可以放在任何线程里进行的，但是要显示出来就必须在主线程操作了。比如，对一个图片添加一个变色滤镜，这个过程就可以看成是绘制。Twitter客户端会把一条微博显示成一个cell，但是速度非常快，就是因为先对cell做了offscreen的渲染，然后再显示。
  
4. Subclass 和Category    
   先说一下这2个特性最主要的区别。简单可以这么理解，subclass体现了类的上下级关系，而category是类间的平级关系。  
    ![](/images/categary_subclass.png)   
 	
    Category methods should not override existing methods (class or instance).  
    Two different categories implementing the same method results in undefined behavior.    
    category不仅可以为原有class添加方法，而且如果category方法与类内某个方法具有同样的method signature，那么category里的方法将会替换类的原有方法。  
    这是category的替换特性。利用这个特性，category还可以用来修复一些bugs。例如已经发布的Framework出现漏洞，如果不便于重新发布新版本，可以使用category替换特性修复漏洞。另外，由于category有run-time级别的集成度，所以使得cocoa程序安全性有所下降。   
    许多黑客就是利用这个特性（和posting技术2）劫持函数、破解软件，或者为软件增加新功能。
    
5. drawing Issues   
大家都知道，MacOS是一个非常注重UI的系统。所以在MacOS编程里绘制是一个非常重要的部分。第10部分，我会从2点介绍MacOS下绘制编程。首先是绘制技术分类；其次是绘制代码结构。 
从绘制技术分类上看，Cocoa程序员能接触的几种绘制技术列表如下：

 		Cocoa Drawing(NS-prefix)  
 		Core Graphics(CG-prefix, called Quazrtz 2D)    
 		Core Animation    
 		Core Image   
 		OpenGL 
 
###Cocoa Drawing 
Cocoa Drawing应该是学习Cocoa程序开发最先接触的绘制技术。也是目前大多数MacOS程序所使用的绘制技术，其底层使用Quazrtz 2D(Core Graphics)。苹果对应文档为 [Cocoa Drawing Guide](https://developer.apple.com/library/mac/#documentation/Cocoa/Conceptual/CocoaDrawingGuide/Introduction/Introduction.html)。   
Cocoa Drawing并没有统一的绘制函数，所有绘制函数分散在几个主要的NS类的下面。例如, NSImage, NSBezierPath, NSString, NSAttributedString, NSColor, NSShadow，NSGradient … 
所以很简单，当你看到如下代码就可以判断，使用的是Cocoa Drawing方法:   

{% highlight objective-c %}
[anImage drawInRect:rect fromRect:NSZeroRect operation:NSCompositeSourceOver fraction:1.0];
[@"some text" drawAtPoint:NSZeroPoint withAttributes:attrs];
NSBezierPath *p=[NSBezierPath bezierPathWithRect:rect];
[[NSColor redColor] set];
[p fill];
{% endhighlight %}   
 
这种代码多出现在NSView的drawRect函数内。Cocoa Drawing 的渲染上下文是 NSGraphicsContext，我不断的看到很多新手把 NSGraphicsContext 和 CoreGraphics 的 CGContextRef 搞混。虽然它们很像并且也确实是有关系的，不过如果你不了解当绘制时候的 render context 很多时候将得到一个空白页面的结果。 

###Core Graphics 
Core Graphics 是 Cocoa Drawing layer 的底层技术，在 iOS 开发中非常普遍，因为 iOS 系统中并不存在 Cocoa layer 所以网上可以找到的多是 Core Graphics 绘制代码段子，这给那些不了解 Mac 开发的新手来说造成了很大困扰。  
Cocoa 是 Mac OS 下的 application framework 而 iOS 下的 application framework 则是 UIKit.framework又叫 Cocoa Touch，它们分享部分代码基础但又不完全一样。例如，Cocoa Touch 下的 UIView 的渲染上下文会使用 UIGraphicsGetCurrentContext() 取得，它得到的是一个 CGContextRef 指针，而在 NSView 里多用 [NSGraphicsContext currentContext] 取得渲染上下文。它得到的是一个 NSGraphicsContext 对象。当然 NSView 里也可以通过    

	CGContextRef ctx = [[NSGraphicsContext currentContext] graphicsPort];    

来取得一个 Core Graphics 渲染上下文。 可见 Mac OS 下的开发更为灵活一些。因为 iOS 中的 UIKit 开发初期就瞄准了显卡硬件加速，所有 UIView 都是默认 layer-backed 的。iOS 开发者必须使用 Core Graphics 和 Core Animation 这几个相对底层的绘制技术。   
请看下面等价代码，作用是绘制一个白色矩形。但是分别使用 Core Graphics 和 Cocoa Drawing: 
{% highlight objective-c %}
const CGFloat white[]={ 1.0, 1.0, 1.0, 1.0 };
CGContextSetFillColor(cgContextRef, white);
CGContextSetBlendMode(cgContextRef, kCGBlendModeNormal);
CGContextFillRect(cgContextRef, CGRectMake(0, 0, width, height));
[[NSColor whiteColor] set];
NSRectFillUsingOperation(NSMakeRect(0, 0, width, height), NSCompositeSourceOver);
{% endhighlight %}   

可以看出，这是2种*风格完全不同*的绘制技术。Cocoa Drawing 是分散式的绘制函数，而 Core Graphics 是传统的类似 OpenGL 的集成式的绘制方式。其实 Cocoa Drawing 下层是 Core Graphics， Core Graphics 的下层是 OpenGL。  

###Core Animation 
如果说 Core Graphics 和 Cocoa Drawing 是通用的 UI 绘制框架的话，那么 CA 显然是界面动画绘制的高级技术。 Core Animation 的对应 Cocoa Animation 部分应该是 NSAnimation 和 NSViewAnimation，但这2个差距比较大。NSAnimation 出现与 OS X 10.4，Core Animation 是 10.5 后出现的。NSViewAnimation 功能和使用相对简单。  
简单来说，Core Animation 的作用对象是 CALayer, NSAnimation 的作用对象是 NSView。 

###Other
6. MVC
MVC模型将所有功能划分成3个不同类别:   
	模型：保存应用程序数据的类   
    视图：窗口、控件和其他用户可以看到并能与之交互的元素的组成部分   
    控制器：将模型和视图绑定在一起，确定图和处理用户输入的应用程序逻辑   
编写的任何对象都应该能很明显地划分为其中的一类，并且其功能大部分不属于或完全不属于另外两类。
MVC可以帮助确保实现最大可重用性。

7. 自动旋转屏
    分别针对不同的设备屏幕状态，设计两个UIView 关联到同一个xib文件中，然后都创建输出口到ViewController里面，这样在监测到屏幕变换时，直接设定不同View的hidden属性，这样就避免了自己调整控件位置。

8. 多视图   
	多视图应用程序都使用相同的基本模式。
    常见的有UINavigationController或UITabBarController，还有UIViewController自定义的子类。  
    需要重点注意的是，多视图控制器也是一个视图控制器。Navigation和TabBar都是UIViewController的子类，并且能够执行其他视图控制器能够执行的所有工作。  
    根控制器是应用程序的主要视图控制器，也是指定是否应该自动旋转到新方向的视图。
    
    内容视图剖析：
    每个视图控制器（包括多视图控制器）都控制一个内容视图，应用程序的用户界面就在这些内容视图中构建。  
    每个内容视图通常由2或3个部分组成：视图控制器、nib文件以及一个可选的UIView子类。

9. NSBundle   
	NSBundle只是一种特定的文件类型，其中的内容遵循特定的结构。应用程序和框架都是束NSBundle。  
    NSBundle的一个主要作用是获取添加到项目的Resources文件夹的资源。在构建应用程序的时候，这些文件将被复制到应用程序的束中。  
    比如，使用mainBundle来获取需要的资源路径：  
  
  		NSString *plistPath = [[NSBundle mainBundle] pathForResource:@"statedictionary", ofType:@"plist"];
    	NSDictionary *dictionary = [[NSDictionary alloc] initWithContentsOfFile:plistPath];

10. 首选项
    用户默认设置时应用程序首选项的一部分，由NSUserDefault类实现，用于保存和获取首选项。
    首选项存储在Library/Preferences文件夹中。  
    
11. 对模型对象进行归档   
     在Cocoa世界中，术语“归档”是指另一种形式的序列化，但是它是任何对象都可以实现的更常规的类型。专门编写用于保存数据（模型对象）的任何对象都应该支持归档。只要在类中实现的每个属性都是标量（如int或float）或者都是符合NSCoding协议的某个类的实例，你就可以对你的对象进行完整的归档。  
    由于大多数支持存储数据的Foundation和Cocoa Touch类都符合NSCoding，因此比较容易实现。
    尽管对使用归档没有严格要求，但是应该与NSCoding一起实现另一个协议，即NSCopying协议，该协议允许复制对象。
    
12. 基本数据持久性   
  方式：首选项、文件、归档、SQLite

13. 使用Quartz和OpenGL绘图  
  我们可以依靠两个不同的库来满足我们的绘图需要：Quartz 2D，它是Core Craphics框架的一部分，OpenGL ES，它是跨平台的图形库。   
  尽管Quartz 和 OpenGL有许多共性，但它们之间存在明显差别。   
  Quartz是一组函数、数据类型以及对象，专门设计用于直接在内存中对视图或图像进行绘制。  
  尽管在计算机图形中最常用的是RGB模型，但是它不是唯一的颜色模型。其他一些模型也得到了使用，包括：   
  		色调、饱和度、值(HSV)   
  		色调、饱和度、亮度（HSL）   
  		蓝绿色、洋红色、黄色、黑色（CMYK）
    
    尽量避免整个视图重新绘制，而是根据需要，调用setNeedsDisplayInRect:来进行局部重绘。减少重新绘制视图的大量工作，可以在应用程序性能方面产生巨大差别，尤其是当应用程序变得更加复杂时。
        
14. 轻击、触摸和手势    
  手势gesture是指从你用一个或多个手指接触屏幕时开始，直到你的手指离开屏幕为止所发生的多有事件。无论它话费了多长事件，只要一个或多个手指仍在屏幕上，你就仍然位于某个手势之中（除非传入电话呼叫等系统事件中断该手势）。  
    iPhone只跟踪使用一个手指时的轻击，记住这一点非常重要。如果她检测到多个触摸，则会将轻击计数重置为1.  
    
    响应者链 responder chain  
    由于手势是在时间之内传递到系统的，然后事件会传递到响应者链。如果第一个响应者不处理某个特殊事件，则她会将改时间传递到响应者链的下一级。
    
15. Core Location定位    
  iPhone可以使用Core Location framework来确定它的物理位置。Core Location可以利用三种技术来实现该功能：  
	GPS、蜂窝基站三角定位、wifi定位服务。   
    GPS是三种技术中最精确的，但是在第一代iPhone上不可用。我们只需要传入精确度，不需要指定采用方式。系统本身会进行确定。  
    Cocoa Touch使用的主要类是：CLLocationManager。
    请记住你要求的精确度越高，消耗的电量就会越多；并不能保证你会获得所需精度级别。  
    通过指定距离筛选器可以告知位置管理器不要将每个更改都通知其代理，仅当位置更改超过特定数量是才通知其代理。 如果只是需要确定当前位置而不是需要连续轮询位置，则当它获取应用程序所需要信息之后，应该停止位置管理器。
    获取越精确的位置，就需要更多的电量。

16. 加速计  
  	通过感知特定方向的惯性力总量，加速计可以测量出加速度和重力。   
  	iPhone内的加速计是一个3轴加速器，这意味着它能够检测到三维空间中的运动或重力引力。 使用的类为 UIAccelerometer。
    
17. 应用程序本地化
    