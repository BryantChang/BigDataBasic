# 分布式系统知识总结

## 第一部分：分布式调度系统

### 1、主要问题：

* 任务调度
* 资源调度
* 容错机制
* 规模挑战
* 安全与性能隔离

### 2、典型分布式调度系统

#### Hadoop MR

![image](https://raw.githubusercontent.com/BryantChang/BigDataBasic/master/distributed_system/imgs/hadoop_mr.png)

##### 资源申请流程

```
1、用户通过client向Job Tracker提交任务
2、Job Tracker进行资源分配与任务分配
```

##### 存在问题

```
1、Job Tracker是单点，容错性差
2、资源调度与任务调度未进行分离，不支持热拔插，不支持多种调度策略共存
3、扩展性不强，Task Tracker向Job Tracker注册过程中可能会超出Job Tracker内存上限
```

#### YARN

![image](https://raw.githubusercontent.com/BryantChang/BigDataBasic/master/distributed_system/imgs/yarn.png)

##### 资源申请流程

```
1、用户通过client向Resource Manager提交任务
2、Resource Manager在适当的节点启动该任务所对应的App Master
3、App Master启动后向Resource Manager申请任务实际运行所需要的资源
4、RM将资源分配结果返回给AM
5、AM在具体的NM上启动对应的Container运行任务
6、任务执行完毕，NM回收Container归还相应资源
```

##### 与HadoopMR的区别

```
1、将任务调度与资源调度区分开
2、可支持更大的计算规模
```


##### 存在问题

```
1、RM仅支持内存维度的调度
2、资源交互性能（Container无法复用）
```

#### Mesos(Twitter)

![image](https://raw.githubusercontent.com/BryantChang/BigDataBasic/master/distributed_system/imgs/mesos.png)

##### 资源申请流程

```
1、Scheduler向Mesos Master发起资源请求。
2、Mesos Master经结果发给Scheduler,由Schduler进行任务调度。
```

##### 问题

```
1、scheduler 与 master不能描述精确的资源需求
2、一次资源分配需要两次通信交互，效率不高
3、不支持资源抢占
```


4、Ali Fuxi

![image](https://raw.githubusercontent.com/BryantChang/BigDataBasic/master/distributed_system/imgs/fuxi.png)

##### 资源申请流程

```
1、用户通过Client 向Fuxi Master提交任务
2、FuxiMaster 在空闲的节点启动APP Master
3、App Master 发送资源请求给FuxiMaster
4、FuxiMaster 将结果返回给App Master
5、AppMaster 通知对应的Tubo进程启动App Worker
6、App Worker 向App Master注册
7、App Master 向App Worker下发Instance(数据分片，存储位置，处理结果)
```


### 3、任务调度--海量数据如何进行兵法处理

* 将输入数据切分，做映射，在进行归约

#### 技术要点

* 数据Locality
    - 不是只考虑最大的数据本地化，还要综合考虑是否负载均衡
* 数据Shuffle
    - 3种形态：1:1,1:N,M:N
* Instance重试和Backup Instance
    - 长尾任务（由于集群中节点性能不同）
    - Backup Instance触发机制
        + Instance运行时间超过其他Instance
        + 处理速度远低于其他Instance
        + 已完成的Instance比例

### 4、资源调度

#### 目标

* 最大化集群资源利用率
* 最小化任务等待时间
* 支持资源配额
* 支持任务抢占

##### 策略优先级

* Ali Fuxi做法
    - 每个Job有有衔接，整数值，越小越高
    - 优先级相同按照提交时间
    - 优先分配给高优先级的任务，剩余资源分配给次优先级Job

##### 任务抢占

* Ali Fuxi做法
    - 从最低优先级任务强制收回，分配给紧急任务
    - 抢占递归，指导被抢占任务优先级不高于紧急任务

##### 资源配额

* Ali Fuxi做法
    - 多个任务组成任务组按业务区分
    - 每个组的走也按所分配的资源付费
    - 没用完的配额按比例发给其他分组
    - 可共享的资源便宜
    - 最低保证的资源贵

### 5、容错

#### 考虑的因素

* 正在运行的任务不能被中断
* 对用户透明
* 自动故障恢复
* 系统恢复时保证可用性

##### Ali Fuxi做法

###### 任务调度恢复

* App Worker错误通过Instance重试进行恢复
* App Master错误通过App Worker上报的持久化CheckPoint进行恢复

###### 资源调度恢复

* 两种状态Soft State(可通过其他组件推出) Hard State(用户初始提交的任务配置信息)

![image](https://raw.githubusercontent.com/BryantChang/BigDataBasic/master/distributed_system/imgs/failover.png)
 

### 5、Scale Out

* 多线程异步
    - 线程池分离

* 增量消息通信

### 6、安全与性能隔离

* Capability
* Token
* Sandbox

### 7、未来方向

* 在线应用于离线应用混跑（超迈：夜间在线机器上调度一些离线任务）
* 实时计算
* 更大规模














