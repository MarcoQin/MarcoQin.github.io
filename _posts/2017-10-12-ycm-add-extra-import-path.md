---

layout: post
title: YCM 添加额外 Python import 补全目录
category : rest
tagline: "Supporting tagline"
tags : [YCM, Python, Vim, Jedi]

---

因为项目原因，同时写有5个Python工程，同时这几个工程有大量的重复代码，所以最好的办法就是把重复代码拿出来做单独的submodule。

大概项目结构如下：

```bash
- src/   <-- submodule
  - model/
    - test.py
- views/  <-- 正在写的东西, 到时候做个链接到src/目录下
  - views.py
```

蛋疼的是, 若是在 views.py 里 "from src.model.test import xxxx" 的时候, vim 插件 YCM 就会直接报错，类似"Can't go to definition"。

折腾了一阵，看到 YCM 对 Python 做补全是用的 Jedi 这个插件, 我的 vim 也装了 jedi-vim。于是进入 jedi-vim , 修改 autoload/ 目录下的.vim 文件，
在最上方增加代码如下:

```python
python << EOF
import sys
import os
sys.path.insert(0, os.getcwd())
EOF
```

本以为这样就可以了，然而还是不行。

但是总体思路是对的，就是在jedi加载前，修改他们所在vim buffer 使用的 Python path。

钻到 ycm 目录下找，发现 ycm 用的 jedi 是自己 third-party 目录下的jediHTTP，并非共用一个。 jediHTTP 自己也有个 jedi 的库，修改这个库的内容，加上修改
 path 的内容，完美解决。
