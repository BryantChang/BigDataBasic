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





