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

![Time Synchronous](https://raw.githubusercontent.com/BryantChang/BigDataBasic/master/distributed_system/distributed_storage/imgs/timestamp.png)







