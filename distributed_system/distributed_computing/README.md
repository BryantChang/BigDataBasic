## 第三部分：分布式计算

### 增量计算与流计算

```
流计算是一种特殊的增量计算
```

* 实时计算：
    - ad_hoc_computing 数据不可枚举（数据的实时计算）
    - streaming_computing 数据可枚举，确定性的，计算在数据发生变化时产生（实时数据的计算）
    - continueous_computing（实时数据的实时计算）

### 增量模型定义

```
f(x+delta) = g(f(x), delta)
等式左面表示批量计算的结果
等式右面f(x)是oldValue(是状态)，delta是新的数据
该等是表示增量计算与批量计算在计算结果上应该是完全相同的
增量计算与批量计算都是有状态的计算
```

* 增量计算是系统时效性，系统性能与系统复杂性的tradeoff
* 增量计算的优势：
    - 中间结果实时输出
    - 分摊计算峰值
    - 中间结果膨胀较小
    - 有状态的failover(容错速度快)
    - 较好的克服数据倾斜
* 增量计算与流计算的应用场景
    - 日志采集与在线分析
    - 大数据的ETL
    - 风险监控与告警
    - 网站与移动应用的统计分析
    - 网络安全检测
    - 在线的服务计量计费
* 流数据的特征：
    - 有向无界
    - 不可控性（不同数据通路的数据到达时机不可控）

### 全量计算与増量计算的区别

* 全量计算：
    - 粒度：partition/文件
    - 计算：局部
    - 生命周期：处理完该进程就“退出”
    - 容错监控：对进程监控
    - 面向：吞吐
    - DAG：串行

* 增量计算：
    - 粒度：batch
    - 计算：有状态
    - 生命周期：keepalive--长进程
    - 容错监控：数据
    - 面向：延时
    - DAG：并行

### 典型流式计算系统

#### Storm Twitter

* 核心概念
    - Topology:完整的计算作业
    - spout:收集数据任务（自驱动）
    - bolt:数据触发，相关的计算任务
    - Task:调度的最小单位，负责某一数据分片处理的实体
    - Acker:负责跟踪，消息是否被处理的节点

* 使用异或的特性来判断消息是否被处理

![Storm](https://raw.githubusercontent.com/BryantChang/BigDataBasic/master/distributed_system/distributed_computing/imgs/storm.png)

* 容错：

```
数据接收->发送->处理->Acker成功（丢失，重复）
```

* Transection Topology
    - 跟踪源头的所有子孙消息（异或）
    - 限制：
        + 整个Topology统一时刻只有一个batch正在执行
    - 优点：
        + 消息在框架内不落盘
        + 保证消息至少被处理
    - 不足：
        + Transaction Topology对batch串行方式，大幅影响性能

### Kinesis（Amazon）

![Kinesis](https://raw.githubusercontent.com/BryantChang/BigDataBasic/master/distributed_system/distributed_computing/imgs/kinesis.png)

* 每一级消息都落盘

### MillWheel（Google）

* 使用BigTable持久化中间结果
* 将每个节点的计算进行持久化


### 数据收集

* 输入
    - 拉：
        + kafka
        + HBase,HDFS
    - 推：
        + 需要实现HTTP处理模块
* 输出
    - 业务方订阅：
        + 结果数据写入消息队列，业务方订阅
    - 服务
        + 直接提供在线的数据服务

* Shuffle机制
    - 下游节点拉
    - 上游节点推（需要知道子DAG的全部拓扑）

### 计算

* 长进程，不同的调度，不同的消息机制
* 容错：任务跟踪
* 有状态的计算：中间状态的存储
* Batch
    - 系统跟踪的最小单位
    - 数据的最小单位
    - 处理的最小单位
    - 能够Scale化
        + 全量计算
        + 一条数据
* 消息机制：
    - 分发->接收->处理
    - shuffle-服务化
    - 异常情况
        + 程序crash
        + 容错
        + 网络问题
    - 消息源头重发：
        + 无异常情况执行流畅
        + 出现异常时如果DAG规模较大时恢复时间较长
    - 节点内部重发
        + 出现异常时恢复迅速
        + 每一步都需要落磁盘，影响系统性能

### 增量计算模型

* Map Reduce Merge
* 有状态的计算
* 状态处理：
    - Storm && Kinesis 将状态处理交给用户
    - MillWheel 使用BigTable持久化中间状态（系统性能受限与外部存储的最大吞吐限制）
    - 备选方案：存储与计算合一（将小的snapshot写入到内存，或延展到SSD，聚集后刷入持久化存储，存储不会阻塞计算）
* 串行DAG（重吞吐，兼顾延时）
    - 优点：
        + 模型简单
        + 吞吐高
    - 缺点：
        + 时效性差
        + 数据倾斜严重
* 并行DAG（重延时，兼顾吞吐）
    - 优点：
        + 失效性好
        + 对倾斜较为友好
    - 不足：
        + 调度复杂
        + 建模复杂










