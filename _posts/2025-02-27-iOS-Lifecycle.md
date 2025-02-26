---
title: "iOS Lifecycle"
date: 2025-02-27 00:50:00 +0800
categories: [iOS,Lifecycle]
tags: [iOS, Lifecycle]
---
## APP 生命周期

#### 应用程序的五种状态：

- 未运行状态（Not running）：程序尚未启动，或者应用正在运行但是中途被系统停止，当设备内存紧张的时候，也会将挂起的应用当前状态写入到内存，然后退出应用并释放内存，这时候我们虽然能够在任务栏看到图标，但是它已经退出，我们称之为应用墓碑。
- 未激活状态（Inactive）：当前应用正在前台运行，但是焦点被其他抢去。比较典型的是用户锁屏或者离开应用去响应来电，信息等事件等时候。还有一种是比较常见的就是在不同状态切换的时候会短暂处于该状态,这时候App会停止运行，但是依然占用内存空间，用于保存当前状态。
- 激活状态（Active）：当前应用正常运行，应用焦点在当前应用上，所有的事件都会被分发到当前应用，这时应用占用内存和CPU时间。
- 后台状态（Backgroud）：程序在后台而且能执行代码，大多数程序进入这个状态后会在在这个状态上停留一会。时间到之后会进入挂起状态(Suspended)。有的程序经过特殊的请求后可以长期处于Backgroud状态
- 挂起状态（Suspended）: 应用处在后台，并且已经停止执行代码。这时候应用还驻留在内存中，并没有被系统完全回收，只有在系统发出低内存告警的时候，系统才会把处于挂起状态的应用清除出内存给前台正在运行的应用。这时候不占用CPU资源，但是内存依然占用。

#### App 生命周期的切换会通过AppDelegate来通知开发者，下面是一些常见方法：

```objectivec
// 告诉代理进程启动但还没进入状态保存
- (BOOL)application:(UIApplication *)application willFinishLaunchingWithOptions:(NSDictionary *)launchOptions

// 启动基本完成,程序准备开始运行
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions

// 当应用程序将要进入非活动状态执行，在此期间，应用程序不接收消息或事件，比如打来电话
- (void)applicationWillResignActive:(UIApplication *)application

// 当应用程序进入活动状态执行，此方法跟上面那个方法相反
- (void)applicationDidBecomeActive:(UIApplication *)application

// 当程序被推送到后台的时候调用。所以要设置后台继续运行，则在这个函数里面设置即可
- (void)applicationDidEnterBackground:(UIApplication *)application

// 当程序从后台将要重新回到前台时候调用，此方法跟上面的那个方法相反
- (void)applicationWillEnterForeground:(UIApplication *)application

// 当程序将要退出是被调用，通常是用来保存数据和一些退出前的清理工作
- (void)applicationWillTerminate:(UIApplication *)application 
```

## UIViewController生命周期

生命周期方法介绍

1. alloc
   创建对象，分配空间
2. init (initWithNibName|initWithCoder)
   初始化对象，初始化数据
   nitWithCoder:反归档,如果对象是从文件解析来的就会调用
   initWithNibName:使用代码创建对象的时候会调用这个方法
3. awakeFromNib
   所有视图的outlet和action已经连接，但还没有被确定。awakeFromNib:从xib或者storyboard加载完毕会调用。
4. loadView
   完成一些关键view的初始化工作，加载view。当你alloc并init了一个ViewController时，这个ViewController是还没有创建view的.
   loadView用于加载控制器管理的 view，不能直接手动调用该方法
5. viewDidLoad
   载入完成，可以进行自定义数据以及动态创建其他控件
6. viewWillAppear
   视图将出现在屏幕之前
7. updateViewConstraints
   在该函数中用于更新视图的约束.在控制器的view更新视图布局时，会调用updateViewConstraints函数,可以重写这个函数来更新当前视图的布局.这个函数只有在Autolayout布局的时候才会被调用。初始化约束时，最好写到init或viewDidLoad类似的函数中，updateViewConstraints适合于更新约束
8. viewWillLayoutSubviews
   将要对子视图进行调整,该方法在通知控制器将要布局 view 的子控件时调用。
   在这个函数中布局子视图，如果用了Autolayout,那么会在viewWillLayoutSubviews和viewDidLayoutSubviews之间用Autolayout机制布局，但是需要注意的是该方法调用时，AutoLayout 未起作用。
9. viewDidLayoutSubviews
   对子视图进行调整完毕,控制器的子视图的布局已完成，这里获取的frame才是最正确的frame。如果用约束来布局，在该函数去设置视图的frame 是无效的。如果用frame来布局的，在该函数中去设置视图的frame是有效的。self.view在该函数中去设置frame是有效的。该方法调用时，AutoLayout 已经完成。
10. viewDidAppear
    视图已在屏幕上渲染完成
11. viewWillDisappear
    视图将被从屏幕上移除
12. viewDidDisappear
    视图已经被从屏幕上移除
13. dealloc
    视图被销毁，此处需要对你在init和viewDidLoad中创建的对象进行释放
14. didReceiveMemoryWarning
    内存警告,当系统内存不足时，当前控制器以及所在的Navigation堆栈上的控制器都会调用didReceiveMemoryWarning函数.该函数会判断当前控制器的view是否显示在window上，如果没有会将view以及子view全部销毁.

#### extra:

在main函数执行前，会初始化objc运行时环境，这时会加载所有类并调用类的load方法，一般在这个方法中实现方法交换(Method Swizzle);

UIViewController的self.view是通过懒加载方式创建的，每次调用控制器的view属性时并且view为nil时，loadView函数就会被调用.加载成功后接着调用viewDidLoad函数，如果self.view已经是非空的情况下会直接调用viewDidLoad函数。

如果在loadView函数中自定义了view，那么xib. storyboard中对页面的设置会失效，因为它是在加载之后调用的，所以如果使用 Interface Builder创建view，则务必不要重写该方法

[super loadView]默认的逻辑:如果控制器由xib或storyboard初始化,那么会根据xib或storyboard的传入的名字来初始化view；如果没有显示的指定名称，就默认加载和控制器同名的文件；如果没有找到文件，就会创建一个空白的UIView，这个view的frame为屏幕的大小。所以在覆写该方法的时候不应该再调用父类的该方法。

ViewController init里不要调用self.view,一般在init里应该只有关键数据的初始化。
如果确实需要重写loadView，在重写的代码中只初始化view，其他的工作放在viewDidLoad方法中完成。
viewDidLoad 这时候view已经有了，可以创建并添加界面上的其他子视图，以及设置这些视图的属性。
viewWillAppear 这个一般在view被添加到superview之前，切换动画之前调用，一般可以用于弹出键盘，status bar和navigationbar颜色设置等。
viewWillLayoutSubViews/viewDidLayoutSubviews viewDidLayoutSubviews的时候frame值已经确定了，可以在这里做一些依赖frame的操作
