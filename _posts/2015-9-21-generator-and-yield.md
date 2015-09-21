---

layout: post
title: 生成器、yield和协程
category : Python
tagline: "Supporting tagline"
tags : [python]

---

Python使用到一定阶段，肯定绕不开yield。

最近终于有时间整理一下自己对yield、生成器、以及协程的一些认识，算是做一下备忘。

注：以下所有代码均在Python2.7.10下执行

####Generator&yield
</br>

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

该生成器函数在`next()`被调用（Python3是`__next__()`)时执行函数：

{% highlight python %}
>>> c.next()
counting down from 10
10
>>> c.next()
9
{% endhighlight %}

这点上很像Python中的另一个对象的行为：迭代器（iterator）。使用iterator也是调用其`next()`方法。

在调用`next()`时，生成器将不断执行语句（这里是while循环），直到再次碰到yield语句为止。yield语句在
函数执行停止的地方返回一个结果，在下一次执行`next()`时接着从yield的后方的语句开始执行，直到再次碰到
yield或引发StopIteration异常。(所以说在这里很像iterator)

生成器在使用结束后会自动调用`close()`表示关闭生成器。也可以手动调用：

{% highlight python %}
>>> c.close()
>>> c.next()
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  StopIteration()
{% endhighlight %}

####Coroutine&yield
</br>

在函数内，yield还可以用作表达式出现在赋值运算符（=）的右边：

{% highlight python %}
>>> def receiver():
...    print 'ready to receive'
...    while 1:
...        n = (yield)
...        print 'got {0}'.format(n)
...
>>>
{% endhighlight %}

以这种方式使用的yield语句成为协程（croutine）。它的执行是为了响应发送给它的值，它的行为也十分类似生成器：

{% highlight python %}
>>> r = receiver()
>>> r.next()
ready to receive
>>> r.send(1)
got 1
>>> r.send(2)
got 2
>>> r.send('hello world')
got hello world
>>>
{% endhighlight %}

想生成器send数据前，必须先执行一次`next()`使语句执行到yield部分。随后yield会等待`send()`给它发送值，拿到值后
紧接着执行下面的语句，直到再次遇到yield语句。若没有调用`send()`，yield会处于挂起状态。

为了防止忘掉首次调用`next()`方法，可以写一个装饰器帮我们自动完成该步骤：

{% highlight python %}
def croutine(func):
    def start(*args, **kwargs):
        g = func(*args, **kwargs)
        g.next()
        return g
    return start
{% endhighlight %}

装饰器可以如下使用：

{% highlight python %}
>>> @croutine
>>> def receiver():
...    print 'ready to receive'
...    while 1:
...        n = (yield)
...        print 'got {0}'.format(n)
...
>>>
{% endhighlight %}

如果yield表达式中提供了值，协程可以通过yield同时接受和发出返回值：

{% highlight python %}
def spliter(delimiter=None):
    print 'Ready to split'
    result = None
    while 1:
        line = (yield result)
        result = line.split(delimiter)
{% endhighlight %}

调用方式跟生成器很像：

{% highlight python %}
>>> s = spliter(',')
>>> s.next()
Ready to split
>>> s.send('A,B,C')
['A', 'B', 'C']
>>> s.send('hello,world')
['hello', 'world']
>>>
{% endhighlight %}

在上面的代码中，`next()`先让协程执行到(yield result)，这时将result的初始值None返回，yield挂起，等待send()调用。
`send()`方法调用后，yield将接收到值放到line中，接着向下执行，line被split，结果放到result中。然后接着往下走，
再次遇到yield，将上次的result的“生成”出来，然后挂起等待`send()`，直到被`close()`关闭。

