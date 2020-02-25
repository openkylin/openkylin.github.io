---
layout: post
title:  "Docker Compose初探"
date:   2020-02-24 10:10:04 +0800
author: Kiki<huihui.fu@cs2c.com.cn>
tags: [docker]
---
## 简介

Docker Compose是docker提供的一个命令行工具，用于定义和运行多容器的应用程序。通过Compose，您可以使用YML文件来配置应用程序需要的所有服务，通过单一命令，就可以根据YML文件配置，创建并启动所有服务。

对于研发、测试、环境配置，还有CI工作流来说，Compose都是一个不错的工具，更多应用示例请参考: [Common use cases](https://github.com/docker/docker.github.io/blob/master/compose/index.md#common-use-cases)

Compose使用的三个步骤：

- 使用Dockerfile定义应用程序的环境。

- 使用docker-compose.yml定义构成应用程序的服务，这样它们可以在隔离环境中一起运行。

- 最后执行docker-compose up命令来启动并运行整个应用程序。

一个简单的docker-compose.yml配置案例如下：

```yaml
version: '2'

services:
  web:
    build: .
    ports:
     - "5000:5000"
    volumes:
     - .:/code
  redis:
    image: redis
```

详细的docker-compose.yml配置的编写规则请参考：[Compose file reference](https://github.com/docker/docker.github.io/blob/master/compose/compose-file/compose-versioning.md)

Compose通过以下一些命令来管理应用程序的整个生命周期：

- 启动、停止、重新构建服务

- 查看正在运行的服务状态

- 流式处理正在运行的服务的日志输出

- 对服务运行一次性命令

## 安装配置

安装Docker Compose前，请先确认以下依赖包是否安装： py-pip，python-dev，libffi-dev，openssl-dev，gcc，libc-dev，和 make。

在Linux操作系统下，从Github上下载它的二进制包来使用，目前最新版本的下载地址：<https://github.com/docker/compose/releases>

通过以下命令下载Docker Compose当前的稳定版本：

```shell
curl -L https://github.com/docker/compose/releases/download/1.25.4/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
```

若想安装其他版本，可直接替换以上命令中版本号1.25.4即可

将可执行权限应用于二进制文件：

```shell
chmod +x /usr/local/bin/docker-compose
```

创建软连接：

```shell
ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
```

测试是否安装成功：

```shell
[root@localhost ~]# docker-compose --version
docker-compose version 1.25.4, build 8d51620a
```

**首次执行docker-compose --version遇到如下错误：**

```shell
[root@localhost ~]# docker-compose --version
[2513] Error loading Python lib '/tmp/_MEIc8S5bO/libpython3.7m.so.1.0': dlopen: libcrypt.so.1: cannot open shared object file: No such file or directory
```

原因：libcrypt.so.1文件或目录不存在，未安装相关的软件包。

解决办法：通过dnf whatprovides查找相关文件所属的软件包，安装即可

```shell
[root@localhost ~]# dnf whatprovides '*/libcrypt.so.1'
上次元数据过期检查：0:28:41 前，执行于 2020年02月25日 星期二 09时34分16秒。
libxcrypt-compat-4.4.12-1.fc30.i686 : Compatibility library providing legacy API functions
仓库        ：updates
匹配来源：
文件名    ：/lib/libcrypt.so.1

libxcrypt-compat-4.4.12-1.fc30.x86_64 : Compatibility library providing legacy API functions
仓库        ：updates
匹配来源：
文件名    ：/lib64/libcrypt.so.1

glibc-arm-linux-gnu-2.27-4.fc29.x86_64 : Cross Compiled GNU C Library targeted at arm-linux-gnu
仓库        ：fedora
匹配来源：
文件名    ：/usr/arm-linux-gnu/sys-root/lib/libcrypt.so.1

libxcrypt-compat-4.4.4-2.fc30.i686 : Compatibility library providing legacy API functions
仓库        ：fedora
匹配来源：
文件名    ：/lib/libcrypt.so.1

libxcrypt-compat-4.4.4-2.fc30.x86_64 : Compatibility library providing legacy API functions
仓库        ：fedora
匹配来源：
文件名    ：/lib64/libcrypt.so.1
[root@localhost ~]# dnf install libxcrypt-compat
上次元数据过期检查：0:29:23 前，执行于 2020年02月25日 星期二 09时34分16秒。
依赖关系解决。
===========================================================================================================================================
 Package                               Architecture                Version                              Repository                    Size
===========================================================================================================================================
安装:
 libxcrypt-compat                      x86_64                      4.4.12-1.fc30                        updates                       95 k
升级:
 libxcrypt                             x86_64                      4.4.12-1.fc30                        updates                      122 k
 libxcrypt-devel                       x86_64                      4.4.12-1.fc30                        updates                       35 k

事务概要
===========================================================================================================================================
安装  1 软件包
升级  2 软件包

总下载：252 k
确定吗？[y/N]： y
下载软件包：
(1/3): libxcrypt-devel-4.4.12-1.fc30.x86_64.rpm                                                            253 kB/s |  35 kB     00:00
(2/3): libxcrypt-compat-4.4.12-1.fc30.x86_64.rpm                                                           455 kB/s |  95 kB     00:00
(3/3): libxcrypt-4.4.12-1.fc30.x86_64.rpm                                                                  433 kB/s | 122 kB     00:00
-------------------------------------------------------------------------------------------------------------------------------------------
总计                                                                                                        84 kB/s | 252 kB     00:02
运行事务检查
事务检查成功。
运行事务测试
事务测试成功。
运行事务
  准备中  :                                                                                                                            1/1
  升级    : libxcrypt-4.4.12-1.fc30.x86_64                                                                                             1/5
  安装    : libxcrypt-compat-4.4.12-1.fc30.x86_64                                                                                      2/5
  升级    : libxcrypt-devel-4.4.12-1.fc30.x86_64                                                                                       3/5
  清理    : libxcrypt-devel-4.4.4-2.fc30.x86_64                                                                                        4/5
  清理    : libxcrypt-4.4.4-2.fc30.x86_64                                                                                              5/5
  运行脚本: libxcrypt-4.4.4-2.fc30.x86_64                                                                                              5/5
  验证    : libxcrypt-compat-4.4.12-1.fc30.x86_64                                                                                      1/5
  验证    : libxcrypt-4.4.12-1.fc30.x86_64                                                                                             2/5
  验证    : libxcrypt-4.4.4-2.fc30.x86_64                                                                                              3/5
  验证    : libxcrypt-devel-4.4.12-1.fc30.x86_64                                                                                       4/5
  验证    : libxcrypt-devel-4.4.4-2.fc30.x86_64                                                                                        5/5

已升级:
  libxcrypt-4.4.12-1.fc30.x86_64                                    libxcrypt-devel-4.4.12-1.fc30.x86_64

已安装:
  libxcrypt-compat-4.4.12-1.fc30.x86_64

完毕！
```

## 应用实例

使用Docker compose前，请确保Docker以及Docker Compose已经正确安装。

1. Step 1: 准备应用程序
    - 创建项目目录

    ```shell
    mkdir composetest
    cd composetest
    ```

    - 在测试目录中创建一个名为 app.py 的文件，并复制粘贴以下内容：

    ```python
    import time

    import redis
    from flask import Flask

    app = Flask(__name__)
    cache = redis.Redis(host='redis', port=6379)


    def get_hit_count():
        retries = 5
        while True:
            try:
                return cache.incr('hits')
            except redis.exceptions.ConnectionError as exc:
                if retries == 0:
                    raise exc
                retries -= 1
                time.sleep(0.5)


    @app.route('/')
    def hello():
        count = get_hit_count()
        return 'Hello World! I have been seen {} times.\n'.format(count)

    ```

    在此示例中，redis 是应用程序网络上的 redis 容器的主机名，该主机使用的端口为 6379

    在 composetest 目录中创建另一个名为 requirements.txt 的文件，内容如下：

    ```txt
    flask
    redis
    ```

2. Step 2: 创建Dockerfile
    Dockerfile文件用于构建Docker镜像，这个镜像中包含应用程序需要的Python环境以及相关的依赖。
    创建Dockerfile文件，内容如下：

    ```dockerfile
    FROM python:3.7-alpine
    WORKDIR /code
    ENV FLASK_APP app.py
    ENV FLASK_RUN_HOST 0.0.0.0
    RUN apk add --no-cache gcc musl-dev linux-headers
    COPY requirements.txt requirements.txt
    RUN pip install -r requirements.txt
    COPY . .
    CMD ["flask", "run"]
    ```

    Dockerfile文件注释如下：
    - 使用Python 3.7镜像作为应用程序的基础镜像
    - 设置工作目录：/code
    - 设置flask命令使用的环境变量
    - 安装gcc用于编译
    - 拷贝requirements.txt文件
    - 通过requirements.txt文件安装python相关的依赖项
    - 拷贝项目当前目录到镜像的工作目录
    - 容器提供默认的执行命令为：flask run

    Dockerfile更多相关内容，请移步：
    - [Docker user guide](https://github.com/docker/docker.github.io/blob/master/engine/tutorials/dockerimages.md#building-an-image-from-a-dockerfile)
    - [Dockerfile reference](https://github.com/docker/docker.github.io/blob/master/engine/reference/builder.md)

3. Step 3: 通过Compose配置文件定义应用程序的服务
    在项目目录下，创建docker-compose.yml，其内容如下：

    ```yaml
    version: '3'
    services:
      web:
        build: .
        ports:
          - "5000:5000"
      redis:
        image: "redis:alpine"
    ```

    该Compose文件定义了两个服务：web 和 redis。
    - Web service：该web服务使用通过Dockerfile构建的镜像。它将容器和主机绑定到暴露的端口5000。此示例服务使用Flask Web服务器的默认端口5000
    - Redis service：该redis服务使用Docker Hub中的公共Redis映像

4. Step 4: 使用Compose命令构建和运行应用程序
    通过docker-compose up命令启动应用程序：

    ```shell
    [root@localhost composetest]# docker-compose up
    Creating network "composetest_default" with the default driver
    Building web
    Step 1/9 : FROM python:3.7-alpine
    3.7-alpine: Pulling from library/python
    c9b1b535fdd9: Pull complete
    2cc5ad85d9ab: Pull complete
    53a2fca3c2ea: Pull complete
    30fce49de8b1: Pull complete
    ca406aaf66e0: Pull complete
    Digest: sha256:4e91ff3eed8bdd013178b6d8e55f656b394aeea42945f04389e4af5462e7cb6d
    Status: Downloaded newer image for python:3.7-alpine
    ---> a5d195bb2a63
    Step 2/9 : WORKDIR /code
    ---> Running in 383b9c047bc0
    Removing intermediate container 383b9c047bc0
    ---> b1d1aa7fb542
    Step 3/9 : ENV FLASK_APP app.py
    ---> Running in 7c15e2ac5605
    Removing intermediate container 7c15e2ac5605
    ---> eff98003a302
    Step 4/9 : ENV FLASK_RUN_HOST 0.0.0.0
    ---> Running in 5a27ed3f6c64
    Removing intermediate container 5a27ed3f6c64
    ---> be6f583a0052
    Step 5/9 : RUN apk add --no-cache gcc musl-dev linux-headers
    ---> Running in 7cbe2a06a6ff
    fetch http://dl-cdn.alpinelinux.org/alpine/v3.11/main/x86_64/APKINDEX.tar.gz
    fetch http://dl-cdn.alpinelinux.org/alpine/v3.11/community/x86_64/APKINDEX.tar.gz
    (1/12) Installing libgcc (9.2.0-r3)
    (2/12) Installing libstdc++ (9.2.0-r3)
    (3/12) Installing binutils (2.33.1-r0)
    (4/12) Installing gmp (6.1.2-r1)
    (5/12) Installing isl (0.18-r0)
    (6/12) Installing libgomp (9.2.0-r3)
    (7/12) Installing libatomic (9.2.0-r3)
    (8/12) Installing mpfr4 (4.0.2-r1)
    (9/12) Installing mpc1 (1.1.0-r1)
    (10/12) Installing gcc (9.2.0-r3)
    (11/12) Installing linux-headers (4.19.36-r0)
    (12/12) Installing musl-dev (1.1.24-r0)
    Executing busybox-1.31.1-r9.trigger
    OK: 131 MiB in 47 packages
    Removing intermediate container 7cbe2a06a6ff
    ---> 8954c4530c9e
    Step 6/9 : COPY requirements.txt requirements.txt
    ---> 8ca0e7e75f7d
    Step 7/9 : RUN pip install -r requirements.txt
    ---> Running in a13015b04175
    Collecting flask
      Downloading Flask-1.1.1-py2.py3-none-any.whl (94 kB)
    Collecting redis
      Downloading redis-3.4.1-py2.py3-none-any.whl (71 kB)
    Collecting click>=5.1
      Downloading Click-7.0-py2.py3-none-any.whl (81 kB)
    Collecting Werkzeug>=0.15
      Downloading Werkzeug-1.0.0-py2.py3-none-any.whl (298 kB)
    Collecting Jinja2>=2.10.1
      Downloading Jinja2-2.11.1-py2.py3-none-any.whl (126 kB)
    Collecting itsdangerous>=0.24
      Downloading itsdangerous-1.1.0-py2.py3-none-any.whl (16 kB)
    Collecting MarkupSafe>=0.23
      Downloading MarkupSafe-1.1.1.tar.gz (19 kB)
    Building wheels for collected packages: MarkupSafe
      Building wheel for MarkupSafe (setup.py): started
      Building wheel for MarkupSafe (setup.py): finished with status 'done'
      Created wheel for MarkupSafe: filename=MarkupSafe-1.1.1-cp37-cp37m-linux_x86_64.whl size=32609 sha256=a4fabd314ab107e7c0d3fde9b41c3dc8a4ea39f9a2f31261e04a67f7815c0357
      Stored in directory: /root/.cache/pip/wheels/b9/d9/ae/63bf9056b0a22b13ade9f6b9e08187c1bb71c47ef21a8c9924
    Successfully built MarkupSafe
    Installing collected packages: click, Werkzeug, MarkupSafe, Jinja2, itsdangerous, flask, redis
    Successfully installed Jinja2-2.11.1 MarkupSafe-1.1.1 Werkzeug-1.0.0 click-7.0 flask-1.1.1 itsdangerous-1.1.0 redis-3.4.1
    Removing intermediate container a13015b04175
    ---> 1f7ad3ce85c3
    Step 8/9 : COPY . .
    ---> 93aa9e4318b7
    Step 9/9 : CMD ["flask", "run"]
    ---> Running in 13ba37d94b83
    Removing intermediate container 13ba37d94b83
    ---> ea3066cef40e
    Successfully built ea3066cef40e
    Successfully tagged composetest_web:latest
    WARNING: Image for service web was built because it did not already exist. To rebuild this image you must use 'docker-compose build' or 'docker-compose up --build'.
    Pulling redis (redis:alpine)...
    alpine: Pulling from library/redis
    c9b1b535fdd9: Already exists
    8dd5e7a0ba4a: Pull complete
    e20c1cdf5aef: Pull complete
    f06a0c1e566e: Pull complete
    230b5c8df708: Pull complete
    0cb9ac88f5bf: Pull complete
    Digest: sha256:cb9783b1c39bb34f8d6572406139ab325c4fac0b28aaa25d5350495637bb2f76
    Status: Downloaded newer image for redis:alpine
    Creating composetest_web_1   ... done
    Creating composetest_redis_1 ... done
    Attaching to composetest_redis_1, composetest_web_1
    redis_1  | 1:C 25 Feb 2020 03:34:13.765 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
    redis_1  | 1:C 25 Feb 2020 03:34:13.765 # Redis version=5.0.7, bits=64, commit=00000000, modified=0, pid=1, just started
    redis_1  | 1:C 25 Feb 2020 03:34:13.765 # Warning: no config file specified, using the default config. In order to specify a config file use redis-server /path/to/redis.conf
    redis_1  | 1:M 25 Feb 2020 03:34:13.776 * Running mode=standalone, port=6379.
    redis_1  | 1:M 25 Feb 2020 03:34:13.776 # Server initialized
    redis_1  | 1:M 25 Feb 2020 03:34:13.776 # WARNING overcommit_memory is set to 0! Background save may fail under low memory condition. To fix this issue add 'vm.overcommit_memory = 1' to /etc/sysctl.conf and then reboot or run the command 'sysctl vm.overcommit_memory=1' for this to take effect.
    redis_1  | 1:M 25 Feb 2020 03:34:13.776 # WARNING you have Transparent Huge Pages (THP) support enabled in your kernel. This will create latency and memory usage issues with Redis. To fix this issue run the command 'echo never > /sys/kernel/mm/transparent_hugepage/enabled' as root, and add it to your /etc/rc.local in order to retain the setting after a reboot. Redis must be restarted after THP is disabled.
    redis_1  | 1:M 25 Feb 2020 03:34:13.778 * Ready to accept connections
    web_1    |  * Serving Flask app "app.py"
    web_1    |  * Environment: production
    web_1    |    WARNING: This is a development server. Do not use it in a production deployment.
    web_1    |    Use a production WSGI server instead.
    web_1    |  * Debug mode: off
    web_1    |  * Running on http://0.0.0.0:5000/ (Press CTRL+C to quit)
    ```

    在本例中，Compose从公共Docker仓库中拉取Redis镜像，在构建镜像时，直接将程序代码静态的拷贝到镜像中，并启动了定义的应用程序服务。

    在浏览器中输入：http://localhost:5000/ 即可看到运行应用程序：
    ![app运行](/assets/img/docker/docker-compose-test.png "docker-compose-test.png")

    刷新页面，页面内容如下：

    ```txt
    Hello World! I have been seen 2 times.
    ```

    如果你想在后台执行该服务可以加上 -d 参数：

    ```shell
    docker-compose up -d
    ```

    切换终端，通过docker image可查看本地的镜像：

    ```shell
    [root@localhost ~]# docker image list
    REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
    composetest_web     latest              ea3066cef40e        43 minutes ago      222MB
    python              3.7-alpine          a5d195bb2a63        2 weeks ago         97.8MB
    redis               alpine              b68707e68547        5 weeks ago         29.8MB
    ```

    停止应用程序，有两种方式：
    - 切换终端，通过 docker-compose down 命令停止服务
    - 在服务运行终端，通过 CTRL+C 停止服务服务

## 参考文档

- [Docker Compose源码](https://github.com/docker/compose)

- [Docker官网文档-docker compose简介](https://docs.docker.com/compose/reference/overview/)

- [Docker官方文档-docker compose使用手册](https://github.com/docker/docker.github.io/blob/master/compose/gettingstarted.md)

- [菜鸟笔记-Docker Compose](https://www.runoob.com/docker/docker-compose.html)
