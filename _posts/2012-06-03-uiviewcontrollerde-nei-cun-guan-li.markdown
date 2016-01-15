---
layout: post
title: "UIViewController的内存管理"
date: 2012-06-03 02:21
comments: true
categories: [objective-c, memory]
---
正常NSObject都是通过init申请内存，通过dealloc来释放内存，但是UIViewController有时会通过viewDidload来申请内存，需要注意的是：通过viewDidload来申请的内存需要由viewDidUnload和dealoc两方来释放！

这样做的原因是  
viewDidUnload是在内存不足的时候才会执行的，所以在内存充足的情况下只通过viewDidUnload来释放内存的话，内存将不会释放

因此正确的做法是：  
通过 init 申请的内存需要  dealloc 来释放
通过 viewDidload 申请的内存则需要 viewDidUnload 和 dealloc 两方来释放

