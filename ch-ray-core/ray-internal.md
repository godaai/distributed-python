(ray-internal)=
# Ray 系统与设计

## Ray 集群

如 {numref}`ray-cluster` 所示，Ray 集群由一系列计算节点组成，其中两类关键的节点：头节点（Head node）和工作节点（Worker node）。这些节点可以部署在虚拟机、容器或者是裸金属服务器上。

```{figure} ../img/ch-ray-core/ray-cluster.svg
---
width: 800px
name: ray-cluster
---
Ray 集群
```

所有节点上都运行着一些进程：

* Worker

每个计算节点上运行着一个或多个 Worker 进程，Worker 进程负责计算任务的运行。每个 Worker 进程运行特定的计算任务。Worker 进程或者是无状态的，即可以被反复执行 Remote Function 对应的 Task；又或者是一个 Actor，即只能执行有状态的 Remote Class 的方法。默认情况下，Worker 的数量等于其所在的计算节点的 CPU 核数。

* Raylet

每个计算节点上运行着一个 Raylet。与一个计算节点上运行多个 Worker 进程不同，每个计算节点上只有一个 Raylet 进程，或者说 Raylet 被多个 Worker 进程所共享。Raylet 主要有两个组件：一个调度器（Scheduler），负责资源管理、任务分配等。各个计算节点上的 Scheduler 共同组成了整个 Ray 集群的分布式调度器；一个基于共享内存的对象存储（Share-memory Object Store），负责本地的数据存储，各个计算节点上的 Object Store 共同组成了 Ray 的分布式对象存储。

从 {numref}`ray-cluster` 中也可以看到，头节点还多了：

* Global Control Service（GCS）

GCS 是 Ray 集群的全局的元数据管理服务，这里的元数据信息比如某个 Actor 被分配到哪个计算节点上。它管理的元数据是所有 Worker 共享的。

* Driver

Driver 执行一些顶层的应用，比如，作为 Python 入口的  `__main__` 函数。