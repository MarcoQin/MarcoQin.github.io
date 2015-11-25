---

layout: post
title: tmux配置备份及说明
category : Linux
tagline: "Supporting tagline"
tags : [tmux, docker]

---

目前在docker环境开发，用tmux十分方便。

默认设置太难用，稍作修改放到这里备忘。
tmux 默认操作如下，只拿常用的我记得的说。。

>在bash界面：
>
>   * tmux  打开tmux
>
>   * tmux ls 列出所有的session
>
>   * tmux attach 直接进入最后一次登出的session中
>
>   * tmux attach-session -t <session-name> 进入名为<session-name>的session中
>
>   * tmux new-session -s <session-name> 创建名为<session-name>的session，并进入
>
>在tmux界面：
>
>   * 所有的命令均得先按前缀C-b
>
>   * s 打开目前所有的会话列表
>
>   * d 保存当前会话并退出tmux（切换到终端界面）
>
>   * c 新建tmux tab
>
>   * % 当前tmux tab中垂直分割一个panel
>
>   * " 当前tmux tab中水平分割一个panel
>
>   * C-方向键 设置当前pannel大小
>
>   * n 切换到下一个tmux tab
>
>   * p 切换到上一个tmux tab
>
>   * 数字切换到第n个tmux tab
>
>   * [ 切换到复制模式，光标可以上下移动
>
>   * : 输入命令状态


配置文件
file: ~/.tmux.conf

第一次可以在tmux命令模式下使用`source-file ~/.tmux.conf`

{% highlight bash%}
unbind C-b  # 解除旧的前缀绑定
bind r source-file ~/.tmux.conf \; display "Reloaded!"  # 绑定r建为reload conf文件
bind-key k select-pane -U  # hjkl切换panel
bind-key j select-pane -D
bind-key h select-pane -L
bind-key l select-pane -R
setw -g mode-keys vi  # 复制模式使用vi的按键（跳转、搜索等）
bind-key -t vi-copy 'v' begin-selection
bind-key -t vi-copy 'y' copy-selection

#### COLOUR (Solarized 256)

# default statusbar colors
set-option -g status-bg colour235 #base02
set-option -g status-fg colour136 #yellow
set-option -g status-attr default

# default window title colors
set-window-option -g window-status-fg colour244 #base0
set-window-option -g window-status-bg default
#set-window-option -g window-status-attr dim

# active window title colors
set-window-option -g window-status-current-fg colour166 #orange
set-window-option -g window-status-current-bg default
#set-window-option -g window-status-current-attr bright

# pane border
set-option -g pane-border-fg black #base02
set-option -g pane-active-border-fg brightgreen #base01

# message text
set-option -g message-bg colour235 #base02
set-option -g message-fg colour166 #orange

# pane number display
set-option -g display-panes-active-colour colour33 #blue
set-option -g display-panes-colour colour166 #orange

# clock
set-window-option -g clock-mode-colour colour64 #green

# bell
set-window-option -g window-status-bell-style fg=colour235,bg=colour160 #base02, red
{%endhighlight%}
