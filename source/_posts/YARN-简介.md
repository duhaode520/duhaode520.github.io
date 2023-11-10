---
title: Yarn 简介
date: 2023-03-17 20:17:27
tags:
    - Hadoop
    - 引擎基础
catagories:
    - Hadoop 权威指南读书笔记
---
## Yarn的运行机制

![Yarn 的运行机制](yarn_arch.png)

Yarn中三类重要实体

- resource manager：管理集群上资源使用
- node manager：运行在所有节点上，能够启动和监控容器
- Container：用于执行特定的应用程序，有一定的资源限制

运行流程：

- 客户端联系 resource manager，要求启动一个 application master 进程
- resource manager 找到一个可以启动 application master 的 node manager，然后启动这个容器
    - application运行起来后可能是返回一个计算 application 给客户端，也可能是执行分布式计算——申请资源、创建新的 application 和容器

Yarn 本身不提供通信机制，只是资源的申请分配

申请分配为了节省带宽还是采用了在网络拓扑上的就近（节点——机架——any）

按照用户和application之间来分类

- 一个用户作业对应一个application：MapReduce
- 用户作业中的一个工作流对应一个application：Spark
- 多个也用户对一个 application： Slider

{% note primary %}
🔴 其实就是：一对一，一对多，多对一
{% endnote %}

## Yarn的优势

- Scalability：Yarn中对于jobtracker 的资源调度和任务监控的职责分离带来的好处
- Availability：同样是职责分离，resource manager 和 application master 可以分别做高可用
- Utilization: Yarn 对于资源的管理更精细，slot比较大
- Multitenancy：Yarn 可以为更多的应用提供服务，MapReduce仅是期中的一个

## Yarn的调度

三种调度器：

- FIFO调度器：简单不用配置，不适合共享集群，每个任务必须都能到轮到自己才行
    {% note primary %}
     感觉基本上只适合自己玩玩的时候用，或者资源特别丰盛的时候，每个人实际上own了一个集群可以自己玩 
    {% endnote %}   

- **容量调度器：**有一个独立的队列保证小作业的提交和启动，大作业等待的时间相对长 (default)

- 公平调度器：每一个用户平分所有资源（在原有任务释放出来后），每一个用户的任务再分别平分这个用户拥有的所有的资源

**容量调度器的配置 `capacity-scheduler.xml`：**

可以配置每一个队列拥有的固定的 capacity（%） 和用于弹性使用的 max capacity（不设置默认可以占用全部的容量）, 队列还可以通过 `a.b.queues` 的方式定义子队列

通过属性 `mapreduce.job.queuename` 决定任务推送的队列

**公平调度器的配置 `fair-scheduler.xml`：**

- 权重元素：可以设置哪一个队列的占比更大，通过 `queue.weight` 属性
- 调度策略：默认是 fair，也可以有队列级的 fifo（只对该队列中的各个任务），DRF (Dominant Resource Fairness, 根据占用最多的资源进行 fair 调度)
- 可以通过 `queuePlacementPolicy` 设置队列放置规则，默认会有 “specified”, “user”, “primaryGroup”, “default” 等
  
    {% note primary %}
    💡 如果只使用 default 则相当于便捷的实现了对于所有任务的公平调度，因为所有任务都会被塞进一个队列中
    {% endnote %}
    
- 抢占：公平调度器允许抢占，但是抢占会降低集群的效率

****延迟调度****

一个非常tricky的调度策略：如果请求节点容器时发现没有，可以等待数秒（数个heartbeat（一般是1s）），这样可以显著提高在当前节点获得一个容器的机会，而不是下一级节点（节点→机架→others）

每一个 heartbeat 是一个潜在的调度机会
