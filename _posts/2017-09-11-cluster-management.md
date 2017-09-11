---
layout:     post
title:      "集群管理利器-polysh"
subtitle:   "使用polysh同时查看多台服务器的日志"
date:       2017-09-11 11:40:00
author:     "Bryancy"
header-img: "img/home-bg-o.jpg"
tags:
    - linux
---

## 前言
在没有集成日志的情况下，想同时查看多台机器的日志目前知道的最好用的方法是使用polysh。它可以通过本地机器在远程同时执行命令，并显示结果。

[polysh主页](http://guichaz.free.fr/polysh/)


## 安装
`sudo pip install polysh`

## 用法
- 在多台机器执行同一命令
![](/img/in-post/post-cluster-ref.png)

- 在本地集中打印日志
![](/img/in-post/post-cluster-ref2.jpg)

