---
layout: post
title:  "Docker与Podman的对比分析"
date:   2020-02-21 17:10:04 +0800
author: Kiki<huihui.fu@cs2c.com.cn>
tags: [docker]
---
## 它们是啥

### Docker

Docker是一个开源的应用容器引擎，属于Linux容器的一种封装，Docker提供简单易用的容器使用接口，让开发者可以打包他们的应用以及依赖包到一个可移植的容器中，然后发布到任何流行的Linux机器上。容器是完全使用沙箱机制，相互之间不会有任何接口。

- [Docker官网地址：https://www.docker.com/](https://www.docker.com/)
- [Docker项目地址：https://github.com/docker/](https://github.com/docker/)

### Podman

Podman是一个无守护进程的容器引擎，用于在Linux系统上开发、管理和运行任何符合 OCI（Open Container Initiative）规范的容器和容器镜像。它可以在root权限下运行，也可以在没有root权限的情况下运行。

Podman提供的功能与Docker相似，大部分命令与Docker兼容，可以通过设置 alias docker=podman 将命令统一。

- [Podman官网地址：https://podman.io/](https://podman.io/)
- [Podman项目地址：https://github.com/containers/libpod](https://github.com/containers/libpod)

## 有啥不一样

1. Docker需要在系统上运行一个守护进程docker daemon，而Podman没有守护进程。Docker需要使用root用户创建容器，而Podman可以在没有root权限的情况下运行

    > Docker在Linux上作为守护进程运行，会产生一定的开销，并且还需要任何想创建容器的用户具有root访问权限，一旦授权，就可能存在安全风险。Podman的无守护进程架构就相对的更加灵活和安全。

2. 容器的启动方式不同：

    > Docker CLI命令通过gRPC API跟Docker Engine交互，通知engine需要创建一个container，然后Docker Engine会调用OCI container runtime(runc)来启动一个container。这就代表container的进程不是Docker CLI的子进程，而是Docker Engine的子进程。
    > Podman没有Daemon，它是直接跟OCI containner runtime(runc)进行交互来创建container的，所以container 的进程直接就是podman的子进程。

   相比较来说Podman这种交互模式就展现出了一些优势：

    - 系统管理员可以知道某个容器进程到底是谁启动的；
    - 如果利用cgroup对podman做一些限制，那么所有创建的容器都会被限制；
    - 如果将podman命令放入systemd单元文件中，容器进程可以通过podman返回通知，表明服务已准备好接收任务；
    - 可以将连接的socket从systemd传递到podman，并传递到容器进程以便使用它们

3. 容器自动重启的实现不同

    > Docker因为有docker daemon，因此docker启动的容器支持--restart策略
    > Podman没有守护进程，也就不能通过守护进程去实现自动重启容器的功能。若在k8s中，可以通过设置pod重启策略来实现；若在操作系统中，系统通常是以Systemd作为守护进程管理工具，因此可以使用Systemd来实现开机重启容器。

## 安装与配置

下面以Fedora 30安装为例，介绍Docker与Podman的安装配置流程。

### Docker的安装

#### 1. 添加Docker源

```bash
[root@localhost ~]# dnf config-manager --add-repo https://download.docker.com/linux/fedora/docker-ce.repo
添加仓库自：https://download.docker.com/linux/fedora/docker-ce.repo
[root@localhost ~]# cat /etc/yum.repos.d/docker-ce.repo
[docker-ce-stable]
name=Docker CE Stable - $basearch
baseurl=https://download.docker.com/linux/fedora/$releasever/$basearch/stable
enabled=1
gpgcheck=1
gpgkey=https://download.docker.com/linux/fedora/gpg

[docker-ce-stable-debuginfo]
name=Docker CE Stable - Debuginfo $basearch
baseurl=https://download.docker.com/linux/fedora/$releasever/debug-$basearch/stable
enabled=0
gpgcheck=1
gpgkey=https://download.docker.com/linux/fedora/gpg

[docker-ce-stable-source]
name=Docker CE Stable - Sources
baseurl=https://download.docker.com/linux/fedora/$releasever/source/stable
enabled=0
gpgcheck=1
gpgkey=https://download.docker.com/linux/fedora/gpg

[docker-ce-edge]
name=Docker CE Edge - $basearch
baseurl=https://download.docker.com/linux/fedora/$releasever/$basearch/edge
enabled=0
gpgcheck=1
gpgkey=https://download.docker.com/linux/fedora/gpg

[docker-ce-edge-debuginfo]
name=Docker CE Edge - Debuginfo $basearch
baseurl=https://download.docker.com/linux/fedora/$releasever/debug-$basearch/edge
enabled=0
gpgcheck=1
gpgkey=https://download.docker.com/linux/fedora/gpg

[docker-ce-edge-source]
name=Docker CE Edge - Sources
baseurl=https://download.docker.com/linux/fedora/$releasever/source/edge
enabled=0
gpgcheck=1
gpgkey=https://download.docker.com/linux/fedora/gpg

[docker-ce-test]
name=Docker CE Test - $basearch
baseurl=https://download.docker.com/linux/fedora/$releasever/$basearch/test
enabled=0
gpgcheck=1
gpgkey=https://download.docker.com/linux/fedora/gpg

[docker-ce-test-debuginfo]
name=Docker CE Test - Debuginfo $basearch
baseurl=https://download.docker.com/linux/fedora/$releasever/debug-$basearch/test
enabled=0
gpgcheck=1
gpgkey=https://download.docker.com/linux/fedora/gpg

[docker-ce-test-source]
name=Docker CE Test - Sources
baseurl=https://download.docker.com/linux/fedora/$releasever/source/test
enabled=0
gpgcheck=1
gpgkey=https://download.docker.com/linux/fedora/gpg

[docker-ce-nightly]
name=Docker CE Nightly - $basearch
baseurl=https://download.docker.com/linux/fedora/$releasever/$basearch/nightly
enabled=0
gpgcheck=1
gpgkey=https://download.docker.com/linux/fedora/gpg

[docker-ce-nightly-debuginfo]
name=Docker CE Nightly - Debuginfo $basearch
baseurl=https://download.docker.com/linux/fedora/$releasever/debug-$basearch/nightly
enabled=0
gpgcheck=1
gpgkey=https://download.docker.com/linux/fedora/gpg

[docker-ce-nightly-source]
name=Docker CE Nightly - Sources
baseurl=https://download.docker.com/linux/fedora/$releasever/source/nightly
enabled=0
gpgcheck=1
gpgkey=https://download.docker.com/linux/fedora/gpg
```

#### 2. 安装Docker-CE

```bash
[root@localhost ~]# dnf install docker-ce
上次元数据过期检查：0:07:02 前，执行于 2020年02月19日 星期三 04时22分53秒。
依赖关系解决。
======================================================================================
 Package              Arch      Version                     Repository           Size
======================================================================================
安装:
 docker-ce            x86_64    3:19.03.6-3.fc30            docker-ce-stable     24 M
安装依赖关系:
 containerd.io        x86_64    1.2.10-3.2.fc30             docker-ce-stable     23 M
 docker-ce-cli        x86_64    1:19.03.6-3.fc30            docker-ce-stable     39 M
 container-selinux    noarch    2:2.95-1.gite3ebc68.fc30    fedora               46 k
 libcgroup            x86_64    0.41-21.fc30                fedora               61 k

事务概要
======================================================================================
安装  5 软件包

总下载：87 M
安装大小：363 M
确定吗？[y/N]： y
下载软件包：
(1/5): containerd.io-1.2.10-3.2.fc30.x86_64.rpm       2.3 MB/s |  23 MB     00:10
(2/5): container-selinux-2.95-1.gite3ebc68.fc30.noarc 107 kB/s |  46 kB     00:00
(3/5): libcgroup-0.41-21.fc30.x86_64.rpm              592 kB/s |  61 kB     00:00
(4/5): docker-ce-cli-19.03.6-3.fc30.x86_64.rpm                    1.6 MB/s |  39 MB     00:24
(5/5): docker-ce-19.03.6-3.fc30.x86_64.rpm                        958 kB/s |  24 MB     00:25
--------------------------------------------------------------------------------------------------
总计                                                              3.2 MB/s |  87 MB     00:26
警告：/var/cache/dnf/docker-ce-stable-75f6791067d84a00/packages/containerd.io-1.2.10-3.2.fc30.x86_64.rpm: 头V4 RSA/SHA512 Signature, 密钥 ID 621e9f35: NOKEY
Docker CE Stable                                                  1.0 kB/s | 1.6 kB     00:01
导入 GPG 公钥 0x621E9F35:
 Userid: "Docker Release (CE rpm) <docker@docker.com>"
 指纹: 060A 61C5 1B55 8A7F 742B 77AA C52F EB6B 621E 9F35
 来自: https://download.docker.com/linux/fedora/gpg
确定吗？[y/N]： y
导入公钥成功
运行事务检查
事务检查成功。
运行事务测试
事务测试成功。
运行事务
  准备中  :                                                                                   1/1
  安装    : container-selinux-2:2.95-1.gite3ebc68.fc30.noarch                                 1/5
  运行脚本: container-selinux-2:2.95-1.gite3ebc68.fc30.noarch                                 1/5
  安装    : containerd.io-1.2.10-3.2.fc30.x86_64                                                                                                                                                  2/5
  运行脚本: containerd.io-1.2.10-3.2.fc30.x86_64                                                                                                                                                  2/5
  运行脚本: libcgroup-0.41-21.fc30.x86_64                                                                                                                                                         3/5
  安装    : libcgroup-0.41-21.fc30.x86_64                                                                                                                                                         3/5
  安装    : docker-ce-cli-1:19.03.6-3.fc30.x86_64                                                                                                                                                 4/5
  运行脚本: docker-ce-cli-1:19.03.6-3.fc30.x86_64                                                                                                                                                 4/5
  安装    : docker-ce-3:19.03.6-3.fc30.x86_64                                                                                                                                                     5/5
  运行脚本: docker-ce-3:19.03.6-3.fc30.x86_64                                                                                                                                                     5/5
  验证    : containerd.io-1.2.10-3.2.fc30.x86_64                                                                                                                                                  1/5
  验证    : docker-ce-3:19.03.6-3.fc30.x86_64                                                                                                                                                     2/5
  验证    : docker-ce-cli-1:19.03.6-3.fc30.x86_64                                                                                                                                                 3/5
  验证    : container-selinux-2:2.95-1.gite3ebc68.fc30.noarch                                                                                                                                     4/5
  验证    : libcgroup-0.41-21.fc30.x86_64                                                                                                                                                         5/5

已安装:
  docker-ce-3:19.03.6-3.fc30.x86_64  containerd.io-1.2.10-3.2.fc30.x86_64  docker-ce-cli-1:19.03.6-3.fc30.x86_64  container-selinux-2:2.95-1.gite3ebc68.fc30.noarch  libcgroup-0.41-21.fc30.x86_64

完毕！
```

Docker安装完成，可通过以下方式检测安装：

```bash
[root@localhost ~]# docker version
Client: Docker Engine - Community
 Version:           19.03.6
 API version:       1.40
 Go version:        go1.12.16
 Git commit:        369ce74
 Built:             Thu Feb 13 01:28:18 2020
 OS/Arch:           linux/amd64
 Experimental:      false

Server: Docker Engine - Community
 Engine:
  Version:          19.03.6
  API version:      1.40 (minimum version 1.12)
  Go version:       go1.12.16
  Git commit:       369ce74
  Built:            Thu Feb 13 01:26:53 2020
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          1.2.10
  GitCommit:        b34a5c8af56e510852c35414db4c1f4fa6172339
 runc:
  Version:          1.0.0-rc8+dev
  GitCommit:        3e425f80a8c931f88e6d94a8c831b9d5aa481657
 docker-init:
  Version:          0.18.0
  GitCommit:        fec3683
[root@localhost ~]#
```

#### 3. 启动Docker服务

启动Docker服务

```bash
sudo systemctl start docker
```

允许Dcoker服务在系统启动时运行

```bash
sudo systemctl enable docker
```

Docker只允许root和docker用户组中的用户才能使用。基于安全原因，通常不会使用root账户。在docker-ce安装完毕后，Linux系统中会自动创建好docker用户组，可将当前用户添加到docker用户组之中即可。

```bash
sudo usermod -aG docker $(whoami)
```

#### 4. 配置Docker国内镜像源

创建/etc/docker/daemon.json文件并写入源地址：

- Docker中国区官方镜像：https://registry.docker-cn.com
- 网易：http://hub-mirror.c.163.com
- ustc：https://docker.mirrors.ustc.edu.cn
- 中国科技大学：https://docker.mirrors.ustc.edu.cn
- 阿里云容器服务：https://cr.console.aliyun.com/

```bash
sudo tee /etc/docker/daemon.json  << EOF
{
  "registry-mirrors": ["https://registry.docker-cn.com"]
}
EOF
```

注意：必须确保daemon.json文件内容为json格式

### Podman的安装

```bash
[root@localhost ~]# dnf install podman
上次元数据过期检查：0:33:44 前，执行于 2020年02月19日 星期三 06时55分03秒。
依赖关系解决。
======================================================================================================================================================================================================
 Package                                                    Architecture                          Version                                                Repository                              Size
======================================================================================================================================================================================================
安装:
 podman                                                     x86_64                                2:1.7.0-3.fc30                                         updates                                 13 M
升级:
 libseccomp                                                 x86_64                                2.4.2-2.fc30                                           updates                                 65 k
安装依赖关系:
 conmon                                                     x86_64                                2:2.0.10-2.fc30                                        updates                                 36 k
 containernetworking-plugins                                x86_64                                0.8.5-1.fc30                                           updates                                 20 M
 containers-common                                          x86_64                                1:0.1.40-2.fc30                                        updates                                 46 k
 fuse3-libs                                                 x86_64                                3.6.2-1.fc30                                           updates                                 88 k
 fuse3                                                      x86_64                                3.4.2-3.fc30                                           fedora                                  45 k
安装弱的依赖:
 fuse-overlayfs                                             x86_64                                0.7.5-2.fc30                                           updates                                 62 k
 libvarlink-util                                            x86_64                                18-1.fc30                                              updates                                 47 k
 slirp4netns                                                x86_64                                0.4.0-4.git19d199a.fc30                                updates                                 85 k

事务概要
======================================================================================================================================================================================================
安装  9 软件包
升级  1 软件包

总下载：33 M
确定吗？[y/N]： y
下载软件包：
(1/10): containers-common-0.1.40-2.fc30.x86_64.rpm                                                                                                                     79 kB/s |  46 kB     00:00
(2/10): conmon-2.0.10-2.fc30.x86_64.rpm                                                                                                                                50 kB/s |  36 kB     00:00
(3/10): fuse-overlayfs-0.7.5-2.fc30.x86_64.rpm                                                                                                                         75 kB/s |  62 kB     00:00
(4/10): fuse3-libs-3.6.2-1.fc30.x86_64.rpm                                                                                                                             60 kB/s |  88 kB     00:01
(5/10): libvarlink-util-18-1.fc30.x86_64.rpm                                                                                                                           31 kB/s |  47 kB     00:01
(6/10): slirp4netns-0.4.0-4.git19d199a.fc30.x86_64.rpm                                                                                                                 60 kB/s |  85 kB     00:01
(7/10): fuse3-3.4.2-3.fc30.x86_64.rpm                                                                                                                                  26 kB/s |  45 kB     00:01
(8/10): libseccomp-2.4.2-2.fc30.x86_64.rpm                                                                                                                             56 kB/s |  65 kB     00:01
(9/10): podman-1.7.0-3.fc30.x86_64.rpm                                                                                                                                 52 kB/s |  13 MB     04:12
(10/10): containernetworking-plugins-0.8.5-1.fc30.x86_64.rpm                                                                                                           52 kB/s |  20 MB     06:31
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
总计                                                                                                                                                                   86 kB/s |  33 MB     06:34
运行事务检查
事务检查成功。
运行事务测试
事务测试成功。
运行事务
  准备中  :                                                                                                                                                                                       1/1
  安装    : fuse3-libs-3.6.2-1.fc30.x86_64                                                                                                                                                       1/11
  安装    : fuse3-3.4.2-3.fc30.x86_64                                                                                                                                                            2/11
  安装    : fuse-overlayfs-0.7.5-2.fc30.x86_64                                                                                                                                                   3/11
  运行脚本: fuse-overlayfs-0.7.5-2.fc30.x86_64                                                                                                                                                   3/11
  升级    : libseccomp-2.4.2-2.fc30.x86_64                                                                                                                                                       4/11
  安装    : slirp4netns-0.4.0-4.git19d199a.fc30.x86_64                                                                                                                                           5/11
  安装    : libvarlink-util-18-1.fc30.x86_64                                                                                                                                                     6/11
  安装    : containers-common-1:0.1.40-2.fc30.x86_64                                                                                                                                             7/11
  安装    : containernetworking-plugins-0.8.5-1.fc30.x86_64                                                                                                                                      8/11
  安装    : conmon-2:2.0.10-2.fc30.x86_64                                                                                                                                                        9/11
  安装    : podman-2:1.7.0-3.fc30.x86_64                                                                                                                                                        10/11
  清理    : libseccomp-2.4.0-0.fc30.x86_64                                                                                                                                                      11/11
  运行脚本: libseccomp-2.4.0-0.fc30.x86_64                                                                                                                                                      11/11
  验证    : conmon-2:2.0.10-2.fc30.x86_64                                                                                                                                                        1/11
  验证    : containernetworking-plugins-0.8.5-1.fc30.x86_64                                                                                                                                      2/11
  验证    : containers-common-1:0.1.40-2.fc30.x86_64                                                                                                                                             3/11
  验证    : fuse-overlayfs-0.7.5-2.fc30.x86_64                                                                                                                                                   4/11
  验证    : fuse3-libs-3.6.2-1.fc30.x86_64                                                                                                                                                       5/11
  验证    : libvarlink-util-18-1.fc30.x86_64                                                                                                                                                     6/11
  验证    : podman-2:1.7.0-3.fc30.x86_64                                                                                                                                                         7/11
  验证    : slirp4netns-0.4.0-4.git19d199a.fc30.x86_64                                                                                                                                           8/11
  验证    : fuse3-3.4.2-3.fc30.x86_64                                                                                                                                                            9/11
  验证    : libseccomp-2.4.2-2.fc30.x86_64                                                                                                                                                      10/11
  验证    : libseccomp-2.4.0-0.fc30.x86_64                                                                                                                                                      11/11

已升级:
  libseccomp-2.4.2-2.fc30.x86_64

已安装:
  podman-2:1.7.0-3.fc30.x86_64                    fuse-overlayfs-0.7.5-2.fc30.x86_64       libvarlink-util-18-1.fc30.x86_64 slirp4netns-0.4.0-4.git19d199a.fc30.x86_64 conmon-2:2.0.10-2.fc30.x86_64
  containernetworking-plugins-0.8.5-1.fc30.x86_64 containers-common-1:0.1.40-2.fc30.x86_64 fuse3-libs-3.6.2-1.fc30.x86_64   fuse3-3.4.2-3.fc30.x86_64

完毕！
```

验证：

通过podman version查看版本信息

```bash
[root@localhost ~]# podman version
Version:            1.7.0
RemoteAPI Version:  1
Go Version:         go1.12.14
OS/Arch:            linux/amd64
```

通过podman info查看相关环境信息

```bash
[root@localhost ~]# podman info
host:
  BuildahVersion: 1.12.0
  CgroupVersion: v1
  Conmon:
    package: conmon-2.0.10-2.fc30.x86_64
    path: /usr/bin/conmon
    version: 'conmon version 2.0.10, commit: 35d9b09d9d2791c7167091a9f25792535a380967'
  Distribution:
    distribution: fedora
    version: "30"
  MemFree: 308760576
  MemTotal: 4123508736
  OCIRuntime:
    name: runc
    package: containerd.io-1.2.10-3.2.fc30.x86_64
    path: /usr/bin/runc
    version: |-
      runc version 1.0.0-rc8+dev
      commit: 3e425f80a8c931f88e6d94a8c831b9d5aa481657
      spec: 1.0.1-dev
  SwapFree: 3047591936
  SwapTotal: 3057643520
  arch: amd64
  cpus: 1
  eventlogger: journald
  hostname: localhost
  kernel: 5.4.18-100.fc30.x86_64
  os: linux
  rootless: false
  uptime: 13h 36m 0.97s (Approximately 0.54 days)
registries:
  search:
  - docker.io
  - registry.fedoraproject.org
  - quay.io
  - registry.access.redhat.com
  - registry.centos.org
store:
  ConfigFile: /etc/containers/storage.conf
  ContainerStore:
    number: 0
  GraphDriverName: overlay
  GraphOptions:
    overlay.mountopt: nodev,metacopy=on
  GraphRoot: /var/lib/containers/storage
  GraphStatus:
    Backing Filesystem: extfs
    Native Overlay Diff: "false"
    Supports d_type: "true"
    Using metacopy: "true"
  ImageStore:
    number: 0
  RunRoot: /var/run/containers/storage
  VolumePath: /var/lib/containers/storage/volumes
```

## 参考文档

- [Docker官方安装文档 — Fedora](https://docs.docker.com/install/linux/docker-ce/fedora/)
- [Podman官网](https://podman.io/)
- [podman初试-和docker对比](https://blog.51cto.com/13447608/2448072?source=dra)
