---

layout: post
title: mysql has gone away 的初步解决
category : mysql
tags : [mysql]

---

在使用mysql时，总是出现如题所示的这种情况。

而且使用sqlalchemy时，它会把异常转化为它自己的
*StatementError: Can't reconnect until invalid transaction is rolled back.*实在是令人不爽。

一般较为普遍的解决方式是改变mysql的timeout时间。

首先，`SHOW VARIABLES;`可以查看所有的内置option。这里我们需要的是：

`SET GLOBAL connect_timeout=28800;`设置全局连接超时时间，然后用 `SHOW VARIABLES LIKE 'connect_timeout';`查看设置。

接下来也同样：

    SET GLOBAL interactive_timeout=28800;
    SHOW VARIABLES LIKE 'interactive_timeout';
    SET SESSION wait_timeout = 999999;
    SHOW VARIABLES LIKE 'wait_timeout';
    SET SESSION net_read_timeout = 1000;
    SHOW variables LIKE 'NET_read_timeout';

