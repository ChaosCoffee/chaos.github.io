---
title: 利用Chrome原生工具进行网页长截图
comments: false
date: 2018-08-15 11:10:57
categories: tools
tags: 
img:
---
# Chrome原生工具进行网页长截图
<!-- TOC -->

- [Chrome原生工具进行网页长截图](#chrome原生工具进行网页长截图)
    - [准备工作](#准备工作)
        - [打开调试界面](#打开调试界面)
    - [网页截图](#网页截图)
        - [普通网页长截图](#普通网页长截图)
            - [1. 打开网页截图工具](#1-打开网页截图工具)
            - [2. 输入截图命令](#2-输入截图命令)
            - [3. 截图](#3-截图)
        - [手机版网页长截图](#手机版网页长截图)
            - [1. 模拟手机设备](#1-模拟手机设备)
            - [2. 手机截图](#2-手机截图)
            - [3. 输入命令](#3-输入命令)
    - [区域截图](#区域截图)
        - [1. 区域选中](#1-区域选中)
        - [2. 截图命令](#2-截图命令)
    - [扩展](#扩展)

<!-- /TOC -->

## 准备工作
在想要截图的网页中，按着以下快捷键  
### 打开调试界面  

> MacOs `⌘Command + ⌥Option + I`  
> Windows 为 `F12`  
> 
![image](https://raw.githubusercontent.com/ChaosCoffee/ChaosCoffee.github.io/master/img/chrome_debug.png)


## 网页截图
### 普通网页长截图

![image](https://raw.githubusercontent.com/ChaosCoffee/ChaosCoffee.github.io/master/img/capture_full_size_screenshot.png)

#### 1. 打开网页截图工具  
> MacOs `⌘Command + ⇧Shift + P`  
> Window `Ctrl + Shift + P`

#### 2. 输入截图命令  
输入前几个字母就会匹配到,输入以下命令  
`Capture full size screenshot`
   
#### 3. 截图  
选中所在的命令,敲下回车 `Enter`


### 手机版网页长截图
#### 1. 模拟手机设备  
> MacOs `⌘Command + ⇧Shift + M`   
> Windows `Ctrl + Shift + M`  

**注意:** 后面再使用普通的网页截图，仍然使用以上的命令进行切换
   
#### 2. 手机截图  
在顶部的工具栏中，你可以选择要模拟的设备和分辨率等设置。

#### 3. 输入命令  
与 *普通网页网页长截图* 中`1. 2. 3.`步骤相同
   
## 区域截图

![image](https://raw.githubusercontent.com/ChaosCoffee/ChaosCoffee.github.io/master/img/capture_node_screenshot.png)  

### 1. 区域选中
> MacOs `⌘Command + ⇧Shift + C`  
> Windows  Ctrl + Shift + C  
  
按照上面命令然后鼠标悬浮在上面,出现蓝色选中区域为止

### 2. 截图命令
先打开截图工具(参见: [普通网页长截图 步骤1](#1-打开网页截图工具)),  
再按着命令 `Capture node screenshot` 实现区域截图

## 扩展
`Capture full size screenshot` 命令实现完成网页截图  
`Capture node screenshot` 命令实现区域截图  
`Capture screenshot` 命令可以截取当前网页的可视部分  