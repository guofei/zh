---
layout: post
title: "Facebook Graph API"
date: 2012-05-04 21:34
comments: true
categories: [facebook, graph api]
---
## Object
facebook中的所有人物，照片，活动，page等都是object，每个object都有固有的ID，通过graph api访问object的方法：

{% highlight bash %}
https://graph.facebook.com/ID
{% endhighlight %}

## Fields
每个Object都有多个Fields
比如：id,name,first name,email,birthday,education等等

## connection(连接目标)
所有的object都通过connection连接在一起，通过
{% highlight bash %}
https://graph.facebook.com/ID/CONNECTION_TYPE
{% endhighlight %}

可以获取object间的connection

## Authorization
要取得用户许可的object，必须要在请求中包含access_token，包含access_token之后，就可以代替用户获取或者修改object
{% highlight bash %}
https://graph.facebook.com/220439?access_token=...
{% endhighlight %}

## 情报的取得
默认情况下会去的用户的所有object，如果只想获得自己需要的情报则需要使用fields这个parameter

取得已经安装app的所有朋友的object id(结果会取得所有的friends，install：true 的话说明此朋友已经安装app)
{% highlight bash %}
https://graph.facebook.com/me/friends?fields=installed
{% endhighlight %}

如果想取得复数object，则需要使用ids这个parameter
{% highlight bash %}
https://graph.facebook.com?ids=arjun,vernal
{% endhighlight %}
ids也可以指定url
{% highlight bash %}y
https://graph.facebook.com/?ids=http://www.imdb.com/title/tt0117500/
{% endhighlight %}

### 照片，搜素
todo



