---

layout: post
title: Python C Extending
category : Python
tags : [Python, C]

---
最近业余时间一直在做C的项目，用C写了个音乐播放器 。

刚开始，只做了歌曲的管理及UI（第一个是用Python + Qt [Qplayer](https://github.com/MarcoQin/Qplayer)，
第二次使用C + curses [Cplayer](https://github.com/MarcoQin/Cplayer)，
第三次用C + GTK+ [Cplayer-gui](https://github.com/MarcoQin/Cplayer-gui)），然后用
pipe或FIFO连接mplayer的进程做歌曲播放和操作。进程间用pipe通信效率实在是低的可以，尤其是Python，多线程GUI
卡的实在是难受。于是萌发了直接写音频解码。

后来一切也蛮顺利，使用ffmpeg做解码，SDL2.0做音频播放。用C实现后简直快得不可思议，已经作为我日常使用的音乐播放器了。

然后，C语言和Python用了这么久，自然而然就想把它们结合起来，即能享受C的高效，又能有Python处理各种常用数据的舒适，于是今天
尝试使用Python C-API做扩展。

看Python doc，一步步走了下来，先做了个抽取音频文件metadata的封装，代码如下：

```c
/* file: sound_utils.c */
#include <Python.h>
#include <libavformat/avformat.h>
#include <libavutil/dict.h>

static PyObject * metadata(PyObject *self, PyObject *args) {
    const char *file_name;
    int ret;
    PyObject *dict;
    PyObject *v;
    if (!PyArg_ParseTuple(args, "s", &file_name)) {
        return NULL;
    }
    AVFormatContext *fmt_ctx = NULL;
    AVDictionaryEntry *tag = NULL;
    av_register_all();
    if ((ret = avformat_open_input(&fmt_ctx, file_name, NULL, NULL)))
        return NULL;
    dict = Py_BuildValue("{}");

    while ((tag = av_dict_get(fmt_ctx->metadata, "", tag, AV_DICT_IGNORE_SUFFIX))) {
        v = Py_BuildValue("s", tag->value);
        PyMapping_SetItemString(dict, tag->key, v);
    }

    avformat_close_input(&fmt_ctx);

    return dict;
}

static PyMethodDef SoundUtilsMethods[] = {
    {"metadata",  metadata, METH_VARARGS, "Get input file's metadata."},
    {NULL, NULL, 0, NULL}        /* Sentinel */
};

PyMODINIT_FUNC initSoundUtils(void) {
    (void) Py_InitModule("SoundUtils", SoundUtilsMethods);
}

int main(int argc, char *argv[]) {
    /* Pass argv[0] to the Python interpreter */
    Py_SetProgramName(argv[0]);

    /* Initialize the Python interpreter.  Required. */
    Py_Initialize();

    /* Add a static module */
    initSoundUtils();
    return 0;
}
```

上面是这个模块儿的主体，依照文档的结构，分别定义函数、模块儿结构、和模块初始化函数。

接下来只需要再写个setup.py用Python build就可以了：

```py
#!/usr/bin/env python
# encoding: utf-8
# file: setup.py

from distutils.core import setup, Extension

module1 = Extension('SoundUtils',
                    libraries=['avutil', 'avformat'],
                    sources=['soud_utils.c'])

setup(name='SoundUtils',
      version='1.0',
      description='This is sound parse package',
      author='Marco Qin',
      author_email='qyyfy2009@gmail.com',
      url='https://marcoqin.github.io',
      long_description=''' This is SoundUtils ''',
      ext_modules=[module1]
)
```

将上述两个文件放在同一目录下，运行`python setup.py build`就能在当前目录的build/目录下编译出SoundUtils.so的动态链接库。

下面用Python测试一下功能：

```bash
╭─marcoqin@marcoqin-begin  ~/marco/C/audio/build/lib.linux-x86_64-2.7
╰─$ la
total 12M
-rw------- 1 marcoqin marcoqin 12M Mar 17 10:37 a.flac
-rwxrwxr-x 1 marcoqin marcoqin 59K Mar 17 11:01 SoundUtils.so*
╭─marcoqin@marcoqin-begin  ~/marco/C/audio/build/lib.linux-x86_64-2.7
╰─$ python
Python 2.7.6 (default, Jun 22 2015, 17:58:13)
[GCC 4.8.2] on linux2
Type "help", "copyright", "credits" or "license" for more information.
>>> import SoundUtils
>>> dir(SoundUtils)
['__doc__', '__file__', '__name__', '__package__', 'metadata']
>>> from SoundUtils import metadata
>>> dir(metadata)
['__call__', '__class__', '__cmp__', '__delattr__', '__doc__', '__eq__', '__format__', '__ge__', '__getattribute__', '__gt__', '__hash__', '__init__', '__le__', '__lt__', '__module__', '__name__', '__ne__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__self__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__']
>>> a = metadata("a.flac")
>>> a
{'ALBUM': '\xe7\xa9\xba\xe4\xb9\x8b\xe5\xa2\x83\xe7\x95\x8c', 'ARTIST': '\xe6\xa2\xb6\xe6\xb5\xa6\xe7\x94\xb1\xe8\xa8\x98'}
>>> for k, v in a.iteritems():
...     print k.lower()
...     print v
...
album
空之境界
artist
梶浦由記
>>>
```

到这里完事儿，接下来只需要把想要的部分再做迁移就可以咯～
