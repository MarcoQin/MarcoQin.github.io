---

layout: post
title: Advanced Bash-Scripting Guide 读书笔记(0)
category : bash
tags : [bash, linux]

---

### 特殊字符

* **#**

    * 注释。行首以#（#!除外）开头是注释。

    * \#也可以出现在特定的参数替换结构中。

* **;**

    命令分隔符，在同一行写两个或以上的命令。

* **;;**

    终止case选项。

```bash
1 case "$variable" in
2 abc) echo "\$variable = abc" ;;
3 xyz) echo "\$variable = xyz" ;;
4 esac
```

* **.**

    * 等价于source命令，用来加载一个数据文件。

        `. data-file # 加载一个数据文件，类似Python的import`

    * 作为目录名，单独的点代表当前工作目录。两个点代表上一级。

* **`**

    命令替换。\`command\`结构可以将命令的输出赋值到一个变量中去。

* **:**

    空命令。等价于"NOP" ( no op , 一个什么也不干的命令)。也可以被认为与shell的内建命令true作用相同。

* **!**

    取反操作。!=（不等于）

* **\***

    通配符\|乘法操作符号\|\*\*求幂运算

* **?**

    * 测试操作符。

    * 在双括号表达中可以做双元运算：(( t = a<45?7:11 )) \# C语言风格的三元操作

    * 通配符。

* **$**

    * 变量替换（引用变量内容）。

    * 正则表达式行结束符。

* **${}**

    同上，参数替换。${parameter}与$parameter 相同。在某些上下文中, ${parameter} 很少会产生混淆。

* **$\*, $@**

    位置参数。

* **$?**

    退出状态码变量. $? 变量 保存了一个命令, 一个函数, 或者是脚本本身的退出状态码。

* **$$**

    进程ID变量。这个$$变量保存了它所在脚本的进程 ID。
