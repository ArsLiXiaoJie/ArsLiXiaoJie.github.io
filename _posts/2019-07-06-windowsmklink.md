---
layout: post
title:  "windows mklink为文件和目录创建链接"
date:   2019-07-06 19:31:01 +0800
categories: windows
tag: windows
typora-root-url: ../../arslixiaojie.github.io
---

windows 的 **mklink** 命令和 linux 上的 ln 命令功能类似。

![](/../../assets/mklink.png)

### 不带参数

不带参数的 mklink 命令可以为**文件**创建符号链接。

当源路径是目录时，命令不会报错，但是生成的链接不可用。

删除不带参数的mklink创建的符号链接，不会影响源路径指向的文件，删除源文件则符号链接无法访问。

```bash
C:\Users\Administrator\Desktop\testmklink>mklink link.txt ttt.txt
为 link.txt <<===>> ttt.txt 创建的符号链接

C:\Users\Administrator\Desktop\testmklink>mklink link2 link
为 link2 <<===>> link 创建的符号链接

C:\Users\Administrator\Desktop\testmklink>dir
 驱动器 C 中的卷是 Win7x64
 卷的序列号是 4C5D-1703

 C:\Users\Administrator\Desktop\testmklink 的目录

2019/07/06  20:07    <DIR>          .
2019/07/06  20:07    <DIR>          ..
2019/07/06  19:39    <DIR>          link
2019/07/06  20:07    <SYMLINK>      link.txt [ttt.txt]
2019/07/06  20:07    <SYMLINK>      link2 [link]
2019/07/06  19:35                 9 ttt.txt
               3 个文件              9 字节
               3 个目录 18,169,765,888 可用字节

```

### 带参数 /D

带上参数 /D 的mklink可以为**目录**创建符号链接。

当源路径是文件时，命令不会报错，但是生成的链接不可用。

删除带参数/D的mklink命令创建的符号链接，不会影响源路径指向的文件，删除源文件则符号链接无法访问。

```ba
C:\Users\Administrator\Desktop\testmklink>mklink /D link.txt ttt.txt
为 link.txt <<===>> ttt.txt 创建的符号链接

C:\Users\Administrator\Desktop\testmklink>mklink /D link2 link
为 link2 <<===>> link 创建的符号链接

C:\Users\Administrator\Desktop\testmklink>dir
 驱动器 C 中的卷是 Win7x64
 卷的序列号是 4C5D-1703

 C:\Users\Administrator\Desktop\testmklink 的目录

2019/07/06  20:10    <DIR>          .
2019/07/06  20:10    <DIR>          ..
2019/07/06  19:39    <DIR>          link
2019/07/06  20:10    <SYMLINKD>     link.txt [ttt.txt]
2019/07/06  20:10    <SYMLINKD>     link2 [link]
2019/07/06  19:35                 9 ttt.txt
               1 个文件              9 字节
               5 个目录 18,170,404,864 可用字节
```

### 带参数/H

带上参数 /H 的mklink命令可以为文件创建硬链接。

当源路径为目录时，带参数 /H 会报错，只能对文件使用。

硬链接的样式跟正常文件没什么不同，相当于拷贝了一份，且相互之间是联系在一起的。

删除硬链接不会影响源文件，删除源文件也不会影响硬链接，只有当一个文件的所有硬链接和源文件都被删除时，文件才被真正删除。

```bash
C:\Users\Administrator\Desktop\testmklink>mklink /H link.txt ttt.txt
为 link.txt <<===>> ttt.txt 创建了硬链接

C:\Users\Administrator\Desktop\testmklink>mklink /H link2 link
拒绝访问。

C:\Users\Administrator\Desktop\testmklink>dir
 驱动器 C 中的卷是 Win7x64
 卷的序列号是 4C5D-1703

 C:\Users\Administrator\Desktop\testmklink 的目录

2019/07/06  20:14    <DIR>          .
2019/07/06  20:14    <DIR>          ..
2019/07/06  19:39    <DIR>          link
2019/07/06  19:35                 9 link.txt
2019/07/06  19:35                 9 ttt.txt
               2 个文件             18 字节
               3 个目录 18,169,892,864 可用字节
```



### 带参数/J

带上参数 /J 的mklink命令可以为目录创建链接。

当源文件是文件时，带参数 /J 不会报错，但是生成的链接不可用。

在资源管理器中可以看到链接的类型是文件夹。

删除 mklink /J 命令生成的链接，不会影响源路径，删除源路径则链接无法访问。

```bash
C:\Users\Administrator\Desktop\testmklink>mklink /J link2 link
为 link2 <<===>> link 创建的联接

C:\Users\Administrator\Desktop\testmklink>mklink /J link.txt ttt.txt
为 link.txt <<===>> ttt.txt 创建的联接

C:\Users\Administrator\Desktop\testmklink>dir
 驱动器 C 中的卷是 Win7x64
 卷的序列号是 4C5D-1703

 C:\Users\Administrator\Desktop\testmklink 的目录

2019/07/06  20:22    <DIR>          .
2019/07/06  20:22    <DIR>          ..
2019/07/06  19:39    <DIR>          link
2019/07/06  20:22    <JUNCTION>     link.txt [C:\Users\Administrator\Desktop\testmklink\ttt.txt]
2019/07/06  20:20    <JUNCTION>     link2 [C:\Users\Administrator\Desktop\testmklink\link]
2019/07/06  20:16                13 ttt.txt
               1 个文件             13 字节
               5 个目录 18,172,420,096 可用字节

```

