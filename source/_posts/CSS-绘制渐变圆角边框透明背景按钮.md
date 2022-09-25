---
title: CSS 绘制渐变圆角边框透明背景按钮
toc: true
categories:
  - Web 技术
tags:
  - CSS
abbrlink: a86434d
cover: /cover/20220116170143314.bmp
date: 2022-01-16 15:50:43
---

最近有个比较酷炫的项目，其中有个需求是这样的：**圆角**矩形按钮，边框是**渐变色**，背景需**透明**。网上找了很多资料，在我快要放弃时还是找到了解决方案，不得不说 CSS 相当强大，只要你想不到，没有它做不到。尝试过的方法总结下来就以下三种，下面就回顾一下踩坑的过程。

<!--more-->

## border-image 实现渐变

这是最开始尝试的方案，直接用 `border-image` 来实现边框的渐变效果，但是三个需求中只能实现**边框渐变**和**背景透明**，无法实现**圆角**。可以看下面的代码示例：

{% codepen KKXJEdO css,result 330 100% cherrow dark %}

## 利用绝对定位进行背景遮盖

由于 `border-image` 与 `border-radius` 无法同时生效，只能继续找方案。网上最多的方式就是这一种，原理其实很简单，首先利用绝对定位在按钮的下面放置一个 `div`（伪元素也可），然后将这个 `div` 的 `background-image` 设置成渐变色，最后再将按钮的背景色设置为一个纯色，且设置 `background-clip: padding-box`，这是为了避免上层的背景延伸至下层外边框导致渐变色被覆盖（具体可查看 [MDN](https://developer.mozilla.org/zh-CN/docs/Web/CSS/background-clip)），这样底层与上层的间隔看起来就像是一个渐变色的边框。

但是很遗憾，这种方式也只能满足两个需求：**边框渐变**与**圆角**，无法实现**背景透明**。不过这种方式适用于页面背景是固定色的场景，因为页面背景如果是固定色，我们只需要将上层的背景色也设置为这个背景色就可以了，看起来就像是透明的一样。代码示例：

{% codepen NWaoJdV css,result 330 100% cherrow dark %}

## 利用 mask 实现背景透明

不得不说 stackoverflow 上的教程比国内大部分知识论坛网站都要好，下面这个方案就是在 stackoverflow 上找到的，原文提供了更多的解决方案，包括利用 `svg` 自己绘制背景，[点击查看原文](https://stackoverflow.com/questions/51496204/border-gradient-with-border-radius#)。原理其实和上一种方式差不多，不同之处就在于使用了 `mask` 相关的属性实现了背景的透明，至此终于满足了三个需求：边框渐变、圆角、背景透明。

 {% codepen KKXJJjL css,result 330 100% cherrow dark %}

