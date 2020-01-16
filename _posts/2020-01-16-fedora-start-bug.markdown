---
layout: post
title: Fedora30 物理断电后重启问题
author: Sindweller-ch <jinghan.chen@cs2c.com.cn>
tags: [Fedora]
---
## Fedora30 物理断电后重启问题
Fedora30 **物理重启** 后，停留在
```
generating /run/initramfs/rdsosreport.txt
```
进入journalctl查看log，发现报错：
```
Failed to start File System Check on /dev/mapper/fedora-root.
```
另有提醒**fsck failed**，说明是文件系统的问题，且修复失败。
### 解决办法
```
fsck /dev/mapper/fedora-root
```
重启即可。注意根据**具体报错**修改指令(fedora-root)。
- 注：重启后发现点击图标进入文件系统会崩溃，不知是否与文件系统未完全修复有关。其他启动正常。
