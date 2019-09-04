---
layout: post
title:  oVirt 架构预览
author: Joey <majunjiev@gmail.com>
tags: [Virtualization]
---

本文翻译自 oVirt 官网文章：[oVirt Arch](https://www.ovirt.org/documentation/architecture/architecture/)，介绍了 oVirt 的系统架构图。

<!--more-->

先上一张 oVirt 的模块/接口概览图。
![oVirt-illustration](https://www.ovirt.org/images/wiki/Ovirt-1024x698.png?1478101462) 

## oVirt 架构
一套完整的 oVirt 部署主要包含如下：
* ovirt-engine： 管理平台，提供deploy/monitor/move/stop/create vm，配置存储和网络等
* ovirt-nodes： 运行 vm 的主机
* 存储节点：存储虚拟机镜像和 ISO

### 总体架构
下面这张图给出了 oVirt 中的不同组件。
![oVirt-Arch](https://www.ovirt.org/images/wiki/Architecture.png?1478101463)

特别说明：
* vdsm（Host Agent）：运行在 ovirt-nodes 上，为 ovirt-engine 提供 vm 相关操作
* Guest Agent：运行在虚拟机内部，监控资源使用并上报给 vdsm，通信方式是**虚拟 serial**
* DWH：数据仓库，执行 ETL 后将数据写入 history DB
* Reports Engine：基于 Jasper Reports，根据 history DB 中的数据生成报表

## ovirt-engine
ovirt-engine 是一个基于 JBoss 的 Java Web 应用，其直接向**vdsm 发送基于 XML-RPC**的指令。
主要功能包括：
* vm 全生命周期管理
* 网络、存储、镜像管理
* vm高可用 - 可自动在可用主机上重启失败的 vm
* 系统调度 - 根据资源 使用情况/策略，在 vm 间做 load balance
* 监控 - 监控系统内所有的实体运行情况
* 导入/导出 - 使用 OVF 格式导入导出 vm 和模板
* V2V - 将 VMware、RHEL/Xen 的 vm 导出到 oVirt 中

下图画出了 ovirt-engine 中各组件的分层情况：
![ovirt-engine-layers](https://www.ovirt.org/images/wiki/Engine-arch.png?1478101462)

### ovirt-engine-core 架构
下图给出了 ovirt-engine-core 的组件图：
![ovirt-engine-core 组件图](https://www.ovirt.org/images/wiki/Engine-arch2.png?1478101462)
主要包括：
* DB Broker - 执行所有数据库操作
* VDS Broker - 执行所有与 vdsm 的通信

## vdsm
vdsm 基于 Python 开发，常驻于 ovirt-nodes 上，其功能涵盖了ovirt-engine 所需的所有功能。
* vdsm API 基于 XML-RPC
* 使用 libvirt 库实现 vm 生命周期管理
* 自身多线程、多进程
* __通过 virtio-serial 与 guest agent通信__

vdsm 架构图如下图所示：
![vdsm-arch](https://www.ovirt.org/images/wiki/Vdsm-arch.png?1478101462)

### Hooks 机制
Hooks 机制是 vdsm 特有的一项功能，其具有如下特点：
* 提供在特定事件时，执行自定义脚本修改 vm 配置
* 支持 oVirt 集成新的 KVM 功能
* 提供更简单的方式测试新的 KVM/libvirt/linux 功能

下图说明了 Hooks 在 vm 生命周期中被执行的时机：
![hooks](https://www.ovirt.org/images/wiki/Hook-arch.png?1478101462)



