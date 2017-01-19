---
title: windows查看端口占用
date: 2017-01-12 13:47:40
categories: 杂
tags: Windows
---
1. 运行命令行或者cmd+r
2. 执行命令`netstat -ano`查看端口占用情况
3. 查看被占用端口对应的PID，输入命令：netstat -ano|findstr "49157"
4. 继续输入tasklist|findstr "2720"，回车，查看是哪个进程或者程序占用了2720端口，结果是：svchost.exe
5. 或者打开任务管理器ctrl + shift + esc 查找对应PID 然后kill掉
