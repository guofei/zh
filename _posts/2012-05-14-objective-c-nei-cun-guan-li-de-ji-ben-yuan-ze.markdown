---
layout: post
title: "objective-c 内存管理的基本原则"
date: 2012-05-14 00:46
comments: true
categories: [objective-c]
---
## 基本的原则
* 「alloc」、「new」、「copy」、「mutableCopy」开头的method要release
* retain之后要release
* 以+开始的class method一般不需要release
* assign的property不需要release
* CF开始的函数，带有Create的需要通过CFRelease()来释放
* 一般以init开头的函数需要release，而以生成的数据的名字开头的则不需要，比如string... , array...等

## xcode的内存检测工具
长按run按钮，然后选择profile，点击profile后选择leaks
