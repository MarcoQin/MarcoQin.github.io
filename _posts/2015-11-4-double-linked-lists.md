---

layout: post
title: double linked list C实现
category : C
tagline: "Supporting tagline"
tags : [C, data structure]

---

抄[learn c the hard way](http://c.learncodethehardway.org/book/)终于到了最后了。

刚弄完双链表，做笔记记录下。

双链表数据结构还算比较简单的，也非常容易理解。

代码的前置"dbg.h"是一个debug调试宏，代码如下：

{% gist MarcoQin/1886d28d85ecd57af411 %}

"list.h"定义了ListNode 和List结构，以及常用函数：

{% gist MarcoQin/932e4fbeb45f1522dd9c %}

"list.c"实现全部函数：

{% gist MarcoQin/8f2f2be1378507b00854 %}

接下来是测试代码。测试需要一个前置宏"minunit.h"，是一个轻量化的单元测试宏：

{% gist MarcoQin/5300d70556892faab380 %}

在代码目录下编译：

{% highlight bash%}
gcc -g -Wall -c -o list.o list.c
gcc -g -Wall list_test.c list.o -o list_test
{% endhighlight %}

输出结果如下：

{% highlight bash%}
$ ./list_test
DEBUG list_test.c:113: ----- RUNNING: ./list_test
----
RUNNING: ./list_test
DEBUG list_test.c:103:
----- test_create
DEBUG list_test.c:104:
----- test_push_pop
DEBUG list_test.c:105:
----- test_unshift
DEBUG list_test.c:106:
----- test_remove
DEBUG list_test.c:107:
----- test_shift
DEBUG list_test.c:108:
----- test_destroy
ALL TESTS PASSED
Tests run: 6
{% endhighlight %}
