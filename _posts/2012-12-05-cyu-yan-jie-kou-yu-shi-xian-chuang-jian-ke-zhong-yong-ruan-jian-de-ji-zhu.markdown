---
layout: post
title: "C语言接口与实现 创建可重用软件的技术"
date: 2012-12-05 16:37
comments: true
categories: [book,c]
---
### 抽象数据类型
隐藏相关的细节
{% highlight objective-c %}
// typedef struct T *T;
// example
#ifndef STACK_INCLUDED
#define STACK_INCLUDED
#define T Stack_T
typedef struct T *T;
extern T     Stack_new  (void);
extern int   Stack_empty(T stk);
extern void  Stack_push (T stk, void *x);
extern void *Stack_pop  (T stk);
extern void  Stack_free (T *stk);
#undef T
#endif
{% endhighlight %}

typedef定义了T类型，这是一个指针，指向一个同名结构。该定义是合法的，因为结构，联合，枚举的名称占据了一个命名空间。

该结构没有给出任何信息，因而是个不透明指针类型，客户可以自由操纵这种指针，但是不可以查看指针指向结构的内部信息。这种做法隐藏了表示细节，而且有助于捕获错误。只有Stackck类型可以传给上述函数，传递其他类型都会出现编译错误。

### continue
