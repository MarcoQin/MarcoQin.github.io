---

layout: post
title: partial and piping
category : Python
tagline: "Supporting tagline"
tags : [Python]

---

Python中一直有个非常有用的工具函数`functools.partial`，用来对函数进行打包。

官方文档中写有partial的实现：

```python
def partial(func, *args, **keywords):
    def newfunc(*fargs, **fkeywords):
        newkeywords = keywords.copy()
        newkeywords.update(fkeywords)
        return func(*(args + fargs), **newkeywords)
    newfunc.func = func
    newfunc.args = args
    newfunc.keywords = keywords
    return newfunc
```

可以这样使用：

```python
>>> def add (x, y):
...     return x + y
...
>>> add (1, 2)
3
>>> add2 = partial(add, 2)
>>> add2(1)
3
>>> add2(2)
4
```

新版的Python中（比如3.5.1），有两个我们不是很常用的特性：

- 省略号：

```python
>>> print(...)
Ellipsis
```

- 矩阵乘操作符（@）：

```python
class Str(str):
	def __matmul__(self, other):
		print("{}@{}".format(self, other))
>>> Str('here') @ 'there'
here@there
```

对于上面partial的例子，我们想要用…做占位符，像这样：

```python
add2 = add(..., 2)
```

我们可以这样重新实现以下partial函数：

```python
from functools import wraps

class Partial(object):
    def __init__(self, func, args, kwargs):
        self._func = func
        self._args = args
        self._kwargs = kwargs

    def __call__(self, replacement):
        args = [replacement if arg is ... else arg
                for arg in self._args]
        kwargs = {k: replacement if v is ... else v
                  for k, v in self._kwargs.items()}
        return self._func(*args, **kwargs)

    def __repr__(self):
        return '<Partial: {}(*{}, **{})>'.format(
        	self._func.__name__, repr(self._args), repr(self._kwargs))

def ellipsis_partial(func):
    @wraps(func)
    def _wrapper(*args, **kwargs):
        ellipsis_num = (list(args) + list(kwargs.values())).count(...)
        if ellipsis_num > 1:
            raise TypeError('Only one ... allowed as an argument')
        elif ellipsis_num:
            return Partial(func, args, kwargs)
        else:
            return func(*args, **kwargs)
    return _wrapper
```

如果我们在参数中找到 … ，就返回Partial实例。当Partial的实例被调用时，把 … 替换成给定的参数。

```python
@elipsis_partial
def add(x, y):
    return x + y

add2 = add(2, ...)
>>> add2(10)
12
```

Linux系统中的shell一般都有管道操作符`|`，用管道可以在不同的功能之间进行通讯，如常见的：

```shell
cat xxx.txt | grep xxx
```

在F#中也有管道操作：

```
> [1..10] |> List.filter (fun x -> x % 3 = 0)
val it : int list = [3; 6; 9]
```

现在用Python3也可以很容易实现类似的东西。（ `|` 在Python中是常用的操作符，所以用@代替）:

```python
range(1, 10) @ filter(lambda x: x % 3 == 0, ...)
```

所以在Partial类中加入对 @ 操作符的支持：

```python
class Partial(object):
    def __init__(self, func, args, kwargs):
        self._func = func
        self._args = args
        self._kwargs = kwargs

    def __call__(self, replacement):
        args = [replacement if arg is ... else arg
                for arg in self._args]
        kwargs = {k: replacement if v is ... else v
                  for k, v in self._kwargs.items()}
        return self._func(*args, **kwargs)

    def __rmatmul__(self, replacement):
        return self(replacement)

    def __repr__(self):
        return '<Partial: {}(*{}, **{})>'.format(
        	self._func.__name__, repr(self._args), repr(self._kwargs))
```

使用：

```python
filter = ellipsis_partial(filter)
to_list = ellipsis_partial(list)
>>> range(1, 10) @ filter(lambda x: x % 3 == 0, ...) @ to_list(...)
[3, 6, 9]
```

每个想用的方法都写一遍@elipsis_partial也太费劲了，可以用更简单的方式:

```python
def _(func, *args, **kwargs):
	return ellipsis_partial(func)(*args, **kwargs)
```

使用起来就会简单很多：

```python
from functools import reduce
>>> range(1, 10) @ _(map, lambda x: x + 4, ...) \
				 @ _(filter, lambda x: x % 3 == 0, ...) \
				 @ _(reduce, lambda x, y: x * y, ...) \
				 @ _('value: {}'.format, ...)
value: 648
```
