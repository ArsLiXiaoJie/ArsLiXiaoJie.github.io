---
layout: post
title:  "bat 获取管理员权限执行命令"
date:   2019-08-07 15:00:00 +0800
categories: windows
tag: windows
typora-root-url: ../../arslixiaojie.github.io
---

在用mklink命令的时候，提示我没有管理员权限，网上查了下：

```bash
REM 在前面加上这段，在 :Admin 后面加自己的脚本
@echo off
set dir=%~dp0
cd /d "%~dp0"
cacls.exe "%SystemDrive%\System Volume Information" >nul 2>nul
if %errorlevel%==0 goto Admin
if exist "%temp%\getadmin.vbs" del /f /q "%temp%\getadmin.vbs"
echo Set RequestUAC = CreateObject^("Shell.Application"^)>"%temp%\getadmin.vbs"
echo RequestUAC.ShellExecute "%~s0","","","runas",1 >>"%temp%\getadmin.vbs"
echo WScript.Quit >>"%temp%\getadmin.vbs"
"%temp%\getadmin.vbs" /f
if exist "%temp%\getadmin.vbs" del /f /q "%temp%\getadmin.vbs"
exit

:Admin

rd /S /Q C:\work\xxx\proj.android\app\src\main\assets\src
rd /S /Q C:\work\xxx\proj.android\app\src\main\assets\res
del C:\work\xxx\proj.android\app\src\main\assets\config.json

mklink /D C:\work\xxx\proj.android\app\src\main\assets\src %dir%\src
mklink /D C:\work\xxx\proj.android\app\src\main\assets\res %dir%\res
mklink C:\work\xxx\proj.android\app\src\main\assets\config.json %dir%\config.json

@pause
```

