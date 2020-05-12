---
title: 总结的OC代码规范
date: 2016-05-10 11:47:55
tags:
---

### 命名:

* 驼峰命名法:

所有的单词首字母大写和加上与类名有关的前缀,例如:`MyRootViewControllerNavigationAnimationDuration`

<Excerpt in index | 首页摘要>
 <!-- more -->
<The rest of contents | 余下全文>

### 注释:

```
/** ---- */  格式
```
或者
```
/**
  *
  */      格式
```
并且标明此段代码的意思以及用处

### 代码块:
不同业务代码和protocol/delegate实现中使用`#pragma mark -` 来分类方法

`#pragma mark - UITableView Delegate`


### 方法空格:

* 在方法 (-/+) 后面 有一个空格
* 在方法各个参数段之间有一个空格

### 变量名称:

变量尽量以描述性的方式来命名。单个字符的变量命名应该尽量避免，除了在`for()`循环。


### 属性特性:
所有属性特性应该显式地列出来,例如 strong / weak / copy / readonly / readwrite

### 点语法:

点语法是一种很方便封装访问方法调用的方式。当你使用点语法时，通过使用getter或setter方法，属性仍然被访问或修改.
点语法应该总是被用来访问和修改属性，因为它使代码更加简洁

### 常量:
常量是容易重复被使用和无需通过查找和代替就能快速修改值。常量应该使用`static`来声明而不是使用`#define`，除非显式地使用宏.

### 枚举类型:

当使用enum时，推荐使用宏`NS_ENUM()`和`NS_OPTIONS`
* NS_OPTIONS
```
typedef NS_OPTIONS(NSUInteger, UISwipeGestureRecognizerDirection) {
    UISwipeGestureRecognizerDirectionNone = 0,  //值为0
    UISwipeGestureRecognizerDirectionRight = 1 << 0,  //值为2的0次方
    UISwipeGestureRecognizerDirectionLeft = 1 << 1,  //值为2的1次方
    UISwipeGestureRecognizerDirectionUp = 1 << 2,  //值为2的2次方
    UISwipeGestureRecognizerDirectionDown = 1 << 3  //值为2的3次方
};
```
* NS_ENUM
```
typedef NS_ENUM(NSInteger, NSWritingDirection) {
    NSWritingDirectionNatural = -1,  //值为-1    
    NSWritingDirectionLeftToRight = 0,  //值为0
    NSWritingDirectionRightToLeft = 1  //值为1       
};
```
NS_ENUM与NS_OPTIONS区别:
* `NS_ENUM`枚举项的值为`NSInteger`，`NS_OPTIONS`枚举项的值为`NSUInteger`；
* `NS_ENUM`定义通用枚举，`NS_OPTIONS`定义位移枚举

位移枚举即是在你需要的地方可以同时存在多个枚举值,而NS_ENUM定义的枚举不能几个枚举项同时存在，只能选择其中一项

### 布尔值:
Objective-C使用`YES`和`NO`,不用`true`和`false`,
因为true和false应该只在CoreFoundation，C或C++代码使用。

### 三元操作符/三目运算符:
当需要提高代码的清晰性和简洁性时，三元操作符`?:`才会使用。单个条件求值常常需要它。多个条件求值时，如果使用`if`语句或重构成实例变量时，代码会更加易读。一般来说，最好使用三元操作符是在根据条件来赋值的情况下。

### Init方法:
Init方法应该遵循Apple生成代码模板的命名规则。返回类型应该使用`instancetype`而不是`id`

### 类构造方法:
当类构造方法被使用时，它应该返回类型是`instancetype`而不是`id`

### CGRect函数:
当访问CGRect里的x, y, width, 或 height时,使用`CGRectMake`, 不要使用 (CGRect){}

### 单例模式:
单例对象应该使用线程安全模式来创建共享实例。

```
+ (instancetype)sharedInstance {
  static id sharedInstance = nil;

  static dispatch_once_t onceToken;
  dispatch_once(&onceToken, ^{
    sharedInstance = [[self alloc] init];
  });

  return sharedInstance;
}
```
