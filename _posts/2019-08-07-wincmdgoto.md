---
layout: post
title:  "bat 打开命令行并进入当前目录"
date:   2019-08-07 15:00:00 +0800
categories: windows
tag: windows
typora-root-url: ../../arslixiaojie.github.io
---

在某个目录下，执行bat脚本，自动打开cmd并且进入到当前目录

```bash
@echo off
set dir=%cd%
start cmd /K "cd /d %dir%"
```

