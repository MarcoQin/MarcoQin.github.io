---

layout: post
title: Gentoo的一些坑及解决方式
category : linux
tags : [linux]

---

在使用Gentoo的时候，配置上总是出一些奇怪且令人抓狂的事情。

有些问题解决了许久，今日随意一想居然有要忘掉的感觉。想到当初搜索的艰难，还是自己记下来靠谱。

想到什么写什么，不断备忘吧。

####emerge profile问题

在使用emerge包管理器的时候，初始可能会有一些配置错误，导致根本无法使用。

例如`Found 2 make.conf files`。解决此问题也简单，直接将一个合并到另一个然后删除其中一个就可以了。

再例如：

> Repository 'x-portage' is missing masters attribute in '/usr/portage/metadata/layout.conf'

> Set 'masters = gentoo' in this file for future compatibility

直接创建那个文件，然后将缺的东西写进去就可以了。

上面两个都搞定后， 一定要记得执行`emerge --sync`，可能需要sudo权限。这个需要很久，推荐在tmux中整，省得session断了白费功夫。

比较麻烦的是这个：

> /etc/portage/make.profile is not a symlink and will probably prevent most merges.

> It should point into a profile within /usr/portage/profiles/

google找了一圈，各种办法试过后，终于找到了解决办法：

**eselect profile list**

执行后会列出所有的profile列表如下：

```bash
Available profile symlink targets:
  [1]   default/linux/amd64/13.0
  [2]   default/linux/amd64/13.0/selinux
  [3]   default/linux/amd64/13.0/desktop
  [4]   default/linux/amd64/13.0/desktop/gnome *
  [5]   default/linux/amd64/13.0/desktop/gnome/systemd
  [6]   default/linux/amd64/13.0/desktop/kde
  [7]   default/linux/amd64/13.0/desktop/kde/systemd
  [8]   default/linux/amd64/13.0/desktop/plasma
...
```

然后用**eselect profile set 4**来设置profile，之后emerge就可以恢复正常了。

emerge好了后，剩余的坑也就好多了。想起来后再补充。
