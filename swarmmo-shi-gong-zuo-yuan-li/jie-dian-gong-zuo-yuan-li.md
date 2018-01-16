# 节点工作原理

Docker Engine 1.12中的Swarm模式帮助我们创建Docker Engine集群，我们称之为Swarm。一个Swarm是由安装了Docker Engine的物理机或者虚拟机节点组成，这些节点上的Docker Engine都采用Swarm模式运行。

Swarm集群中的节点分为两类：manager和worker。

![](/assets/swarm-diagram.png)

## Manager节点

Manager节点用来处理集群管理任务：

- 维护集群状态

- 调度service

- 提供swarm模式[HTTP API](https://docs.docker.com/engine/api/) 端点

Manager通过[Raft](https://raft.github.io/raft.pdf)的实现，来使Swarm的整体维持在一个一致的内部状态上，service则运行在这个一致的内部状态之上。为了测试的目的，我们可以使用单独manager节点。如果单独的manager节点出现问题，service依然会一直运行，但是这样就需要创建一个新的集群来恢复一致的内部状态维护。

利用Swarm模式的容错特性，推荐使Swarm的节点为奇数个，来实现高可用的要求。当采用多个manager节点时，我们就可以在不停止集群运行的情况下恢复停止运行的manager节点。

- 3个manager节点最大容错允许1个manager不可用

- 5个manager节点最大容错允许2个manager不可用。

- N个manager节点最大容错允许`(N-1)/2`个manager节点不可用。

- 建议最大manager节点数为7个。

    > **重要提示：**不限制的增加manager节点并不会增加可扩展性或者增加执行性能。通常还会起到相反的作用。
    
## Worker节点

Worker节点也是Docker Engine的实例，目的是用来运行Container。Worker节点不会像Manager节点那样提供集群的管理、任务调度和API。

可以创建仅有一个manager节点的Swarm，但是不能在没有manager节点的前提下，创建Worker节点。在默认情况下，manager节点同时也是一个worker节点。在一个manager节点的集群中，执行`docker service create`命令，调度器会将所有的task调度在本地Docker Engine上执行。

要阻止调度器将task分配到manager节点上执行，就绪要将manager节点的可用性状态设置成`DRAIN`。调度器会停止`DRAIN`节点上的task，转移到其他状态为`ACTIVE`的节点上继续运行停止的task。并且调度器不会再将新的task分配给`DRAIN`状态的节点。
    
可以查看`docker node update`命令的帮助，学习如何修改节点的可用性状态。

## 改变节点角色

我们可以通过`docker node promote`来将一个worker节点修改成一个manager节点。例如，某些维护需要，我们将停止一个manager节点，这时我们可以将某一个正在运行的worker节点变成manager节点，来代替停止的那个manager节点。详情请参考：[Node promote](https://docs.docker.com/engine/reference/commandline/node_promote/)

同样也可将一个manager节点改成一个worker节点。详情请参考：[Node demote](https://docs.docker.com/engine/reference/commandline/node_demote/)
    
    