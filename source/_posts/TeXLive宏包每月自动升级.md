---
layout: post
title: TeXLive 宏包每月自动升级
tags:
  - TeXLive
  - SchTasks
comments: true
translate_title: texlive-packages-are-automatically-upgraded-every-month
categories: LaTeX
date: 2018-08-09 10:19:07
---
{% note warning %}
TeXLive 是一个很好的 TeX 发行版，但是其宏包管理工具做得不太好 —— tlmgr 的 GUI 比较卡顿，而且不能够定期升级。
本文提供一个每月自动升级的脚本，可供大家参考。
{% endnote %}
<!-- more -->

## 脚本下载

{% btn http://pd10ibe5c.bkt.clouddn.com/TeXLive%E5%AE%8F%E5%8C%85%E6%AF%8F%E6%9C%88%E8%87%AA%E5%8A%A8%E6%9B%B4%E6%96%B0.zip, 点击下载, download fa-lg fa-fw %}

## 脚本源码

``` 
@echo off
if exist "C:\Windows\Tasks\AutoTeXLivePackageUpdaterMonthly.bat" goto run
move /y %0 "C:\Windows\Tasks"
schtasks /delete /tn "TeXLivePackage Updater Task" /f
schtasks /create /tn "TeXLivePackage Updater Task" /sc monthly /d /st 15:00:00 /tr "C:\Windows\Tasks\AutoTeXLivePackageUpdaterMonthly.bat"
:run
echo ============================开始============================
echo Writen By 有龙则灵_USTB
echo 是否更新TeXLive Package？
set Choice=
set /p Choice=请输入：y/n?
IF "%Choice%"=="y" (goto ya) else (goto n)
:ya
call tlmgr option repository http://mirror.ctan.org/systems/texlive/tlnet
echo ============================更新tlmgr============================
echo Writen By 有龙则灵_USTB
call tlmgr update --self
echo ============================显示待更新的宏包以及可自动安装的项============================
call tlmgr update --list
echo Writen By 有龙则灵_USTB
echo 是否更新TeXLive Package？
set Choice=
set /p Choice=请输入：y/n?
IF "%Choice%"=="y" (goto yb) else (goto n)
:yb
echo ============================更新所有宏包============================
call tlmgr update --all
echo ============================结束============================
echo Writen By 有龙则灵_USTB
pause
:n
```

## 脚本阐释

### 利用 Windows 自带的 SchTasks 创建定时任务

第一部分用于将该脚本移动到定时任务的根目录，并创建一个计划任务项。
为什么不用 AT 呢？因为 AT 在 Win10 中已经被取缔了。

```
if exist "C:\Windows\Tasks\autoTeXLivePackageUpdaterMonthly.bat" goto run
move /y %0 "C:\Windows\Tasks"
schtasks /delete /tn "TeXLivePackage Updater Task" /f
schtasks /create /tn "TeXLivePackage Updater Task" /sc monthly /d /st 15:00:00 /tr "C:\Windows\Tasks\TeXLivePackageUpdater.bat"
```

更多关于计划任务的操作，可以去搜索，也可以参考[这篇文章](https://www.flighty.cn/html/tutorial/20170406_442.html)。

### 调用 tlmgr 进行更新

第二部分是调用 tlmgr 进行更新TeXLive宏包。

```
tlmgr option repository http://mirror.ctan.org/systems/texlive/tlnet
tlmgr update --self
tlmgr update --list
tlmgr update --all --no-auto-install
```

以上四条命令分别实现的是**选取宏包源、更新 tlmgr 自身、列出可更新的宏包名、更新所有宏包**。
--no-auto-install 实现的是不自动安装。众所周知 TeXLive 是发行几乎所有投稿的宏包，所有每次更新里面都有太多自动安装的宏包。如果你想要这个功能，删掉这个参数即可。

更多关于 tlmgr 的操作，请参考[官方文档](https://www.tug.org/texlive/doc/tlmgr.html)。

### 批处理编写
代码里面其余部分均是 bat 编程的基本语句，可参考[百度百科](https://baike.baidu.com/item/%E6%89%B9%E5%A4%84%E7%90%86/1448600?fr=aladdin)。
