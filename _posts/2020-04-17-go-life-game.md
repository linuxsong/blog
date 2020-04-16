---
title: Golang实现的基于命令行界面的康威生命游戏
key: Golang实现的基于命令行界面的康威生命游戏
tags: go
author: LinuxSong
---

之前用js实现过一个康威生命游戏，关于康威生命游戏的介绍，可以参考之前的文章：[康威生命游戏的js+html5实现](https://www.topbyte.cn/2015/11/life-game-js-html5/){:target="_blank"}。

最近正在学习Go语言，做为练习，用Go语言实现了一个基于命令行界面的版本。其中的tui库采用了[tcell](https://github.com/gdamore/tcell){:target="_blank"}

<!--more-->

运行效果如下：

![go-lifegame](https://www.topbyte.cn/assets/images/posts/go-lifegame.gif)

代码放到了Github中:[go-lifegame](https://github.com/linuxsong/go-lifegame){:target="_blank"}