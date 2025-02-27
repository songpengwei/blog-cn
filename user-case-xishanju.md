---
title: TiDB 在西山居实时舆情监控系统中的应用
author: ['邹学']
date: 2018-06-09
summary: 舆情系统目前总数据量数 T，已正式上线三个月，期间从未出现过异常，系统平稳、使用效果也非常好。可以说 TiDB 给我们的体验远超预期。
tags: ['游戏']
category: case
url: /case/user-case-xishanju/
weight: 1
logo: /images/blog-cn/customers/xishanju-logo.png
customer: 西山居
customerCategory: 游戏
---


> **作者介绍**：邹学，舆情监控系统技术负责人，珠海金山网络游戏科技有限公司（西山居）数据中心架构师，2015 年加入西山居，具有 10 年游戏行业软件开发经验，主要参与了公司的游戏网关设计，数据分析框架底层架构建设等，现专注于实时计算、爬虫、分布式系统方向。

## 公司简介

西山居创建 1995 年初夏，在美丽的海滨小城珠海，西山居工作室孕育而生，一群西山居居士们十年如一日尅勊业业的奋斗。"创造快乐，传递快乐！" 一直是西山居居士们的创作宗旨。西山居以领先的技术作为坚实的基础以独特的本土化产品为玩家提供时尚化服务。在未来，西山居仍以娱乐软件为主导产品，不断进行研发和市场活动，逐步发展成为国内最优秀的集制作、发行于一体的数字化互动娱乐公司。

## 业务背景

由于公司产品的社交属性都非常强，对相关舆情进行分析与了解就显得很有必要，在此背景下，舆情监控系统应运而生。该系统利用算法组提供的分词算法，对文本进行解析与分类，打上各类标记后再通过计算产生中间结果。舆情系统直接查询这些中间结果，产生各类报表与趋势图，为及时掌握各类舆情趋势提供便利。用户可以自由组合舆情关注点，从而对平台有很严格的实时交互性查询要求，是典型的实时 HTAP 类业务。

## 存储技术选型

舆情系统之前我们曾经实现过一个客服系统，这个系统要求能实时查询，但面对是海量的玩家行为记录。在当时情况下（2016 年），可以选择的对象只有 MyCAT 这类数据库中间件，通过综合压力测试后，我们选定了 KingShard 这一款由公司前同事开发的中间件，KingShard 虽然没有 MyCAT 丰富的周边功能，但它在满足我们业务需求的核心功能上有更快的表现。但正因为有了这一次中间件的使用，我们对中间件有了比较全面的了解，它们在查询优化上有着天生的弱点，无法满足更复杂的查询或者表现极不友好，为此我们还不得不砍掉了客服系统的部分业务功能，所以在那时我已开始寻找更优的技术方案，其中分布式数据库是我们考察的重点方向。

BigTable、GFS、MapReduce 是谷歌在分布式存储与查询领域的探索成果，他们没有公开具体实现代码，但却发布了相应论文，对分布式文件系统、大数据挖掘和 NoSQL 发展起了重大促进作用。开源界根据这一成果开发出对应产品是 HBase、HDFS、Hadoop，这三个产品红极一时，相关周边产品更是百花齐放，很多细分领域都同时出现了多个产品竞争，让整个生态非常繁荣但也变得复杂，提高了我们的学习与使用成本。那么，在一些领域中有没有更加简单、直接、具有较强融合能力的解决方案呢？此时距谷歌这三篇论文发表已近 10 年，谷歌内部早已在尝试融合 NoSQL 和 SQL，并对它们进行了多次更新换代，Spanner、F1 两篇论文便是谷歌在这一方向的探索成果。开源分布式数据库 TiDB 便是受论文启发而设计的 HTAP (Hybrid Transactional and Analytical Processing) 数据库，结合了传统的 RDBMS 和 NoSQL 的最佳特性，兼容 MySQL，具有支持分布式事务、无限的水平扩展、数据强一致性保证等核心 NewSQL 特性。

当时，舆情系统接入的第一个游戏平均每天入库数据量就已达到 8500 万条，并且还需要支持各种实时交互性查询，显然中间件已不能满足要求，传统的关系型数据库则更加不可能了。考虑到以后还会有其它游戏接入，我们果断选择了分布式数据库。
随着互联网经济的发展，数据量跟并发数也在飞速增长，单机数据库已越来越不能满足要求了，为此谷歌、阿里等大厂都有了自研的分布式数据库，但都没有开源，而 MySQL 的 MGR 及相关功能进展的步子太小，TiDB 的出现很好的弥补了市场空白，成为我们的唯一选择。

## 服务器配置

舆情系统是内部孵化项目，服务器具体如下：

新购物理机器 6 台：

![](media/user-case-xishanju/1.png)

旧物理机 4 台：

![](media/user-case-xishanju/2.png)

我们将对资源使用相对较小的 PD、监控服务分别放在旧物理机上，TiDB、TiKV 和 TiSpark 则分配在新机器上，详细如下：

![](media/user-case-xishanju/3.png)

其中每个 TiKV 分配 CPU 10C / 内存 64G / 硬盘 2T，每个 TiSpark 分配 CPU 20C / 内存 64G。在资源有限情况下，结合数据量及舆情系统的 AP 业务属性，我们设计了这样相对复杂的架构，目的是为了充分利用好服务器资源，让它们能承担更极限的压力，事后有多次历史数据的导入也证明了我们这样设计的必要性，多谢 TiDB 的兄弟全程耐心指导及帮助。

## 项目开发过程

得出中间结果是一个非常大的计算过程，所以我们使用了 TiSpark。TiSpark 是 PingCAP 为解决用户复杂 OLAP 需求而推出的产品，它是将 Spark SQL 直接运行在分布式存储引擎 TiKV 上的 OLAP 解决方案。有了 TiSpark 我们可以方便地使用 Spark，而不需要再单独搭建一套 Spark 集群。

从 TiDB 的 1.0 RC 3 版本开始，我们就在金山云上搭建集群来试用与压测。期间经历了多次版本热更，集群也一直很稳定，功能与性能越来越强，所以在舆情系统开始开发时我们果断使用了 TiDB。并且 TiDB 有强烈的市场需求，他们的版本更新非常迅速，在试用期间时发现了一些功能不能满足需要，往往在下一个版本就解决了，这让人非常惊叹。

当前版本未加入实时计算业务，再加上使用了 TiSpark，所以整个架构相对简单，详细如下图：

![](media/user-case-xishanju/4.png)

## 项目上线及使用情况

舆情系统目前总数据量数 T，已正式上线三个月，期间从未出现过异常，系统平稳、使用效果也非常好。现在每天原始文本数据在 2500 万条以上，通过算法分词后产生的中间结果则每天有 6000 万条左右（日均入库 8500 万条），高峰时段的平均 QPS 在 3K 以上，闲时的平均 QPS 为 300 多一点。根据这样的量级，在一开始评估时设定的目标是：支持最近一个星期的实时交互性查询，但现在已经远远超过我们的预期。目前所有一个月内的时间跨度查询都在 1 秒左右完成，个别复杂的 3 个月的实时交互性查询则需要 2 秒多一点。

可以说 TiDB 给我们的体验远超预期，这样的数据量级及响应，单机版数据库是不可能达到要求的。
