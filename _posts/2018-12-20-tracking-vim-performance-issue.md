---
layout: post
title: 追溯 Vim 性能问题
category : vim
tagline: "Supporting tagline"
tags : [Vim, linux]
---

上次重新折腾了一下 Vim 后，虽然编辑 Go 代码还是很不错，但有严重的性能问题。
心里觉得还是很不爽，毕竟以前写 Python 用的同样的配置流畅的很，所以还是决定找出这么慢的原因。

搜了一下，[这里](https://stackoverflow.com/questions/12213597/how-to-see-which-plugins-are-making-vim-slow) 有说如何 profile Vim：

```
:profile start profile.log
:profile func *
:profile file *
" At this point do slow actions
:profile pause
:noautocmd qall!
```

得到的结果大致如下：

```
FUNCTIONS SORTED ON TOTAL TIME
count  total (s)   self (s)  function
   88   2.626294   0.018389  <SNR>193_VimCloseCallback()
  143   2.587495   0.023570  <SNR>156_HandleExit()
   85   2.333799   0.376396  ale#engine#HandleLoclist()
   88   1.856442   0.014765  <SNR>193_VimExitCallback()
   85   1.468189   0.026994  ale#engine#SetResults()
   85   1.046549   0.296071  ale#sign#SetSigns()
   47   1.034786   0.024842  <SNR>149_OnTextChangedInsertMode()
  214   1.017875   0.025560  ale#CallWithCooldown()
  321   0.974656   0.040110  airline#check_mode()
   43   0.945296   0.372632  <SNR>149_InvokeCompletion()
   22   0.918518   0.105577  airline#highlighter#highlight()
   42   0.841443   0.008024  ale#Lint()
   11   0.825597   0.006893  <SNR>151_AutoUpdate()
   42   0.819399   0.008091  <SNR>19_ALELintImpl()
```

可以简单看出 ale 这个插件占用了大量运行时间。ale 是 Vim 8 上用来支持各语言实时 error check 的一个工具，
而且是 async 的方式执行，Python 中用得同样是这个插件，却并没慢到哪去。

搜了一波发现也有 [其他人遇到性能问题](https://github.com/w0rp/ale/issues/965#issuecomment-336288413)，大致
可能是 go 文件中错误或不符合规范的 warning 太多了，导致每次检查都会触发大量绘制相关提示符号的操作，拖慢了速度。

改了两个配置： `let g:ale_max_signs = 10` &  `let g:ale_echo_delay = 100`，之后试了一下，发现速度有所提升，但还是
达不到理想状态。

继续 profile，然后关掉了 vim-go 的 fmt on autosave，并且将 ale 的 lint 模式也改了一下。最后配置如下：

```
let g:ale_lint_on_text_changed = 'normal'
let g:ale_lint_delay = 1000
let g:ale_lint_on_enter = 0
let g:go_fmt_autosave = 0
```

恩，现在速度飞快，很满意。
