---
layout:     post
title:      "关于gpu passthrough后device not found"
subtitle:   "guest找不到device"
date:       2021-03-31
author:     "chinasoul"
header-img: "img/post-bg-2.jpg"
catalog: true
tags:
    - kvm
    - gpu
---
## ubuntu20 GPU passthrough to fedora33

##### 环境

- host: ubuntu20.04 desktop
- guest: fedora33
- KVM, Qemu, virt-manager, nvidia RTX3090
- 笔记本vnc访问host(host安装tigervnc+xfce)

##### 步骤
###### host passthrough gpu到guest

不赘述，基本步骤就是bios开虚拟化(我这里是intel，打开VT-d)，iommu打开(grub加intel_iommu=on)，vfio配好(grub加vfio-pci.ids=xxx:xxx，实测vfio conf文件不需要也可以)，update-grub就好。重启后dmesg看到iommu&vfio相关信息识别到gpu device ids信息即可。

```
[    0.038327] DMAR: IOMMU enabled
[    0.096381] DMAR-IR: IOAPIC id 2 under DRHD base  0xfed90000 IOMMU 0
[    0.227596] iommu: Default domain type: Passthrough (set via kernel command line)
[    0.346112] pci 0000:00:00.0: Adding to iommu group 0
```

```
[    0.400446] vfio_pci: add [10de:2204[ffffffff:ffffffff]] class 0x000000/00000000
[    0.420462] vfio_pci: add [10de:1aef[ffffffff:ffffffff]] class 0x000000/00000000
```

###### laptop远程vnc连到host操作

因为经常需要远程连公司台式机，而且台式机的显示器没接上，给笔记本用的双屏，所以之前一直使用tigervnc/tightvnc连台式机虚拟桌面，所以这里也打算vnc连接，然后virt-manager去操作guest machine，**这里坑了我2天，装好driver后nvidia-smi始终找不到gpu**。。。

##### 解决方案

1. 用virt-manager启动guest的时候删掉默认的"Display Spice"显示，然后查看ip，ssh进去

   ```
   virsh domifaddr fedora
    Name       MAC address          Protocol     Address
   -------------------------------------------------------------------------------
    vnet0      52:54:00:94:48:cd    ipv4         192.168.122.141/24
   # then ssh to gest and do your work
   ```

2. 不用virt-manager，直接用qemu命令行，这样任何地方都用不到vnc，缺点是无法使用图形化

3. host接上显示器，face to face直接操作host
