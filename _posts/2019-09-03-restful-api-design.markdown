---
layout: post
title: 关于 REST API URI 设计规则的讨论
author: Joey <majunjiev@gmail.com>
tags: [Restful]
---

## 写在前面

本文是阅读[7 Rules for REST API URI Design](http://blog.restcase.com/7-rules-for-rest-api-uri-design/)后，结合自己的经验，就《如何设计优秀的 REST API URI》做的个人讨论。

以下 “原文” 表示 《7 Rules for REST API URI Design》。

## Q1：反斜杠（ / ）要不要加在 URI 的最后？

原文提到，一个 URI，带和不带 / 就意味着是两个不同的 URI 了；而 REST 的核心就是一个资源**有且只有一个** URI 来表达，所以一个资源应该只有一个对应的 URI。

那么这个 / 是要还是不要，原文建议：不要！

> 对于 REST API 而言，我个人也觉得不要 / ，原因有两点：
> 1.  / 在 URI 中往往表达了 层级 关系，所以 / 后面应该要有更深一层的资源描述符，而不是以 / 结束；
> 2. 一个 URI 唯一定位（标识）一个资源，所以 URI 中应该只包含资源的类型、名称、ID 等可以描述资源的属性，加在最后的 / 并不是资源的属性，所以加上 / 容易混淆且无意义；

## Q2：URI 使用的反斜杠（ / ）只能是用来表达资源层级管理的？

这个是大多数 REST API 设计者的共识，比如要获取某数据中心内某集群的某主机的某虚拟机的详细信息，直接用：`GET /api/datacenter/<datacenterid>/cluster/<clusterid>/host/<hostid>/vm/<vmid>`就可以了， / 表达出了“数据中心->集群->主机->虚拟机”的四个层级。

> 但我在项目就遇到了一些情况，比如虚拟机的操作接口，支持开机、关机、重启等，项目初期我设计时用的： `POST /api/datacenter/<datacenterid>/cluster/<clusterid>/host/<hostid>/vm/<vmid>?action=poweron/poweroff/reboot`。但实际上，REST API 设计时并不推荐使用 query param 的形式来表达资源。所以，可以从另外一个角度，将操作也看成一种资源，那么接口就变成了：`POST /api/datacenter/<datacenterid>/cluster/<clusterid>/host/<hostid>/vm/<vmid>/action/<actionname>`（其中 actionname 的值是poweron/poweroff/reboot）。

> 或许你要说，改成 POST /action/reboot，岂不是变成了“创建了一个某虚拟机的重启操作”？！的确是，其实也可以换个角度：“虚拟机在后台会**自动执行**为其**新创建的操作**”，这样想的话，“重启虚拟机”就变成了“为虚拟机创建重启操作”了，也就完全符合了 REST API 中的资源思想了。

## Q3：在 URI 中，短中线 - 应该替代空格来增强阅读性？

在平时 URI 设计中，我很少用到短中线。其实短中线也是随着 wordpress 兴起来的，主要用来表示文章的 title，用 - 替代了空格，比如：http://www.example.com/blogs/guy-levin/posts/this-is-my-first-post。

> 原文说短中线肯定可以增强阅读性，这个我是非常赞同的。不过，我认为在 REST API URI 中，短中线还是能不用就不用吧。

> 因为，短中线本意是用来区分开长句子，或者作为UUID 的一部分，而且只会在资源唯一标识中使用，滥用会导致 URI 过分冗长。但反过来，资源唯一标识中有没有短中线，是由资源唯一标识决定的，假如标识有没有短中线，那没必要为了增强阅读星而加上短中线。

## Q4：在 URI 中，应该不用下划线 _ ？

> 我非常同意原文的说法，尽量不用下划线 -。

## Q5：在 URI 中，尽量使用小写字母？

> 我非常同意原文的说法，尽量使用小写字母。

## Q6：在 URI 中，应不应该包含文件的后缀？

原文提到应结合 Content-Type 来实现文件后缀的标识；这个问题其实我之前并没有想过，所以下载文件的 URL 都带了后缀，现在想想，URI 中还带着一个 dot，也实在是很丑。

> 这个建议，我欣然接受，以后会按照此规则执行！

## Q7：endpoint 名称应该用单数还是复数？

这个问题我之前就纠结过，我的原则是：获取列表时用复数，获取单个时用单数。虽然之前纠结过，但没碰到一个能说服我的资料。我设计的虚拟机 URI 如下：

* `GET /api/vms`：获取虚拟机列表
* `GET /api/vm/<vmid>`：获取虚拟机详细信息
* `POST /api/vms`：创建虚拟机

原文建议统一使用复数，比如获取虚拟机详细信息接口使用 URI：`GET /api/vms/<vmid>`。原文给出的理由这次有些让我动摇了，它提到，即使获取单个资源也用复数有些违背常规，但基于“简单”原则， URI 需要统一词语。（vm 和 vms 相差不大，但 person 和 people 就相差很大了）

> 基于 person/people 的这个理由，我服了。此建议，我欣然接受，以后会按照此规则执行。


## 写在最后

REST API URI 的设计，一般也只是规范、建议，并没有强制性；我觉得在一定情况下，所有的这些规范都是可以妥协违背的；但要做好你自己内心受煎熬的准备！ ^_^