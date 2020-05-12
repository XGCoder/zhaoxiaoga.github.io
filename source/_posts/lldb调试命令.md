---
title: lldb调试命令
date: 2017-04-10 21:56:03
tags:
---


这里简单记录几个好用的lldb调试命名
用户查看app结构相当方便

**pvc命令**
查看当前页面是什么控制器

**pviews**
查看当前控制器有哪些view层级

**border**
给当前的view或者图片图层/背景添加一个边框 可以在模拟器中看到效果不用再编译运行

**unborder**  
去除border 的图层添加

**caflush**  
debug控制器中刷新当前模拟器的图层 不用重新编译

**presponder**  
打印响应链  查看当前控件的响应顺序

**taglog**
点击屏幕程序会暂停,会打印到你所触摸的view

**pclass**
打印当前类的继承关系

**bmessage**  
当控制器没有实现拿个方法时用这个命令可以在运行时给程序添加断点

**hide/show**   
遮盖或者隐藏哪个view

**pinternals**   
查看当前控制器的所有属性,包括私有的
