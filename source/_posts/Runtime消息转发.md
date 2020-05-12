---
title: Runtime消息转发
date: 2016-07-26 20:03:22
tags:
---

对于运行时的简单剖析,查看上一篇即可,这里查看runtime消息的转发机制

###  方法调用流程

在Objective-C中，消息直到运行时才绑定到方法实现上。编译器会将消息表达式[receiver message]转化为一个消息函数的调用，即objc_msgSend。这个函数将消息接收者和方法名作为其基础参数，如以下所示
```
objc_msgSend(receiver, selector)
```

<Excerpt in index | 首页摘要>
 <!-- more -->
<The rest of contents | 余下全文>

如果消息中还有其它参数，则该方法的形式如下所示：
```
objc_msgSend(receiver, selector, arg1, arg2,...)
```

这个函数完成了动态绑定的所有事情：

a、首先它找到selector对应的方法实现。因为同一个方法可
能在不同的类中有不同的实现，所以我们需要依赖于接收者的类
来找到的确切的实现。
b、调用方法实现，并将接收者对象及方法的所有参数传给它。
c、最后，它将实现返回的值作为它自己的返回值。

消息的关键在于我们前面章节讨论过的结构体objc_class，这个结构体有两个字段是我们在分发消息的关注的：
-> 指向父类的指针
-> 个类的方法分发表，即methodLists。
当我们创建一个新对象时，先为其分配内存，并初始化其成员变量。其中isa指针也会被初始化，让对象可以访问类及类的继承体系

当消息发送给一个对象时首先从运行时系统缓存使用过的方法中寻找。
如果找到，执行该方法,如未找到继续执行下面的步骤

objc_msgSend通过对象的isa指针获取到类的结构体，然后在方法分发表里面查找方法的selector。
如果没有找到selector，objc_msgSend结构体中的指向父类的指针找到其父类，并在父类的分发表里面查找方法的selector。
依此，会一直沿着类的继承体系到达NSObject类。一旦定位到selector，函数会就获取到了实现的入口点，并传入相应的参数来执行方法的具体实现,并将该方法添加进入缓存中如果最后没有定位到selector，则会走消息转发流程

###  消息转发流程

当一个对象能接收一个消息时，就会走正常的方法调用流程。但如果一个对象无法接收指定消息时，又会发生什么事呢？默认情况下，如果是以[object message]的方式调用方法，如果object无法响应message消息时，编译器会报错。但如果是以perform…的形式来调用，则需要等到运行时才能确定object是否能接收message消息。如果不能，则程序崩溃。

通常，当我们不能确定一个对象是否能接收某个消息时，会先调用respondsToSelector:来判断一下。如下代码所示：
```
if([self respondsToSelector:@selector(method)]){
      [self performSelector:@selector(method)];
}
```

不过，我们这边想讨论下不使用respondsToSelector:判断的情况

当一个对象无法接收某一消息时，就会启动所谓“消息转发(message forwarding)”机制，通过这一机制，我们可以告诉对象如何处理未知的消息。默认情况下，对象接收到未知的消息，会导致程序崩溃.
错误由 NSObject的“doesNotRecognizeSelector”方法抛出,不过，我们可以采取一些措施，让我们的程序执行特定的逻辑，而避免程序的崩溃。

消息转发机制基本上分为三个步骤：

1>、动态方法解析
2>、备用接收者
3>、完整转发


* 动态方法解析

对象在接收到未知的消息时，首先会调用所属类的类方法
+resolveInstanceMethod:(实例方法)或者
+resolveClassMethod:(类方法)。

在这个方法中，我们有机会为该未知消息新增一个“处理方法”，通过运行时class_addMethod函数动态添加到类里面就可以了。
这种方案更多的是为了实现@dynamic属性。

备用接收者

如果在上一步无法处理消息，则Runtime会继续调以下方法：

```
- (id)forwardingTargetForSelector:(SEL)aSelector
```

如果一个对象实现了这个方法，并返回一个非nil的结果，则这个对象会作为消息的新接收者，且消息会被分发到这个对象。当然这个对象不能是self自身，否则就是出现无限循环。当然，如果我们没有指定相应的对象来处理aSelector，则应该调用父类的实现来返回结果。

这一步合适于我们只想将消息转发到另一个能处理该消息的对象上。但这一步无法对消息进行处理，如操作消息的参数和返回值

* 完整消息转发

如果在上一步还不能处理未知消息，则唯一能做的就是启用完整的消息转发机制了。
我们首先要通过,指定方法签名，若返回nil，则表示不处理。
如下代码：

```
- (NSMethodSignature *)methodSignatureForSelector:(SEL)aSelector
{
   if ([NSStringFromSelector(aSelector) isEqualToString:@"testInstanceMethod"]){
     return [NSMethodSignature signatureWithObjcTypes:"v@:"];
  }  
return [super methodSignatureForSelector: aSelector];
}
```

若返回方法签名，则会进入下一步调用以下方法，对象会创建一个表示消息的NSInvocation对象，把与尚未处理的消息有关的全部细节都封装在anInvocation中，包括selector，目标(target)和参数。
我们可以在forwardInvocation方法中选择将消息转发给其它对象。我们可以通过anInvocation对象做很多处理，比如修改实现方法，修改响应对象等.
如下所示：
```
- (void)forwardInvovation:(NSInvocation)anInvocation
{
    [anInvocation invokeWithTarget:_helper];
    [anInvocation setSelector:@selector(run)];
    [anInvocation invokeWithTarget:self];
}
```

### Method Swizzling

Swizzling原理

在Objective-C中调用一个方法，其实是向一个对象发送消息，而查找消息的唯一依据是selector的名字。所以，我们可以利用Objective-C的runtime机制，实现在运行时交换selector对应的方法实现以达到我们的目的。

每个类都有一个方法列表，存放着selector的名字和方法实现的映射关系。IMP有点类似函数指针，指向具体的Method实现

每一个SEL与一个IMP一一对应，正常情况下通过SEL可以查找到对应消息的IMP实现。

但是，现在我们要做的就是把链接线解开，然后连到我们自定义的函数的IMP上。当然，交换了两个SEL的IMP，还是可以再次交换回来了。

我们通过swizzling特性，将selectorC的方法实现IMPc与selectorN的方法实现IMPn交换了，当我们调用selectorC，也就是给对象发送selectorC消息时，所查找到的对应的方法实现就是IMPn而不是IMPc了

参考[这里](http://www.jianshu.com/p/adf0d566c887)
