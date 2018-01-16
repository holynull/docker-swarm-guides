# Service工作原理

为了部署一个应用的镜像到Swarm模式的Docker Engine中，我们需要创建一个service。通常service是拥有大型应用系统上下文信息的微服务镜像。例如，一个服务可能包含一个HTTP服务器、一个数据库、或者其他的软件，我们需要这些软件运行在一个分布式环境中。

当创建service时，我们需要指定用什么镜像运行container，以及container内部运行什么样的命令。我们还需要定义以下内容：

- service向Swarm以外暴露的可用端口

- 一个`overlay network`，用来支持service之间相互通信

- CPU和内存的限制和预设

- 滚动更新的策略

- 镜像在Swarm中运行的副本数

## Service，Tasks and Containers

当我们部署一个service到swarm时，manager节点将接受我们对service定义，并作为service的理想状态。Manager节点在swarm节点之间调度service的副本。不同节点上运行的task是相互独立的。

例如，我们设想在3个HTTP监听实例之间实现负载均衡。下图展示了拥有3个副本的HTTP监听器。每一个监听器实例就是一个swarm中的task。

![](/assets/services-diagram.png)

一个container是一个被隔离的进程。在swarm模式的模型中，每一个task运行一个container。task就像调度器放置container使用的一个“槽”。一旦container运行起来，调度器就能意识到task是在运行状态。如果container不可用或者被终止，则task也将被终止。

## task和调度

task是swarm调度的原子单元。当创建和更新service时，我们声明了service的理想状态，协调器通过调度task来实现这个理想状态。例如，我们定义了一个service，要求协调器在任何时候都能保证有3个HTTP监听器在运行。协调器创建3个task作为回应。每一个task就像一个“槽”，调度器将产生3个container放在“槽”中。container是task的一个实例。当一个HTTP监听器不可用或者崩溃，协调器将创建一个新的task副本，并放入一个新的container。

task的状态变化采用一种单向机制。贯穿task生命周期的状态有：已分配状态、已准备状态、正在运行状态，等等。如果task运行失败了，协调器会移除这个task并停止它的container，然后创建新的task替代它，以保证service定义的理想状态。

Swarm模式背后的逻辑其实是一种通用的调度器和协调器逻辑。service和task被抽象设计成并不需要知道他们实现的container是什么样的。假设，我们可以实现其他类型的task，例如虚拟机task或者非容器化的进程task。调度器和协调器并不需要知道task的类型。然而，当前版本的Docker只支持容器task。

下图展示了Swarm模式如何接收service的创建请求，并调度task到worker节点。

![](/assets/service-lifecycle.png)

## Pending services

由于配置的原因，某些service在当前swarm中没有节点可以执行它的task。在这种情况下，service会一直保持`pending`状态。下面的一些例子会造成service保持`pending`状态。

> **注意：**如果有意要阻止service被发布，应该将service的scale设置为0，而不是通过设置让service一直保持在`pending`状态上。

- 如果所有的节点都暂停了或者都处于`DRAIN`状态，创建service时就会一直处于`pending`状态，知道有节点变为可用状态。事实上，当一个节点可用时，所有的任务都会被分配到这个节点上，所以对于生产环境来说并不是一件好事。

- 可以事先给service划定一定的内存。如果没有一个节点具有所需要的内存空间量，service会一直保持`pending`状态，直到一个节点变的可用。

- 可以强制service的放置约束，短时间内可能达不到约束条件。

以上说明需求和task的配置没有紧密的结合到swarm的当前状态。作为Swarm的管理员，生命Swarm的理想状态，manager节点结合其他节点在Swarm中创建出这个状态。就不需要将管理的粒度细化到task级别。

## 复制和全局service

有两种service的部署模式，复制和全局。

设置task的数量来实现复制模式下的service。例如，想要部署3个副本的HTTP服务，每个副本都提供相同的服务。

全局service是在所有节点上都有一个task的service。不能指定task的数量。每次增加一个节点，协调器会创建一个task，并有调度器分配到新节点上。全局service的最佳应用是部署监控代理，病毒扫描，以及其他希望在swarm中每个节点上都部署的service。


下图展示了3个副本的service和一个全局service部署在swarm中的情况，黄色代表3个副本的service，灰色代表全局service：

![](/assets/replicated-vs-global.png)





