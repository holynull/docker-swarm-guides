# Swarm模式关键概念

本章主要介绍一些Docker Engines1.12集群管理和业务流程特性的独特概念。

## Swarm是什么？

Docker Engines的集群管理和业务流程功能是由[swarmkit](https://github.com/docker/swarmkit/)构建的。Swarmkit是一个独立的项目，由Docker直接调用来实现Docker的业务流程层。

Swarm由多个运行了swarm模式的宿主机组成，每一个宿主机扮演了manager角色或者worker角色，再或者即扮演了manager角色也扮演了worker角色。Manager是管理节点关系和委托的节点，而worker是用来运行service的节点。创建service前，我们需要定义一些参数来使service运行时达到一个期望的最优状态，包括副本数，网络，可用的存储资源，service对外暴露的端口，等等。Docker将不断的监控service的运行状态，一旦运行状态发生变化，Docker将对service进行调整使状态回到预设状态。例如，当一个worker节点不可用时，Docker将调度这个节点上的task到其他可用的节点上。Swarm Service由很多个container组成。而`task`是swarm service中的一个container，并有manager节点进行管理，而不是一个独立的container。

Swarm service不同于独立的container的一个好处是可以修改service的配置，包括网络、存储挂载，而不用手动重启service。Docker将会更新配置，停止所有过期配置的service的task，然后根据新配置创建新的task。

当Docker以swarm模式运行时，也可以在Docker下运行独立的container。独立的contianer和swarm service一个主要的不同点是，只有swarm的manager才能管理swarm，而独立的container可以在任何一个daemon上启动。Docker daemon可以以manager身份、worker身份，或者同时以两种身份加入到swarm中。

就像使用Docker Compose一样，我们可以使用Swarm service stacks来定义和运行container。

下面我们将继续介绍关于Docker swarm services的相关概念，包括：nodes、services、tasks和load balancing。

## Nodes

Node是组成swarm的Docker engine的一个实例。你也可以这样理解，我们可以运行多个node在单独的一个物理计算机服务器上或者云服务器上，但是生产环境通常会要求我们将Docker node分散部署在多个物理主机或者云服务器上。

To deploy your application to a swarm, you submit a service definition to a manager node. The manager node dispatches units of work called tasks to worker nodes.

部署应用到swarm时，首先需要向manager提交要发布的service的定义（配置）。manager节点会对worker节点派发task。

Manager nodes also perform the orchestration and cluster management functions required to maintain the desired state of the swarm. Manager nodes elect a single leader to conduct orchestration tasks.

manager节点同时具有执行业务流程的功能，以及为了维持swarm状态而必须的集群管理功能。所有的manager节点会推选一个节点作为leader来实施业务流程任务。

Worker nodes receive and execute tasks dispatched from manager nodes. By default manager nodes also run services as worker nodes, but you can configure them to run manager tasks exclusively and be manager-only nodes. An agent runs on each worker node and reports on the tasks assigned to it. The worker node notifies the manager node of the current state of its assigned tasks so that the manager can maintain the desired state of each worker.

worker节点接收并执行manager节点分配的任务。manager节点默认是可以运行service的，除非我们设置manager节点仅作为manager角色执行管理任务。每个worker节点上会有一个agent，并且不断的汇报分配到这个节点上的task执行情况。woker节点向manager节点汇报分配给它的任务的当前状态，manager节点才能根据task的运行的情况来维持每一个workder上的状态。

## Services and tasks
A service is the definition of the tasks to execute on the manager or worker nodes. It is the central structure of the swarm system and the primary root of user interaction with the swarm.

Service是将要在manager或者worker节点上执行的task的定义。它在swarm系统的结构中处于中心位置，并且是用户的主要操作对象。

When you create a service, you specify which container image to use and which commands to execute inside running containers.

当创建service时，你可以执行container使用哪个镜像，并设定将在container内执行哪些命令。

In the replicated services model, the swarm manager distributes a specific number of replica tasks among the nodes based upon the scale you set in the desired state.

在service的副本模式情况下，manager会使swarm生成n个任务，并分配到所有的node上执行，n等于指定的副本数量。副本数量就是service的container的数量（即，service的scale）。

For global services, the swarm runs one task for the service on every available node in the cluster.

对于全局services，swarm会在集群的每一个node上执行一个任务。

A task carries a Docker container and the commands to run inside the container. It is the atomic scheduling unit of swarm. Manager nodes assign tasks to worker nodes according to the number of replicas set in the service scale. Once a task is assigned to a node, it cannot move to another node. It can only run on the assigned node or fail.

每一个task包括一个运行的container，以及在container内部运行的命令。task是swarm调度的原子单元。manager将根据副本数生成的task分配到所有worker节点上。一定一个task被分配到一个node上，将不能在移动到其他节点上。所以，task状态的原子性在于task只能在一个node上运行或者失败。

## Load balancing
The swarm manager uses ingress load balancing to expose the services you want to make available externally to the swarm. The swarm manager can automatically assign the service a PublishedPort or you can configure a PublishedPort for the service. You can specify any unused port. If you do not specify a port, the swarm manager assigns the service a port in the 30000-32767 range.

manager使用入口负载均衡向swarm之外暴露service的端口。manager可以自动为service分配PublishedPort，也可以通过设置指定PublishedPort。如果没有指定端口号，manager将从30000到32767之间选择一个端口号分配给service。

External components, such as cloud load balancers, can access the service on the PublishedPort of any node in the cluster whether or not the node is currently running the task for the service. All nodes in the swarm route ingress connections to a running task instance.

外部组件，例如cloud load balancers，可以通过service的PublishedPort访问到集群中的某一个节点，然而这个节点并不一定正在运行端口对应service的task。但是swarm中所有的节点都会将入口链接路由指向到对应的service上正在运行的task实例。

Swarm mode has an internal DNS component that automatically assigns each service in the swarm a DNS entry. The swarm manager uses internal load balancing to distribute requests among services within the cluster based upon the DNS name of the service.



