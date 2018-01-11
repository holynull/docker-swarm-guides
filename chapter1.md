# 概述

Docker在1.12.0和以上版本中增加了`swarm`模式。通过`swarm`模式管理的Docker Engines集群称之为“Swarm”。并可以通过Docker CLI来创建Swarm，在Swarm中部署应用服务，以及管理Swarm。

## 功能特点

- **在Docker Engines中集成了集群管理功能：**通过Docker CLI可以创建一个用来发布应用`services`的`swarm`，而不需要安装其他任何额外的软件来管理`swarm`。

- **分散式设计：**与在部署时确定节点角色的处理方式不同，Docker Engines在运行时才会有节点角色的区分。我们可以把节点部署成为即是`mananger`节点同样也是`worker`节点，这意味着我们可以在一个单一的磁盘镜像上创建一个Swarm。

- **规模扩展和收缩：**对于每一个`service`可以定义运行`task`的数量，即`service`的运行规模。当我们对`service`运行的数量进行调整时，`manager`节点会自动的增加或者移除`task`来实现我们所指定的`service`运行规模。

- **状态维持：**manager节点会不断的监视我们设定的swarm的状态和实际状态的变化。例如，我们设定一个service的一个container需要有10个副本，但是其中保存2个副本的workder节点崩溃了，这是manager节点会在其他可用的worker节点上分配两个新副本，并将崩溃的副本移除掉。

- **多宿主机网络：**可以指定`overlay network`给所有的service。当初始化时或者更新应用时，manager节点会为每一个container分配一个在`overlay network`上的地址。

- **服务发现：**manager节点会为每一个swarm中的service分配一个唯一的DNS name，对service的调用会通过负载均衡策略，指向其中一个container上。我们可以通过swarm内部的DNS服务器查询到swarm中正在运行的每一个container。

- **负载均衡：**我们可以通过暴露service的端口给外部的负载均衡设备。而在swarm内部我们可以指定service的container如何在节点上分布。

- **默认安全机制：**在swarm中节点和节点间的通信强制使用TLS协议进行安全通信。用户可以使用自己签署的证书，或者其他CA中心的证书。

- **回滚更新：**在发布应用服务更新时，部署的更新会从一个节点逐渐蔓延到整个swarm。manager节点可以让你控制蔓延的时延，一但更新过程中发生问题，可以使task回滚到之前的一个版本。
