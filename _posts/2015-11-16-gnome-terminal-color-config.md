---

layout: post
title: gnome shell tab 配色
category : gnome
tags : [shell, css]

---

将gnome shell的主题换成Numix后，整个界面显得好看多了， 颜色也很舒服。

唯一不爽的就是自带的terminal的标签栏，还是原来的白森森的一片，与我阴暗的主题一点都不符。

google之，找到解决方案，记录下来，省得以后再费劲儿找。

编辑~/.config/gtk-3.0/gtk.css, 将其中的样式改成如下，标签就会变成阴暗的色彩，嗯嗯，喜欢。

{% highlight css%}
/* gnome-terminal */
@define-color term-win-bg           #262626;
@define-color term-tab-inactive-bg  #333333;
@define-color term-tab-active-bg    #424242;
@define-color ubuntu-orange         #fb9267;

TerminalScreen {
    -TerminalScreen-background-darkness: 0.95;
    background-color: @term-win-bg;
}


TerminalWindow .notebook {
    border: 0;
    padding: 0;
}


TerminalWindow .notebook tab {
    border: 0;
    border-radius: 0px;
    border-image: -gtk-gradient (linear, left top, left bottom,
                                from (alpha (shade (@term-win-bg, 0.9), 0.0)),
                                to (shade (@term-win-bg, 0.9))) 1;
    border-image-width: 0 1px;
    border-color: transparent;
    border-width: 0;
    box-shadow: none;
    background-color: shade(@term-tab-inactive-bg, 1);
}


TerminalWindow .notebook tab:active {
    border: 0;
    border-radius: 0px;
    background-color: shade(@term-tab-active-bg, 1);
}


TerminalTabLabel.active-page .label {
    /*color: @bg_color;
    font-weight: bold
    color: @ubuntu-orange; */
    color: cyan;
}
{% endhighlight %}
