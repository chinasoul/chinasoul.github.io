---
layout:     post
title:      "xfce桌面无法启动gparted"
subtitle:   "万里悲秋常作客，百年多病独登台"
date:       2021-04-01
author:     "chinasoul"
header-img: "img/post-bg-3.jpg"
catalog: true
tags:
    - linux
---
## xfce desktop cannot launch gparted which requires pwd

默认的gnome desktop启动会弹出要求输入密码，但是在xfce环境下没任何反应。

xfce启动项在

```
marvelz@marvelz-ub20:/usr/share/applications$ ls|grep -i gparted
gparted.desktop
```

其中

Exec=/usr/sbin/gparted %f

所以需要拷贝该文件到~/.local/share/applications然后加sudo

(直接修改也行，不过软件升级后会失效)