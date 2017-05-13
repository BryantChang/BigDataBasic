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




