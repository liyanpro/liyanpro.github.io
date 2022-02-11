---
title: Linux命令学习
date: 2022-02-10 17:38:56
tags:
    - Linux
---

##### expect命令

​      脚本文件test


    set user username
    set host ip
    set password  pass
    set port 8802
    spawn ssh $user@$host -p $port
    expect "*assword:*"
    send "$password\r"
    interact
    expect eof


执行expect脚本:		


    expect  ~/test


