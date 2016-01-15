---
layout: post
title: "Metaprogramming Ruby"
date: 2012-12-09 01:17
comments: true
categories: [ruby,book]
---
## 第一章，object model
### class与module
1. class是object，class name是定数。object的instance value住在object里面，object的method住在class里面。
2. class只是一个拥有allocate, new, superclass这三个函数的module。那么什么时候用class，什么时候用module呢？一般当要被include的时候用module（或者当作namespace使用的时候），要生成或者继承的时候使用class。
{% highlight ruby %}
String.instance_methods == 'abc'.methods # => true
Class.instance_methods(false)            # => [:allocate, :new, :superclass]
# class
'hello'.class # => String
String.class  # => Class
Class.class   #=> Class
# superclass
Myclass.superclass        # => Object
String.superclass         # => Object
Class.superclass          # => Module
Module.superclass         # => Object
Object.superclass         # => BasicObject
BasicObject.superclass    # => nil
{% endhighlight %}

#### Kernel module
Object Class include 了 Kernel 这个module，Kernel module里面有print()之类的method，因此可以直接在任何地方呼叫print()，如果展开Kernel，在里面定义自己的method的话，也可以在任何地方呼叫自己的method。

#### private真正的意义
用private关键词定义method有一个规则，那就是不可以指定receiver（接收对象），比如
{% highlight ruby %}
class C
  def pub
    self.pri #这里是错误的
    pri　　　#这里是正确的
  end
  private
  def pri;end
end
{% endhighlight %}
然而呼叫一般的method也有一个规则，就是除了自己，呼叫任何method都必须明确制定receiver。private method也要遵守这个规则，就是说private method只可以呼叫自己。那么从superclass继承的private method呼叫么？答案当然是可以，因为继承的class里面也可以不用明确的制定receiver的。


### 定数
module或者class可以看成dirictory，定数可以看成file
{% highlight ruby %}
module Mymodule
  MyConstant = 'outer const'

  class Myclass
    MyConstant = 'inner const'
  end
end
# path
module M
  class C
    X = 'const'
  end

  C::X  # => 'const'
end
M::C::X # => 'const'
{% endhighlight %}

为了避免自己的class名字跟其他library里面的class名字冲突，可以用module来定义自己的class。比如
{% highlight ruby %}
module Mymodule
  class C1
  end

  class C2
  end
end
{% endhighlight %}

## 第二章，method
### 重复code的排除

#### 动态呼叫method
{% highlight ruby %}
class MyClass
  def my_method(arg)
    arg*2
  end
end
obj = MyClass.new
obj.send(:my_method, 3)
{% endhighlight %}

#### 动态定义method
define_method
{% highlight ruby %}
class Computer
  def initialize(id, source)
    @id = id
	@data_source = source
  end

  def sef.define_component(method_name)
    define_method(method_name){
	  info = @data_source.send "get_#{method_name}_info", @id
	  price = @data_source.send "get_#{method_name}_price", @id
	  result = "#{method_name.to_s.capitalize}: #{info} ($#{price})"
	  return "* #{result}" if price >= 100
	  result
	}
  end

  define_component :mouse
  define_component :cpu
  define_component :keyboard
end
{% endhighlight %}

#### method_missing()
obj.mymethod的时候会先向右在obj的class里查找mymethod，如果class里面没有就向上查找superclass，然后一直向上查找到BasicObject，如果BasicObject里面也没有的话就会呼叫obj的method＿missing，如果重新定义methodmissing的话就可以呼叫没有定义过的method，这个method就叫做__ghost method__
{% highlight ruby %}
class Lawyer
  def method_missing(method, *args)
    puts "#{method}(#{args.join(", ")})"
	puts "block" if block_given?
  end
end
bob = Lawyer.new
bob.talk_simple("a", "b") do
end
# => talk_simple(a, b)
# => block
{% endhighlight %}
与method＿missing一样，Module#const＿missing是Module下面的method

#### Module#undef_method(), Module#remove_method()
undef＿method会删除class里已经继承这个class里的所有method,remove＿method则只会删除receiver里面的method
{% highlight ruby %}
class Computer
  instance_methods.each do |m|
    undef_method m unless m.to_s =~ /^__|method_missing|respond_to?/
  end
  #...
end
{% endhighlight %}

## 第三章，block
### Kernel#block_given?()
{% highlight ruby %}
def a_method
  return yield if block_given?
  "No block"
end
{% endhighlight %}

### 闭包
单独的block code是无法执行的，要执行block还要有local variable，instance variable，self等环境，这个环境在ruby里面也叫做束缚（翻译的可能不太准确），block由code和束缚组成。
{% highlight ruby %}
def my_method
  x = "goodbye"
  yield("cruel")
end

x = "hello"
my_method{|y| "#{x}, #{y} world"} # => "hello, cruel world"
{% endhighlight %}

这个block取得了x = hello这个束缚。这个特性也被称作闭包。

### scope gata
ruby切换scope，然后打开一个新的scope有3个地方，分别是：__1，定义class。2，定义module。3，呼叫method__, 也就是__class, module, def__这三个关键字。那么如何突破scope，传递束缚呢？其实很简单，定义class，或者定义method的时候不用这3个关键字就可以了，比如定义class的时候可以使用Class.new，定义method的时候可以用difine＿method。可以看下面这个例子
{% highlight ruby %}
my_var = "success"
MyClass = Class.new do
  puts "#{my_var}"

  define_method :my_method do
    puts "#{my_var}"
  end
end
{% endhighlight %}

下面这个例子是共享scope
{% highlight ruby %}
def define_methods
  shared = 0
  Kernel.send :define_method, :counter do
    shared
  end
  Kernel.send :define_method, :inc do |x|
    shared += x
  end
end
define_methods
counter # => 0
inc(4)
counter # => 4
{% endhighlight %}

### instance_eval()
在block里面评价object的context(上下文)
{% highlight ruby %}
class MyClass
  def initialize
    @v = 1
  end
end
obj = MyClass.new
obj.instance_eval do
  self   # => <MyClass:0x3340dc @v=1>
  @v     # => 1
  @v = 2 # => 2
end
{% endhighlight %}

cleanroom
{% highlight ruby %}
env = Object.new
@setups.each do |setup|
  env.instance_eval &setup
end
env.instance_eval &event
{% endhighlight %}

### callable object
#### Proc
block转换成Proc，在block前面加上＆修饰符就可以了。block通过yield来呼叫，而Proc通过call来呼叫。Proc转换成block，也是在Proc前面加上＆就可以了。在1.9里面，Proc通过Proc.new或者Kernel#proc()来创建。

#### lambda与Proc的区别
lambda通过Kernel#lambda()或者 ー＞（将来是否可以用还不知道）来创建，lambda像是一个匿名函数，因此在return的时候，lambda会跳出block，而Proc则会跳出block所在的method。另外一方面，lambda在传递引数的时候比较严格，而Proc则比较宽松，比如
{% highlight ruby %}
p = Proc.new {|a, b| [a, b]}
p.call(1,2,3) # => [1,2]
p.call(1)     # => [1.nil]
{% endhighlight %}

#### Method object
Method通过Object#method()来取得，通过Method#call()来呼叫。Method跟lambda基本一样，不一样的是lambda在scope里面评价，而Method在附属的object里面评价。

## 第四章 class的定义
### class
#### class_eval(),别名module_eval()
在block里面评价Class的context(上下文)

#### class instance value
{% highlight ruby %}
class MyClass
  @var = 1
end
{% endhighlight %}
class value则是@@var，一般不建议使用

class的名字只是单纯的__定数__。
{% highlight ruby %}
c = Class.new(Array) do
  def mymethod 'hello' end
end
MyClass = c
{% endhighlight %}

#### 特异method
{% highlight ruby %}
str = "strrrr"
def str.title?
  self.upcase == self
end
{% endhighlight %}
这里的title？就是str object的特异method

__class method 就是 class的特异method__

#### 特异class
{% highlight ruby %}
class << an_object
  # code
end
{% endhighlight %}
特异method就住在特异class里面。

__大统一理论__：1，object只有一种，那就是通常的object或者module。2，module只有一种，那就是module或者class或者特异class或者proxy class。3，method只有一种，就住在module（大部分是class）里面。4，所有的object都拥有class，这个class可能是一般的class或者特异class。5，所有的class都有super class（BasicObject除外），所有的class都继承自BasicObject。6，object的特异class的superclass是object的class。class的特异class的superclass是class的superclas的特异class。7，method呼叫的时候先向右一步找他的class，再向上找superclass。

{% highlight ruby %}
class MyClass
  class << self
    include MyModule
  end
end
# 等同于
Myclass.extend Mymodule
# 等同于
class MyClass
  extend Mymodule
end
{% endhighlight %}

#### alias
{% highlight ruby %}
class String
  alias :real_length :length
  def length
    real_length > 5 ? 'long' : 'short'
  end
end
{% endhighlight %}

## 第五章，code来写code
### Kernel#eval
{% highlight ruby %}
class MyClass
  def my_method
    @x = 1
	binding
  end
end
eval "@x", MyClass.new.my_method # => 1
{% endhighlight %}

instance_eval也可以接收文字，如
{% highlight ruby %}
array = ['a','b','c']
x = 'd'
array.instance_eval "self[1] = x"
array # => ['a','d','c']
{% endhighlight %}

