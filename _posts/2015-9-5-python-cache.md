---

layout: post
title: Python中用缓存提高函数性能
category : Python
tagline: "Supporting tagline"
tags : [cache, python]

---

把之前做的一些东西慢慢地整理一下.

这次谈到的是Python中某些消耗特别大的方法的性能调优.

很久之前在写某些递归函数时, 用很粗糙的手法写了缓存策略. 不过胜在还算清晰,
代码也易懂很多. 

之前做公司内部的OAM平台时, 又遇到了一些消耗资源大户, 而且由于执行效率升值会
影响平台正常的使用.(虽然公司的网烂到根本感觉不出来是什么地方慢了..)

以下是最开始写的一个Django的filter, 用来动态渲染oam系统左侧一级菜单栏的开闭状态.

{% highlight py %}
```
@register.filter(name='one_cl')
def one_cl(cur, base):
    try:
        for item in base.children:
            try:
                for i in item.children:
                    try:
                        for j in i.children:
                            if cur == j.uri:
                                c = 'open'
                                break
                    except:
                        if cur == i.uri:
                            c = 'open'
                            break
            except:
                if cur == item.uri:
                    c = 'open'
                    break
    except:
        if cur == base.uri:
            c = 'open'
    return c
```
{% endhightlight%}

由于时间仓促, 并且权限控制部分的model实在过于难用, 以下代码的可读性比较差..
而且一眼就可以看出, 如此多的循环嵌套, 每打开一次一级菜单都执行一遍的话, 开销
确实有点大..

于是做了点小小的改动:

{% highlight py %}
```
one_dict = {} # cashe dict
@register.filter(name='one_cl')
def one_cl(cur, base):
    if '%s:%s'%(cur, base.id) in one_dict:
        return one_dict['%s:%s'%(cur, base.id)]
    c = ''
    try:
        for item in base.children:
            try:
                for i in item.children:
                    try:
                        for j in i.children:
                            if cur == j.uri:
                                c = 'open'
                                break
                    except:
                        if cur == i.uri:
                            c = 'open'
                            break
            except:
                if cur == item.uri:
                    c = 'open'
                    break
    except:
        if cur == base.uri:
            c = 'open'
    one_dict['%s:%s'%(cur, base.id)] = c
    return c
```
{% endhighlight %}

仅仅是增加了一层薄薄的缓存, 便可以在初始的几次执行之后, 再无需执行大量的循环
代码就可以直接获取结果了.

不过这样代码也变得有点丑, 而且这层cashe也仅仅适用于当下的这个方法. 若以后出现了
类似的问题, 很可能需要再写这样一个, 遂觉不爽.

网上随意一艘, 发现Python3的标准库functools中有实现一个缓存装饰器, 叫functools.lru_cache.
使用很简单:

`@functools.lru_cache(maxsize=None, typed=False)`

说明: maxsize为最多缓存的次数, 如果为None, 则无限制, 设置为2n时, 性能最佳;
如果typed=True, 则不同参数类型的调用将分别缓存, 例如f(3)和f(3.0)

使用此装饰器便可以节省大量代码, 并且让阅读代码的人更容易看懂.

Python2.7中的functools并没有实现此装饰器. 不过有个官方代替品: functools32.
可以在[这里](https://pypi.python.org/pypi/functools32/)下载到
