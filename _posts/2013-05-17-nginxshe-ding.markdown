---
layout: post
title: "nginx設定"
date: 2013-05-17 19:46
comments: true
categories: webserver
---
### passanger and nginx
install
{% highlight bash %}
gem install passenger
passenger-install-nginx-module
{% endhighlight %}

start
{% highlight bash %}
path/to/nginx/sbin/nginx
{% endhighlight %}

restart
{% highlight bash %}
path/to/nginx/sbin/nginx -s  reload
{% endhighlight %}

config(rails)
{% highlight bash %}
location / {
    root   /var/www/railsapp/public;   # rails app 的 public文件夹
    passenger_enabled on;              # passenger
    index  index.html index.htm;
}
{% endhighlight %}

在server的{}之间插入下面代码：
{% highlight bash %}
if ($host != 'your_domain.com' ) {
    rewrite ^/(.*)$ http://your_domain.com/$1 permanent;
}
上面这个是所有子域名都跳转到无www的站点

if ($host = 'www.your_domain.com' ) {
    rewrite ^/(.*)$ http://your_domain.com/$1 permanent;
}
这个是针对单个子域名
{% endhighlight %}
