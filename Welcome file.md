
# 基本概念和架构

## 基本概念
### 1. RDD
Resillient Distributed Dataset 弹性分布式数据集，是分布式内存的一个抽象概念，提供了一种高度受限的共享内存模型

### 2. DAG
Directed Acyclic Graph 有向无环图，反应 RDD 之间的依赖关系

### 3. Executor
运行在工作节点（WorkerNode）上的一个进程，负责运行 Task

### 4. Application
用户编写的 Spark 应用程序

### 5. Task
运行在 Execotr 上的工作单元

### 6. Job
一个 Job 包含多个 RDD 及作用于相应 RDD 上的各种操作

### 7. Stage
是 Job 的基本调度单位，一个 Job 会分为多组 Task，每组 Task 成为 Stage，或者 TaskSet，代表一组关联的，相互之间没有 Shuffle 依赖关系的任务组成的任务集

## 运行架构

![运行架构图](/A01.png)

Spark有点：
1、利用多线程执行具体的任务，减少任务启动开销
2、Executor 中有一个 BlockManager 存储模块，结合内存和磁盘作为存储设备，减少 IO 开销

## Spark 运行流程

![运行流程图](/A02.png)

### STEP 1
- 为应用构建基本的运行环境，由 Driver 创建一个 SparkContext 进行资源的申请、任务的分配和监控。

### STEP 2
- 资源管理器为 Executor 分配资源，并启动 Executor 进程。

### STEP 3
- SparkContext 根据 RDD 的依赖关系构建 DAG 图，DAG 图提交给 DAGScheduler 解析成 Stage，然后把一个个 TaskSet 提交给底层调度器 TaskSchedule 处理。

- Executor 向 SprakContext 申请 Task。

- TaskScheduler 将 Task 分发给 Executor ，并提供应用程序代码

### STEP 4
- Task 在 Excutor 上运行，把执行结果反馈给 TaskScheduler，然后反馈给 DAGScheduler，运行完成之后写入数据并释放资源。

## Spark 运行特点
1、每个 Application 都有自己专属的 Executor 进程，并且该进程在 Application 运行期间一直驻留。Executor进程以多线程的方式运行 Task。
2、Spark 运行过程与资源管理器无关，只要能够获取 Executor 进程并保持通讯即可
3、Task 采用了数据本地性和推测执行优化机制

# RDD

<!--stackedit_data:
eyJoaXN0b3J5IjpbLTExODEwNzY2NzNdfQ==
-->