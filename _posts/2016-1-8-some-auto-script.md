---

layout: post
title: 一些偷懒的自动脚本
category : bash
tags : [bash, expect]

---

实在懒得不行，git pull别人的代码时不想输入用户名密码。。登录别人的docker机器时也不想输密码。。

所以写了两个脚本来自动化之。。

首先是自动pull：


```bash
#!/bin/sh

username="USERNAMEK"
password="PASSWORD"

get_help ()
{
    echo -e "\033[34m\033[1mUsage: \033[0m"
    echo -e "\t ap \033[32mwho\033[0m \033[32mbranch\033[0m"
    echo -e "OR"
    echo -e "\t ap"
}

warning ()
{
    echo -e "\033[31mWARNING:\033[0m \033[1mCommit or stash everything you've changed before use this script!\033[0m"
    echo -e "\033[31mWARNING:\033[0m Set your username and password in this script first! \033[31mBut It's not security!\033[0m"
    echo -e "\033[32mFor help:\033[0m  ap -h   or   ap --help"
    echo
}

enter_who() {
    echo -e -n "Enter \033[33mwho's\033[0m code you want to pull: "
    read who
}

enter_branch() {
    echo -e -n "Enter \033[33mbranch\033[0m name: "
    read branch
}

argc=$#

warning

if [ $argc -eq 0 ]
then
    enter_who
    enter_branch
else
    if [ $argc -eq 1 ]
    then
        if [[ $1 = "-h" || $1 = "--help" ]]
        then
            get_help
            exit
        else
            who=$1
            enter_branch
        fi
    else
        who=$1
        branch=$2
    fi
fi

expect -c '
spawn git pull '"$who"' '"$branch"'
expect -nocase "Username*" { send "'"$username"'\r" }
expect -nocase "Password*" { send "'"$password"'\r"; interact }
'
```

之所以用expect -c而不是重定向EOF（/usr/bin/expect << EOF），是因为实在是有个地方调不通。。所以这个办法比较好用。。

注意： 使用expect -c时，要调用外部变量，需要在变量外包一层单双引号才可以，类似这样： '"$var"'。

把脚本命名为`ap`(auto pull)，给个-x权限然后扔到/usr/local/bin下面，就可以直接使用了。

还有个自动ssh脚本。。

```bash
#!/bin/bash

password="PASWORD"
argc=$#
if [ $argc -lt 1 ]
then
    echo -n Enter port:
    read port
else
    port=$1
fi


expect -c '
spawn ssh -p '"$port"' loginname@host
expect -nocase "password*"
send ""'$password'"\r"
interact'

```

同样好用。。仅针对个人好用哈。。
