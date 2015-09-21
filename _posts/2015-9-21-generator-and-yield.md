---

layout: post
title: 生成器、yield和协程
category : Python
tagline: "Supporting tagline"
tags : [python]

---

Python使用到一定阶段，肯定绕不开yield。

最近终于有时间整理一下自己对yield、生成器、以及协程的一些认识，算是做一下备忘。

函数使用yield关键字可以定义generator对象。generator是一个函数，它可以生成一个值的list，所以一定程度上可以做迭代器使用。

{% highlight python %}
def countdown(n):
    print 'counting down from {0}'.format(n)
    while n > 0:
        yield n
        n -= 1
    return
{% endhighlight %}

调用该函数时，函数不会马上像想象中那样执行。

{% highlight python %}
>>> c = countdown(10)
>>>
{% endhighlight %}

相反，它会返回一个生成器对象:

{% highlight python %}
>>> c
<generator object countdown at 0x10291f9b0>
{% endhighlight %}

