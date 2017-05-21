## 第二部分：分布式存储

### 1、大数据对分布式存储的需求

* 存储容量大
* 高吞吐量
* 高数据可靠性
* 服务高可用(99.95%每年4-5小时不可用)
* 高效运维
* 低成本

### 2、分布式存储系统面临的挑战

* 磁盘错误
* Raid卡故障
    - Raid卡用于高可用节点，用于存储Meta数据

![raid卡](https://raw.githubusercontent.com/BryantChang/BigDataBasic/master/distributed_system/distributed_storage/imgs/raid_card.png)

* 网络故障
    - 避免把所有的数据全部存处在同一个机架内

![网络拓扑](https://raw.githubusercontent.com/BryantChang/BigDataBasic/master/distributed_system/distributed_storage/imgs/network_topology.png)

* 电源故障（Rack级别，机房断电）
    - 保证数据及时落盘

![Memory Buffer](https://raw.githubusercontent.com/BryantChang/BigDataBasic/master/distributed_system/distributed_storage/imgs/memcache.png)

* 数据错误
    - 原因：
        + 磁盘
        + 网络
        + 内存（指针异常，ECC错误）
    - 方案：
        + CRC校验

* 系统异常
    - Linux时间
    - 尽量避免使用机器时间（时间同步时会出现负数或0使进程崩溃）
    - 共享memorybuffer，磁盘损坏导致缓冲区数据无法刷入磁盘进而导致好的磁盘请求也无法处理

![Time Synchronous](https://raw.githubusercontent.com/BryantChang/BigDataBasic/master/distributed_system/distributed_storage/imgs/timestamp.png)

* 热点
    - 如何避免热点频繁迅速的迁移

* 软件bug
    - metaserver管理
        + 副本写失败直接切换文件导致meta数据激增。
        + 数据重试导致meta数据变化。
        + 用户频繁发送读请求导致请求队列打满，无法服务其他用户
* 误操作
    - 误删除目录

### 3、典型分布式存储系统

* Ceph(Open Stack)
* HDFS
* Pangu

#### Ceph

![Ceph](https://raw.githubusercontent.com/BryantChang/BigDataBasic/master/distributed_system/distributed_storage/imgs/ceph_architecture.png)

* 块存储系统
* 无中心节点

#### HDFS

![HDFS](https://raw.githubusercontent.com/BryantChang/BigDataBasic/master/distributed_system/distributed_storage/imgs/hdfs_architecture.png)

* NameNode是单点
* client与NameNode交互负责拿到操作信息和meta信息
* Client与DataNode交互真正负责数据的读写

#### Pangu

* 使用Pasox协议实现PanguMaster,只要多余一般的节点工作就可以工作，去掉了单点问题
* 全链路Checksum
* 异常恢复机制
* 回收站防止误操作
* 数据聚族
* 流量控制与慢盘规避
* 混合存储
* 审计功能


### 4、分布式存储系统设计要点

* 读写流程
* Qos服务质量
* checksum 保证数据的正确性
* replication
* rebalance
* Garbage Collection(异步删除，基于版本)
* Erasure Coding(节约存储空间)

### 5、写入流程

* 链式写入：高吞吐，高延迟

![Link Write](https://raw.githubusercontent.com/BryantChang/BigDataBasic/master/distributed_system/distributed_storage/imgs/link_write.png)

* 场景：从外部导入数据到集群内部
* 弊端：3段的网络延迟


* 主从写入：低吞吐，低延迟

![Master Slave](https://raw.githubusercontent.com/BryantChang/BigDataBasic/master/distributed_system/distributed_storage/imgs/master_slave.png)

* 使用辐射发包方式，导致Primary网络最高利用率为50%
* 不适合流量较高的应用
* 适合低延迟的应用

* Seal and new

![Seal and New](https://raw.githubusercontent.com/BryantChang/BigDataBasic/master/distributed_system/distributed_storage/imgs/seal_and_new.png)

### 6、读取流程

* 一般读流程

![Read Common](https://raw.githubusercontent.com/BryantChang/BigDataBasic/master/distributed_system/distributed_storage/imgs/read_common.png)

* Backup Read(规避慢节点)

![Backup Read](https://raw.githubusercontent.com/BryantChang/BigDataBasic/master/distributed_system/distributed_storage/imgs/backup_read.png)

* 核心点：
    - 同时发送多个请求
    - 接收到一个节点发送成功后向其他节点发送cancel，结束正在执行的I/O请求
* 不足：
    - 会出现过多没用的I/O请求

* 优化方法：
    - 当Client向Master请求数据位置的节点时，Master返回CS列表时同时会将每个CS的期望延迟返回给Client，Client先从中选取最小的一个进行请求，当超过期望值（该节点是热点）时选择次小的CS进行请求，并将实际的等待时间更新到列表中返回给Master.
* 好处：
    - 有效的发现获取当前数据最快的节点
    - 有效的规避了热点（全部）

### 7、QoS

* 对不同用户的请求进行计数
* 限制最高的请求个数


### 8、Checksum

* 对数据进行切块，每块的数据都求crc，分包传送。
* crc与数据同一个文件
    - 数据访问快
    - 校验准确性会受影响

### 9、Replication

* 及时发现坏盘，对其上数据进行复制
* 每隔15s从节点会向Master汇报心跳
* 在复制时充分利用多台机器的网络带宽
* 流量控制
* 按照优先级复制（越接近丢失的数据优先级越高）

### 10、Rebalance（一种特殊情况下的Replication）

* 产生原因：
    - 新加机器
    - 本身不均衡
* 做法：
    - 充分利用多台机器的带宽
    - 复制存在优先级（优先级小于复制）
    - 流量控制

### 11、Garbage Collection

* 场景：
    - 数据被删除（异步删除，先删meta，异步删数据）
    - 写失败，会出现脏数据留在磁盘上
    - 机器宕机

* 触发机制：
    - 某个数据在Master上没有对应的meta
    - 某个数据的版本比当前Master中的meta中该数据的实际版本
    - 某数据的拷贝份数多于meta记录的 


### 12、Erasure Coding 数据压缩（用更少的空间存储更多的数据副本）

* m份数据
* 计算出n份数据
* 最终数据存储了m+n份
* 允许n份同时丢失

![Erasure Coding](https://raw.githubusercontent.com/BryantChang/BigDataBasic/master/distributed_system/distributed_storage/imgs/ensurecoding.png)


### 13、Meta服务高可用

* 一般实现方法：
    - 多个备份互为热备，在故障时可以快速切换
    - 多个备份保证状态一致

* 主从模式：
    - 一个为主服务，其他节点为从节点
    - 通过分布式锁互斥技术进行主节点选取
    - 通过共享存储实现主从之间的数据一致性（主节点通过日志持久化到共享存储）
    - 特点：
        + 简单
        + 所需的其他模块需要依赖于其他模块（分布式锁服务，共享存储）
* 依据分布式协议：
    - paxos，raft等分布式协商协议
    - 独立自包含

* 典型分布式存储的做法
    - HDFS：
        + 主从模式
        + 使用zk提供分布式锁服务
        + 使用NFS提供共享存储
    - Ceph:
        + 主从模式
        + 使用天然的osd作为共享存储
        + 通过节点间的心跳维持代替分布式锁服务
    - Pangu:
        + 基于分布式协议Raft

* Raft协议详解
    - 正常选举流程：
        + 其中一个elector向其他节点发送propose请求
        + 接收到请求的节点进入锁定状态，不再接受其他propose请求
        + 发送请求方升级自己为primary，其他节点降级为secondary,primary节点与secondary节点间维持心跳

![Common Election](https://raw.githubusercontent.com/BryantChang/BigDataBasic/master/distributed_system/distributed_storage/imgs/common_election.png)

- primary节点异常
    + 当secondary超过约定时间无法收到心跳，自动升级为elector
    + elector重新发送propose请求，重新正常选举流程

![Primary Exception](https://raw.githubusercontent.com/BryantChang/BigDataBasic/master/distributed_system/distributed_storage/imgs/primary_exception.png)

- 节点恢复
    + primary首先降级为elector，向当前primary节点发送propose请求
    + 当前primary节点发送deny并发送publish消息
    + 接收到消息的elector降级为secondary

![Node Recovery](https://raw.githubusercontent.com/BryantChang/BigDataBasic/master/distributed_system/distributed_storage/imgs/node_recovery.png)

* Paxos协议
    - 可信赖的分布式协商协议
    - 关键点
        + 大多数协商一致（数据一致性保证）
        + 两阶段协商
        + 协议号递增并需要持久化（协议正确性前提）
    - prepare阶段
        + client向proposer发送修改请求，proposer向accepter发总prepare请求
        + accepter接收到请求，锁定当前协议号，同时不再接收相同版本号的请求
    - 协商返回阶段
        + proposer发送相同协议号的accept请求给accepter，若大多数accepter接受这个请求，则数据已成功修改，通过proposer返回给client端。

### 14、Meta server的线性扩展

* HDFS: Namenode Federation模式
    - 将目录树进行切分，每个目录树各自独立
    - 目录数之下共享DataNode
    - 问题：
        + 相关的目录会被切分
        + 节点间的切分不均衡

* Ceph: MDS的动态切分
    - MDS统计目录树的深度，基于统计做目录树的动态切分
    - 好处：
        + 元数据在不同节点的数量相对平均分布
        + 扩展能力强
    - 问题：
        + 实际应用较少

### 15、混合存储

#### 不同存储介质的特征

*   磁盘：
    -  Capacity:1-4TB
    -  Latency:10ms
    -  Throughput:100~200MB/s
    -  Cost:低

*   SSD：
    -  Capacity:400~800GB
    -  Latency:50~75us
    -  Throughput:400MB/s
    -  Cost:中

*   内存：
    -  Capacity:24~128GB
    -  Latency:100ns
    -  Throughput:20GB/s
    -  Cost:高






