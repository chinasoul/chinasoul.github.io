---
layout:     post
title:      "安装Linux踩过的坑和吐槽"
subtitle:   "谁来与我大战三百回合！"
date:       2021-04-01
author:     "chinasoul"
header-img: "img/post-bg-4.jpg"
catalog: true
tags:
    - linux
---
## Linux的安装坑

自从最近用上virt-manager，gpu也能方便的passthrough到guest，打算把常见的系统都装成vm，碰到不少值得吐槽的地方和坑

1. fedora33/opensuse tumbleweed

   需要手动开启ssh service，可能针对所有rpm based

2. opensuse tumbleweed

   ssh服务打开后无法login，需要把firewall-interfaces里的网卡由default设成external
   
   [https://en.opensuse.org/SDB:OpenSSH_basics](https://en.opensuse.org/SDB:OpenSSH_basics)