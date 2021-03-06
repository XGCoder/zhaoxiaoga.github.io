---
title: UIApplicaion解析
date: 2016-08-09 20:31:18
tags:
---

UIApplicaion 的简介:
UIApplication 的核心作用是提供了 iOS 程序运行期间的控制和协作工作

每一个程序在运行期必须有且仅有一个 UIApplication（或则其子类）的一个实例。

`UIApplicationMain` 函数是程序进入点，并且UIApplication 是一个单例实例。

你可以通过调用 `[UIApplication sharedApplication]` 来得到这个单例实例的指针。

<Excerpt in index | 首页摘要>
 <!-- more -->
<The rest of contents | 余下全文>

UIApplication 的一个主要工作是处理用户事件，它会起一个队列，把所有用户事件都放入队列，逐个处理，在处理的时候，它会发送当前事件 到一个合适的处理事件的目标控件。此外，UIApplication 实例还维护一个在本应用中打开的 window 列表（UIWindow 实例），这样它就 可以接触应用中的任何一个 UIView 对象。UIApplication 实例会被赋予一个代理对象，以处理应用程序的生命周期事件（比如程序启动和关闭）、系统事件（比如来电、记事项警告）等等。

### UIApplicaion 生命周期

一个 UIApplication 可以有如下几种状态：

* Not running -- > 程序没启动
* Inactive -- > 程序在前台运行，不过没有接收到事件
* Active -- > 程序在前台运行而且接收到了事件。这也是前台的一个正常的模式
* Background -- > 程序在后台而且能执行代码，大多数程序进入这个状态后会在在这个状态上停留一会。时间到之后会进入挂起状态 (Suspended)。有的程序经过特殊的请求后可以长期处于 Background 状态
* Suspended -- > 程序在后台不能执行代码。系统会自动把程序变成这个状态而且不会发出通知。当挂起时，程序还是停留在内存中的，当系统内存低时，系统就把挂起的程序清除掉，为前台程序提供更多的内存。

### 常使用的代理:

当应用程序将要入非活动状态执行，在此期间，应用程序不接收消息或事件:
```
- (void)applicationWillResignActive:(UIApplication *)application
```

当应用程序入活动状态执行，这个刚好跟上面那个方法相反:
```
- (void)applicationDidBecomeActive:(UIApplication *)application
```

当程序被推送到后台的时候调用。所以要设置后台继续运行，则在这个函数里面设置即可
```
- (void)applicationDidEnterBackground:(UIApplication *)application
```

当程序从后台将要重新回到前台时候调用
```
- (void)applicationWillEnterForeground:(UIApplication *)application
```

当程序将要退出时被调用，通常是用来保存数据和一些退出前的清理工作。这个需要设置 UIApplicationExitsOnSuspend 的键值。
```
- (void)applicationWillTerminate:(UIApplication *)application
```

iPhone 设备只有有限的内存，如果为应用程序分配了太多内存操作系统会终止应用程序的运行，在终止前会执行这个方法，通常可以在这里进行内存清理工作防止程序被终止
```
- (void)applicationDidReceiveMemoryWarning:(UIApplication *)application
```

当系统时间发生改变时执行
```
- (void)applicationSignificantTimeChange:(UIApplication*)application
```

当程序载入后执行
```
- (void)applicationDidFinishLaunching:(UIApplication*)application
```
