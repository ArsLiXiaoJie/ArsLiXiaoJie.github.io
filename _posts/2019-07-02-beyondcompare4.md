---
layout: post
title:  "Beyond Compare4提示\"这个授权密钥已被吊销\""
date:   2019-07-02 13:31:01 +0800
categories: tool
tag: tool
typora-root-url: ../../arslixiaojie.github.io
---



打开beyond compare4 提示 **这个授权密钥已被吊销**

解决方法就是删除以下目录中的所有文件

**C:/Users/Administrator/AppData/Roaming/Scooter Software/Beyond Compare 4**

python代码

> ```python
> #!/usr/bin/env python
> #-*-coding:utf-8-*-
> import os
> PATH = "C:/Users/Administrator/AppData/Roaming/Scooter Software/Beyond Compare 4/"
> 
> files = os.listdir(PATH)
> for file in files:
> 	print(u"删除" + file)
> 	path = PATH + file
> 	os.remove(path)
> ```
>
> 

