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

task的状态变化采用一种单向机制。贯穿task生命周期的状态有：已分配状态、已准备状态、正在运行状态，等等。如果task运行失败了，调度器会移除这个task并停止它的container，然后创建新的task替代它，以保证service定义的理想状态。

