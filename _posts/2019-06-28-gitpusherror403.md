---
layout: post
title:  "The requested URL returned error: 403解决"
date:   2019-06-28 14:31:01 +0800
categories: git
tag: git
typora-root-url: ../../arslixiaojie.github.io
---

今天git push 的时候报这个错误，发现是自己clone下来的时候没有加上用户名和密码。

**git clone https://github.com/gitname/xxx.git**

clone的时候可以加上用户名，这样会在随后提示你输入密码

**git clone https://usrename@github.com/gitname/xxx.git**

也可以在clone的时候加上密码

**git clone https://username:password@github.com/gitname/xxx.git**



可是像我这种情况，已经clone下来了

**git remote set-url origin https://username:password@github.com/gitname/xxx.git**

