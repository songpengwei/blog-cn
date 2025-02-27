---
title: TiDB 2.0 GA Release
author: ['PingCAP']
date: 2018-04-27
summary: 2018 年 4 月 27 日，TiDB 发布 2.0 GA 版。相比 1.0 版本，对 MySQL 兼容性、系统稳定性、优化器和执行器做了很多改进。
tags: ['TiDB']
---

2018 年 4 月 27 日，TiDB 发布 2.0 GA 版。相比 1.0 版本，对 MySQL 兼容性、系统稳定性、优化器和执行器做了很多改进。

## TiDB

* SQL 优化器

	* 精简统计信息数据结构，减小内存占用

	* 加快进程启动时加载统计信息速度

	* 支持统计信息动态更新 [experimental]

	* 优化代价模型，对代价估算更精准

	* 使用 `Count-Min Sketch` 更精确地估算点查的代价

	* 支持分析更复杂的条件，尽可能充分的使用索引

	* 支持通过 `STRAIGHT_JOIN` 语法手动指定 Join 顺序

	* `GROUP BY`子句为空时使用 Stream Aggregation 算子，提升性能

	* 支持使用索引计算 `Max/Min` 函数

	* 优化关联子查询处理算法，支持将更多类型的关联子查询解关联并转化成 `Left Outer Join`

	* 扩大 `IndexLookupJoin` 的使用范围，索引前缀匹配的场景也可以使用该算法

* SQL 执行引擎

	* 使用 Chunk 结构重构所有执行器算子，提升分析型语句执行性能，减少内存占用，显著提升 TPC-H 结果

	* 支持 Streaming Aggregation 算子下推

	* 优化 `Insert Into Ignore` 语句性能，提升 10 倍以上

	* 优化 `Insert On Duplicate Key Update` 语句性能，提升 10 倍以上

	* 下推更多的数据类型和函数到 TiKV 计算

	* 优化 `Load Data` 性能，提升 10 倍以上

	* 支持对物理算子内存使用进行统计，通过配置文件以及系统变量指定超过阈值后的处理行为

	* 支持限制单条 SQL 语句使用内存的大小，减少程序 OOM 风险

	* 支持在 CRUD 操作中使用隐式的行 ID

	* 提升点查性能

* Server

	* 支持 Proxy Protocol

	* 添加大量监控项, 优化日志

	* 支持配置文件的合法性检测

	* 支持 HTTP API 获取 TiDB 参数信息

	* 使用 Batch 方式 Resolve Lock，提升垃圾回收速度

	* 支持多线程垃圾回收

	* 支持 TLS

* 兼容性

	* 支持更多 MySQL 语法

	* 支持配置文件修改 `lower_case_table_names` 系统变量，用于支持 OGG 数据同步工具

	* 提升对 Navicat 的兼容性

	* 在 `Information_Schema` 中支持显示建表时间

	* 修复部分函数/表达式返回类型和 MySQL 不同的问题

	* 提升对 JDBC 兼容性

	* 支持更多的 `SQL_MODE`

* DDL

	* 优化 `Add Index` 的执行速度，部分场景下速度大幅度提升

	* `Add Index` 操作变更为低优先级，降低对线上业务影响

	* `Admin Show DDL Jobs` 输出更详细的 DDL 任务状态信息

	* 支持 `Admin Show DDL Job Queries JobID` 查询当前正在运行的 DDL 任务的原始语句

	* 支持 `Admin Recover Index` 命令，用于灾难恢复情况下修复索引数据
支持通过 `Alter` 语句修改 Table Options

## PD

* 增加 `Region Merge` 支持，合并数据删除后产生的空 Region [experimental]

* 增加 `Raft Learner` 支持 [experimental]

* 调度器优化

	* 调度器适应不同的 Region size

	* 提升 TiKV 宕机时数据恢复的优先级和恢复速度

	* 提升下线 TiKV 节点搬迁数据的速度

	* 优化 TiKV 节点空间不足时的调度策略，尽可能防止空间不足时磁盘被写满

	* 提升 balance-leader scheduler 的调度效率

	* 减少 balance-region scheduler 调度开销

	* 优化 hot-region scheduler 的执行效率

* 运维接口及配置

	* 增加 TLS 支持

	* 支持设置 PD leader 优先级

	* 支持基于 label 配置属性

	* 支持配置特定 label 的节点不调度 Region leader

	* 支持手动 Split Region，可用于处理单 Region 热点的问题

	* 支持打散指定 Region，用于某些情况下手动调整热点 Region 分布

	* 增加配置参数检查规则，完善配置项的合法性较验

* 调试接口

	* 增加 `Drop Region` 调试接口

	* 增加枚举各个 PD health 状态的接口

* 统计相关

	* 添加异常 Region 的统计

	* 添加 Region 隔离级别的统计

	* 添加调度相关 metrics

* 性能优化

	* PD leader 尽量与 etcd leader 保持同步，提升写入性能

	* 优化 Region heartbeat 性能，现可支持超过 100 万 Region

## TiKV

* 功能

	* 保护关键配置，防止错误修改

	* 支持 `Region Merge` [experimental]

	* 添加 `Raw DeleteRange` API

	* 添加 `GetMetric` API

	* 添加 `Raw Batch Put`，`Raw Batch Get`，`Raw Batch Delete` 和 `Raw Batch Scan`

	* 给 Raw KV API 增加 Column Family 参数，能对特定 Column Family 进行操作

	* Coprocessor 支持 streaming 模式，支持 streaming 聚合

	* 支持配置 Coprocessor 请求的超时时间

	* 心跳包携带时间戳

	* 支持在线修改 RocksDB 的一些参数，包括 `block-cache-size` 大小等

	* 支持配置 Coprocessor 遇到某些错误时的行为

	* 支持以导数据模式启动，减少导数据过程中的写放大

	* 支持手动对 region 进行对半 split

	* 完善数据修复工具 tikv-ctl

	* Coprocessor 返回更多的统计信息，以便指导 TiDB 的行为

	* 支持 ImportSST API，可以用于 SST 文件导入 [experimental]

	* 新增 TiKV Importer 二进制，与 TiDB Lightning 集成用于快速导入数据 [experimental]

* 性能

	* 使用 ReadPool 优化读性能，`raw_get/get/batch_get` 提升 30%

	* 提升 metrics 的性能

	* Raft snapshot 处理完之后立即通知 PD，加快调度速度

	* 解决 RocksDB 刷盘导致性能抖动问题

	* 提升在数据删除之后的空间回收

	* 加速启动过程中的垃圾清理过程

	* 使用 `DeleteFilesInRanges` 减少副本迁移时 I/O 开销

* 稳定性

	* 解决在 PD leader 发送切换的情况下 gRPC call 不返回问题

	* 解决由于 snapshot 导致下线节点慢的问题

	* 限制搬移副本临时占用的空间大小

	* 如果有 Region 长时间没有 Leader，进行上报

	* 根据 compaction 事件及时更新统计的 Region size

	* 限制单次 scan lock 请求的扫描的数据量，防止超时

	* 限制接收 snapshot 过程中的内存占用，防止 OOM

	* 提升 CI test 的速度

	* 解决由于 snapshot 太多导致的 OOM 问题

	* 配置 gRPC 的 `keepalive` 参数

	* 修复 Region 增多容易 OOM 的问题

## TiSpark

TiSpark 使用独立的版本号，现为 1.0 GA。TiSpark 1.0 版本组件提供了针对 TiDB 上的数据使用 Apache Spark 进行分布式计算的能力。

* 提供了针对 TiKV 读取的 gRPC 通信框架

* 提供了对 TiKV 组件数据的和通信协议部分的编码解码

* 提供了计算下推功能，包含

	* 聚合下推

	* 谓词下推

	* TopN 下推

	* Limit 下推

* 提供了索引相关支持

	* 谓词转化聚簇索引范围

	* 谓词转化次级索引

	* Index Only 查询优化

	* 运行时索引退化扫表优化

* 提供了基于代价优化

	* 统计信息支持

	* 索引选择

	* 广播表代价估算

* 多种 Spark Interface 的支持

	* Spark Shell 支持

	* ThriftServer/JDBC 支持

	* Spark-SQL 交互支持

	* PySpark Shell 支持

	* SparkR 支持

**如今，在社区和 PingCAP 技术团队的共同努力下，TiDB 2.0 GA 版已发布，在此感谢社区小伙伴们长久以来的参与和贡献。**

> 作为世界级开源的分布式关系型数据库，TiDB 灵感来自于 Google Spanner/F1，具备『分布式强一致性事务、在线弹性水平扩展、故障自恢复的高可用、跨数据中心多活』等核心特性。TiDB 于 2015 年 5 月在 GitHub 创建，同年 12 月发布 Alpha 版本，而后于 2016 年 6 月发布 Beta 版，12 月发布 RC1 版， 2017 年 3 月发布 RC2 版，6 月发布 RC3 版，8 月发布 RC4 版，10 月发版 TiDB 1.0，并在 2018 年 3 月发版 2.0 RC1。
