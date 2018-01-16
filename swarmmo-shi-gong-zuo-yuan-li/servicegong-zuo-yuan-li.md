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

