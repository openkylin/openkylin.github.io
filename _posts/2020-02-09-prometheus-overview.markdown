---
layout: post
title:  Prometheus简介和框架
author: Jin <jinyi.li@cs2c.com.cn>
tags: [Prometheus]
---

本文翻译自Prometheus官网，介绍了Prometheus系统构架和基础搭建。
## 简介
Prometheus是一套开源的系统监控警报工具包，初始建立在Soundclouds，作为紧随Kubernetes之后第二个托管项目，于2016年正式加入Cloud Native Computing Foundation。
### 特性
*	一个多维度的数据模型，时间序列通过Metric名和键值对来区分。
*	灵活的查询语句PromQL：可以通过多个Metrics进行维度性的操作如乘法，加法，连接等
*	不依赖于分布式储存：Prometheus server是一个单服务器节点
*	使用pull模式采集时间序列数据
*	通过中间网关（intermediary gateway）支持推送时间序列
*	通过服务发现或者静态配置去获取监控的 targets。
*	有多种可视化图形界面

### 组件及构成
![architecture](https://logz.io/wp-content/uploads/2018/07/Prometheus.png)

 - **Prometheus Server**: 用于收集和存储时间序列数据。
 - **Client Library**：客户端库，检测应用代码。
 - **Push Gateway**: 支持短期的 jobs。
 - **Exporters**: 用于暴露已有的第三方服务的 metrics 给 Prometheus，例如 HAProxy, StatsD, Graphite等。
 - **Alertmanager**:处理从Prometheus server接受到的Alert，进行处理后发出警报。
 - 及其他可选项

### 大致工作流程

 1. 由Prometheuss server从被配置好的jobs中 拉取拉取Metrics。
 2. Prometheus server 在本地存储收集到的 metrics，并运行已定义好的 alert.rules以便聚合或记录新的实践序列或者推送警报。
 3. Grafana 或其他 API 使用者可用于可视化收集的数据。


## 初始化配置 ##
### 下载
从平台下载最新发布的Prometheus，然后解压缩

    tar xvfz prometheus-*.tar.gz
    cd prometheus-*
Prometheus server是一个single binary文件（Windows中是prometheus.exe），可以使用--help flag去运行和查找帮助

    ./prometheus --help
    usage: prometheus [<flags>]
    The Prometheus monitoring server
    . . .
在启动Prometheus之前需要先完成配置
### 配置
Prometheus使用YAML配置，初始化下载中包含一个叫做prometheus.yml的配置文件，可以从这里开始。

    global:
      scrape_interval:     15s
      evaluation_interval: 15s
    
    rule_files:
      # - "first.rules"
      # - "second.rules"
    
    scrape_configs:
      - job_name: prometheus
        static_configs:
          - targets: ['localhost:9090']

示例中代码主要分成3部分：global, rule_files, 和scrape_configd。

*global* 用来管理Prometheus server的通用配置。有两种呈现，分别是记录scrape频率的*scrape_interval*，例子里是15s。第二个是评估Rules的频率，Prometheus使用rules创建新的时间序列或生成警报。

*rule_files*指定用户希望Prometheus server加载的Rules的位置，目前示例中为空。

*scrape_configs* 管理Prometheus 监控的资源。在默认配置中，job名为 Prometheus，它scrape（拉取） Prometheus server公开的时间序列数据。这个Job包含一个单个静态的target，Localhost在port 9090。
metrics一般存在路径/metrict，所以默认配置URL地址为：
http://localhost:9090/metrics

更多配置信息可查看这个网页：
https://prometheus.io/docs/prometheus/latest/configuration/configuration/
###启动Prometheus
启动修改过配置文件的通过更改config.file地址

    ./prometheus --config.file=prometheus.yml
启动成功后就可以通过打开http://localhost:9090查看相关状态页了，可能需要30s的时间去收集信息。
打开http://localhost:9090/metrics可以浏览metrics终端。

###Ref

 1. [Prometheus Overview](https://prometheus.io/docs/introduction/overview/)
 2. [ibm.com developerworks](https://www.ibm.com/developerworks/cn/cloud/library/cl-lo-prometheus-getting-started-and-practice/index.html)
 3. [Prometheus 安装与配置](https://www.cnblogs.com/vovlie/p/Prometheus_install.html)
 4. [Prometheus 系统监控方案](https://www.cnblogs.com/vovlie/p/Prometheus_CONCEPTS.html)
