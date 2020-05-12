---
title: 逆向之手机app砸壳记录
date: 2017-04-02 21:38:44
tags:
---

之前做手机逆向把手机给越狱了,过了一些时间了,整理下砸壳手机app的过程.

### 前提条件

* 越狱手机
* dumpdecrypted
* 安装越狱插件: openssh
* 插件 adv-cmds
* 插件 cycript


<Excerpt in index | 首页摘要>
 <!-- more -->
<The rest of contents | 余下全文>

注: 这里只是根据本次砸壳经历记录过程,详情请参考[iOS 逆向手把手教程](http://www.swiftyper.com/2016/05/02/iOS-reverse-step-by-step-part-1-class-dump/)

为了砸壳，我们需要使用到 `dumpdecrypted`，这个工具已经开源并且托管在了 GitHub 上面，我们需要进行手动编译。步骤如下：
1 . 从 GitHub 上 clone 源码：
```
cd ~/iOSReverse
git clone git://github.com/stefanesser/dumpdecrypted/
```

2 . 编译 `dumpdecrypted.dylib`：
```
cd dumpdecrypted/
make
```

执行完 make 命令之后，在当前目录下就会生成一个 dumpdecrypted.dylib，这个就是我们等下要使用到的砸壳工具.

### ssh链接
使用 ssh 连上你的 iOS 系统
```
$ ssh root@192.168.2.142
```

### 查找微信的可执行文件
手机连上之后，使用 ps 配合 grep 命令来找到微信的可执行文件: ps 插件 —> adv-cmds
```
ps -e | grep WeChat
得到路径 :
/var/mobile/Containers/Bundle/Application/5B0950A5-9F44-4EF5-AAA6-FF994DD6EC1D/WeChat.app/WeChat
```

### 查找Documents 目录路径
```
cycript -p WeChat
得到路径 :
/var/mobile/Containers/Data/Application/ADC7EF50-3AD6-43FE-89D5-61308163C34B
```

可以看到，我们打印出了应用的 Home 目录，而 Documents 目录就是在 Home 的下一层，所以在我这里 Documents 的全路径就为 `/var/mobile/Containers/Data/Application/ADC7EF50-3AD6-43FE-89D5-61308163C34B/Documents`

### 开始砸壳
将 `dumpdecrypted.dylib` 拷到 `Documents` 目录下
```
$ scp ~/iOSReverse/dumpdecrypted/dumpdecrypted.dylib root@192.168.2.142:/var/mobile/Containers/Data/Application/ADC7EF50-3AD6-43FE-89D5-61308163C34B/Documents
```
接下来进行真正的砸壳步骤:
```
$ cd  /var/mobile/Containers/Data/Application/ADC7EF50-3AD6-43FE-89D5-61308163C34B/Documents
# 后面的路径即为一开始使用 ps 命令找到的目标应用可执行文件的路径
$ DYLD_INSERT_LIBRARIES=dumpdecrypted.dylib /var/mobile/Containers/Bundle/Application/5B0950A5-9F44-4EF5-AAA6-FF994DD6EC1D/WeChat.app/WeChat
```

完成后 `ls` 会在当前目录生成一个 WeChat.decrypted 文件，这就是砸壳后的文件

### 读取砸壳后文件
从手机中读取文件到电脑 (必须在mac命令行 不是ssh的手机命令行) :
```
scp root@192.168.2.142:/var/mobile/Containers/Data/Application/ADC7EF50-3AD6-43FE-89D5-61308163C34B/Documents/WeChat.decrypted ~/iOSReverse
```

### 获取头文件

class-dump 针对架构( **–arch armv7** )导出头文件:
```
class-dump --arch armv7 -H WeChat.decrypted -o outHeader
```
