---
title: Hexo主题设置
date: 2016-05-06 21:08:23
tags:
categories:
---


Hexo 支持个性化定制主题，可以根据自己的喜好进行修改，想获取更多主题点击[这里](https://github.com/hexojs/hexo/wiki/Themes)哦

<Excerpt in index | 首页摘要>
 <!-- more -->
<The rest of contents | 余下全文>

### 一、安装主题
目前使用的主题是：[yilia](https://github.com/litten/hexo-theme-yilia)

### 克隆主题

```
$ git clone https://github.com/litten/hexo-theme-yilia.git themes/yilia
```

### 执行：

```
$ vim _config.yml
```

### 将 theme 对应的值进行修改
```
theme: yilia
```

### 接着就自动部署一下：

```
$ npm install hexo-deployer-git --save
```

### 最后发布：
```
$ hexo clean && hexo g && hexo d
```

稍等片刻看一下自己的博客主页，你想要的效果就出现了。也可以点击[更多](https://github.com/hexojs/hexo/wiki/Themes)，挑选自己喜欢的主题进行修改，只要你快乐就好

### 二、主题配置

#### 进入到根目录下的 themes\yilia 文件夹，执行

```
$ vim _config.yml
```

头像可以设置  `avatar`  信息,配置文件中都有注释,但是记得冒号后需要加个 `空格`


#### 配置完成后执行
```
$ hexo clean && hexo g && hexo d
```

开心玩耍吧~
