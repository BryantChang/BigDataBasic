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




















