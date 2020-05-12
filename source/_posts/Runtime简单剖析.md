---
title: 10-Runtime简单剖析
date: 2016-07-19 19:32:43
tags:
---


### Runtime 概念
Objective C语言把能在运行期做的事情就推迟到运行期再决定。这就意味着，Objective C不仅需要一个编译器，而且需要一个运行期环境。这个运行期环境就是Runtime.


<Excerpt in index | 首页摘要>
 <!-- more -->
<The rest of contents | 余下全文>

### 基础结构:

* objc_selector

透明的数据结构，可以理解为C String

* SEL
```
typedef struct objc_selector *SEL;
```

SEL是指向一个C String的指针

* id
```
typedef struct objc_object *id;

```

id － 指向一个类的实例对象

* objc_object

objc_object是表示一个类的实例的结构体.
定义如下:

```
struct objc_object {
    Class isa  OBJC_ISA_AVAILABILITY;
};
typedef struct objc_object *id;
```

可以看到，这个结构体只有一个字体，即指向其类的isa指针。这
样，当我们向一个Objective-C对象发送消息时，运行时库会根据
实例对象的isa指针找到这个实例对象所属的类。Runtime库会在类
的方法列表及父类的方法列表中去寻找与消息对应的selector指向
的方法，找到后即运行这个方法。


* Class

Class － 指向Objective C类对象（objc_class）的一个指针

```
typedef struct objc_class *Class;
```


* objc_class

```
struct object_class{
    Class isa OBJC_ISA_AVAILABILITY;
#if !__OBJC2__
     Class super_class                        OBJC2_UNAVAILABLE;  // 父类
     const char *name                         OBJC2_UNAVAILABLE;  // 类名
     long version                             OBJC2_UNAVAILABLE;  // 类的版本信息，默认为0
     long info                                OBJC2_UNAVAILABLE;  // 类信息，供运行期使用的一些位标识
     long instance_size                       OBJC2_UNAVAILABLE;  // 该类的实例变量大小
     struct objc_ivar_list *ivars             OBJC2_UNAVAILABLE;  // 该类的成员变量链表
     struct objc_method_list *methodLists     OBJC2_UNAVAILABLE;  // 方法定义的链表
     struct objc_cache *cache                 OBJC2_UNAVAILABLE;  // 方法缓存
     struct objc_protocol_list *protocols     OBJC2_UNAVAILABLE;  // 协议链表
#endif
}OBJC2_UNAVAILABLE;
```
可以看到，这就是类对象结构体的定义


* method

method - 指向Objective C中的方法的指针

* `_cmd`

SEL 类型的一个变量，Objective C的函数的前两个隐藏参数为self 和 `_cmd`

* Ivar

ivar - objective C中的实例变量

```
typedef struct objc_ivar *Ivar;

```

* 元类(Meta Class):
在上面我们提到，所有的类自身也是一个对象，我们可以向这个对象发送消息(即调用类方法)。
既然是对象，那么它也是一个objc_object指针,它包含一个指向其类的一个isa指针。

为了调用类方法，这个类的isa指针必须指向一个包含这些类方法的一个objc_class结构体。这就引出了meta-class的概念，meta-class中存储着一个类的所有类方法。
所以，调用类方法的这个类对象的isa指针指向的就是meta-class
当我们向一个对象发送消息时，runtime会在这个对象所属的这个类的方法列表中查找方法；而向一个类发送消息时，会在这个类的meta-class的方法列表中查找。

meta-class也是一个类，也可以向它发送一个消息.为了不让这种结构无限延伸下去，Objective-C的设计者让所有的meta-class的isa指向基类的meta-class，以此作为它们的所属类。也就是说 任何NSObject继承体系下的meta-class都使用NSObject的meta-class作为自己的所属类，而基类的meta-class的isa指针是指向它自己。

![](img/runtime的isa指针指向图.png)

* Category

Category是表示一个指向分类的结构体的指针
```
typedef struct objc_category *Category
struct objc_category{
     char *category_name                         OBJC2_UNAVAILABLE; // 分类名
     char *class_name                            OBJC2_UNAVAILABLE;  // 分类所属的类名
     struct objc_method_list *instance_methods   OBJC2_UNAVAILABLE;  // 实例方法列表
     struct objc_method_list *class_methods      OBJC2_UNAVAILABLE; // 类方法列表
     struct objc_protocol_list *protocols        OBJC2_UNAVAILABLE; // 分类所实现的协议列表
}
```

这个结构体主要包含了分类定义的实例方法与类方法，其中instance_methods列表是objc_class中方法列表的一个子集，而class_methods列表是元类方法列表的一个子集。
可发现，类别中没有ivar成员变量指针，也就意味着：类别中不能够添加实例变量和属性

### runtime关联对象

* 设置关联值

object：与谁关联，通常是传self
key：唯一键，在获取值时通过该键获取，通常是使用 `static const void *` 来声明
value：关联所设置的值
policy：内存管理策略，比如使用copy
```
void objc_setAssociatedObject(id object, const void *key, id value, objc _AssociationPolicy policy)
```

* 获取关联值
object：与谁关联，通常是传self，在设置关联时所指定的与哪个对象关联的那个对象
key：唯一键，在设置关联时所指定的键
```
id objc_getAssociatedObject(id object, const void *key)
```

* 取消关联
```
void objc_removeAssociatedObjects(id object)
```

* 关联策略
```
typedef OBJC_ENUM(uintptr_t, objc_AssociationPolicy){
OBJC_ASSOCIATION_ASSIGN = 0,             // 表示弱引用关联，通常是基本数据类型
OBJC_ASSOCIATION_RETAIN_NONATOMIC = 1,   // 表示强引用关联对象，是线程安全的
OBJC_ASSOCIATION_COPY_NONATOMIC = 3,     // 表示关联对象copy，是线程安全的
OBJC_ASSOCIATION_RETAIN = 01401,         // 表示强引用关联对象，不是线程安全的
OBJC_ASSOCIATION_COPY = 01403            // 表示关联对象copy，不是线程安全的
};
```

### 方法与消息

* SEL
SEL又叫选择器，是表示一个方法的selector的指针

```
typedef struct objc_selector *SEL；
```
方法的selector用于表示运行时方法的名字。Objective-C在编译时，会依据每一个方法的名字、参数序列，生成一个唯一的整型标识(Int类型的地址)，这个标识就是SEL。
两个类之间，只要方法名相同，那么方法的SEL就是一样的，每一个方法都对应着一个SEL。所以在Objective-C同一个类(及类的继承体系)中，不能存在2个同名的方法，即使参数类型不同也不行

当然，不同的类可以拥有相同的selector，这个没有问题。不同类的实例对象执行相同的selector时，会在各自的方法列表中去根据selector去寻找自己对应的IMP。

工程中的所有的SEL组成一个Set集合，如果我们想到这个方法集合中查找某个方法时，只需要去找到这个方法对应的SEL就行了，SEL实际上就是根据方法名hash化了的一个字符串，而对于字符串的比较仅仅需要比较他们的地址就可以了，可以说速度上无语伦比！
本质上，SEL只是一个指向方法的指针（准确的说，只是一个根据方法名hash化了的KEY值，能唯一代表一个方法），它的存在只是为了加快方法的查询速度。
通过下面三种方法可以获取SEL:
a、sel_registerName函数
b、Objective-C编译器提供的@selector()
c、NSSelectorFromString()方法


* IMP
IMP实际上是一个函数指针，指向方法实现的地址。
```
id (*IMP)(id, SEL,...)
```
第一个参数：是指向self的指针(如果是实例方法，则是类实例的内存地址；如果是类方法，则是指向元类的指针)
第二个参数：是方法选择器(selector)
接下来的参数：方法的参数列表。

前面介绍过的SEL就是为了查找方法的最终实现IMP的。由于每个方法对应唯一的SEL，因此我们可以通过SEL方便快速准确地获得它所对应的IMP，查找过程将在下面讨论。取得IMP后，我们就获得了执行这个方法代码的入口点，此时，我们就可以像调用普通的C语言函数一样来使用这个函数指针了

* Method

Method用于表示类定义中的方法，则定义如下：
```
typedef struct objc_method *Method
struct objc_method{
    SEL method_name      OBJC2_UNAVAILABLE; // 方法名
    char *method_types   OBJC2_UNAVAILABLE;
    IMP method_imp       OBJC2_UNAVAILABLE; // 方法实现
}
```
我们可以看到该结构体中包含一个SEL和IMP，实际上相当于在SEL和IMP之间作了一个映射。有了SEL，我们便可以找到对应的IMP，从而调用方法的实现代码。
