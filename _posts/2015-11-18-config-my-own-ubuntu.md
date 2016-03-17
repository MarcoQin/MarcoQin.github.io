---

layout: post
title: 打造适合自己的ubuntu
category : system
tags : [system, ubuntu]

---

从刚开始接触ubuntu，到现在成为我的主力工作娱乐系统，中间断断续续地做了无数次配置。

每次做新系统时，配置总是最令人头疼的。因为记性太差，过往觉得非常适合自己的配置，或是
漂亮的主题，在做新系统时，总会因为忘记而重新搜索一遍，浪费大量的精力在上面。

因此，借此准备更新配置重做系统的前夕，将配置及工具整理一遍。是不是应该把题目改成：《从零开始打造属于自己的ubuntu》更
能吸引眼球哈……嘛，反正是给自己备忘用的，估计也写不了多普适，还是就这样吧。

***

**配置字体，语言**

- 字体选用文泉驿微黑，在中文显示方面很不错。
安装`sudo aptitude install ttf-wqy-microhei`

- 使用[ubuntu-tweak](http://ubuntu-tweak.com/) 或者`sudo apt-get install gnome-tweak-tool`来进行设置。

我的字体配置如下：

    {% capture images %} /images/tweak_fonts_conf.png {% endcapture %} {% include gallery images=images caption="Fonts Setting" cols=1 %}

- **英文环境确保中文字体显示正确**
习惯使用英文系统，所以在浏览中文网页的时候，总会出现字体混乱的情况，特别难看。解决方法也简单：

- 编辑/etc/fonts/conf.d/69-language-selector-zh-cn.conf，将其中`<test></test>test>`包围的代码快儿全部删掉，保存即可。

***

**ATI显卡驱动**

诶，ubuntu下ATI显卡驱动要想好好用真是老大难啊。。现阶段对我来说稳定的解决方案是直接用“附加驱动”中的驱动。如下：

    {% capture images %} /images/ATI_additional_drivers.png {% endcapture %} {% include gallery images=images caption="ATI Driver Setting" cols=1 %}

***

**安装gnome3**

自带的桌面实在难看，用我大gnome才爽。

**更新：**

ubuntu14.04安装gnome 3.12.x的时候，不是很稳定。所以直接安装官方提供源的gnome 3.10.04。

```bash
sudo apt-get install gnome-shell gnome-shell-extensions
sudo apt-get install ubuntu-gnome-desktop
```

**以下为安装最新版gnome方法**
```bash
sudo add-apt-repository ppa:gnome3-team/gnome3-staging
sudo add-apt-repository ppa:gnome3-team/gnome3
sudo apt-get update
sudo apt-get install gnome-shell gnome-shell-extensions
sudo apt-get install ubuntu-gnome-desktop
```

***注意：***可能会报错，如`The following packages have unmet dependencies:`之类。不要慌，直接尝试安装其中的依赖，然后再尝试安装gnome-shell。


***

**主题设定**

漂亮最重要！难看的系统能让人抓狂！

- **安装numix主题**

    1. `sudo add-apt-repository ppa:numix/ppa`
    2. `sudo apt-get update`
    3. `sudo apt-get install numix-gtk-theme numix-icon-theme-circle`

- **安装KDE5的breeze主题(鼠标主题)**
    <del>1. `sudo apt-add-repository ppa:kubuntu-ppa/next`</del>
    1. add `deb http://us.archive.ubuntu.com/ubuntu vivid main universe` to  '/etc/apt/sources.list'
    2. `sudo apt-get update`
    3. `sudo apt-get install breeze-cursor-theme`

- 在tweak-tool中进行设置。我的设置如下：

    {% capture images %} /images/tweak_theme_conf.png {% endcapture %} {% include gallery images=images caption="Theme Conf" cols=1 %}

- 下方的dock栏的整法：
    1. `sudo apt-get install docky`
    2. docky栏图标出现问题时（比如模糊），google之。

- **Terminal配色**
    Terminal配色自然要用solarized，配置vim的时候一并整了。链接[solarized](https://github.com/Anthony25/gnome-terminal-colors-solarized)

- **Terminal tab样式修改**
    <del>在本博客中有。。上上篇吧。</del>
    更新：tab栏有变化是因为装了gnome 3.12.x，某些东西不兼容现在系统。换装3.10.04后，tab栏很正常，也很好看。

***

**Zsh&Vim配置**

重中之重。Zsh用[oh-my-zsh](https://github.com/robbyrussell/oh-my-zsh) 的配置，轻松搞定，然后从这个博客把配置拉下去merge一遍
就可以了。

Vim自然使用自己修改过的[k-vim](https://github.com/MarcoQin/k-vim) 跟着步骤很快配完。

发现配好这两个后，基本什么都ok了。。要求如此之简单。。

***

**杂项**

- 输入法：fctix
    1. [安装fctix](http://www.cnblogs.com/yuemengke/archive/2013/04/09/3010207.html)
    2. [这个也是](http://blog.csdn.net/tecn14/article/details/24784047)
- timidity 播放midi文件。
- mplayer 播放器
- java (Oracle版)[按这里](https://www.digitalocean.com/community/tutorials/how-to-install-java-on-ubuntu-with-apt-get)
- 配置google等的hosts（在stars里有）
- 配置vpn
    1. 在gnome中，没法直接从系统设置里配置vpn。但是可以在命令行中输入`nm-connection-editor`打开原始的网络配置器进行配置。
    2. vpn及别的网络配置在/etc/NetworkManager/system-connections目录下。可以把以前备份的配置拷贝过去。
- 安装jekyll[这里](http://jekyllrb.com/docs/installation/) 中途ruby安装可能需要翻墙，注意。执行`jekyll serve`可能
    也会变成`bundle exec jekyll serve`。
- pdftk: 合并pdf文件. 使用方式： pdftk [.pdf files] output [output file]
- Okular: pdf文件阅读器，非常好用，非常快，比自带的documents viewer快无数倍！
- wps for linux: 比自带的libra office速度快很多，尤其是表格软件，简直秒杀。
- xrecode2: windows下最好用的开源音频格式转换软件，非常好用你。在ubuntu14.04下完美使用（wine）。
- MySQL Workbench: ubuntu下目前感觉最好用的mysql客户端，无论是UI还是操作，都是一流的。当然，要是mac的话，推荐用seque pro..


**可能未完待续，想起来再更新...**
