# Swarm中的manager节点

Swarm中的manager节点使用[Raft Consensus](https://docs.docker.com/engine/swarm/raft/)算法来管理Swarm的状态。为了管理Swarm，我们需要了解一些Raft的概念。

对于manager节点个数其实是没有限制的。manager节点数量需要从性能和容错性之间来权衡利弊。增加manager节点的数量可以更好的提高容错性。然而，大量的manager节点会降低数据写的性能，因为在Swarm状态更新时，更多的manager节点需要赞同状态更新（参考Raft Consensus算法，或者选举算法 ）。这意味着节点间需要大量的来回反复的网络通信。

Raft算法要求大部分的manager节点（这个数量在算法中要求超过manager节点个数半数以上）同意Swarm的状态更新，才能使状态更新成功，例如节点的增加和移除。节点之间的关系操作也同样受到这样的约束。

## Manager节点的法定人数（quorum）

如果Swarm中的mananger不能在选举算法中产生投票结果，Swarm就不能执行管理任务。如果我们有多个manager节点，那么节点数必须为2个以上。要保证正常运行，就必须保证法定人数（quorum）以及法定人数以上个manager节点可用。而且manager的个数建议采用奇数个，因为n（n为奇数）个manager节点和（n+1）个manager节点在算法上产生的效果是一样的。3个manager节点和4个mananger节点，都只允许最多1个manager节点不可用，5个manager节点和6个manager节点都只允许最多2个节点不可用。

一旦Swarm中的manager节点不能通过选举算法得到结果，worker节点上的Task仍然继续运行。但是，Swarm的节点不能被添加、更新和移除，并且新的task不能启动，已存在的task不能被停止、移动和更新。

## Manager节点配置静态IP地址

当初始化Swarm是，我们指定参数`--advertise-addr`来广播当前节点的IP地址给其他Manager节点。由于manager节点作为一个稳定的基础组件，我们应该给它分配一个固定的静态IP地址，以防止服务器重启造成IP地址改变。

如果Swarm重启，并且所有的manager节点都因为重启而获得一个新的IP地址，其他节点就无法链接到manager节点。Swarm就会出现故障被挂起。

Worker节点可以使用动态IP地址。

## Manager节点的容错

为了保证manager节点的容错性，我们最好将manager节点个数设定为奇数个。在网络被划分成2个部分情况下，奇数个manager节点能够较高程度的保证有投票结果的可能性。如果网络被划分成2个部分以上，投票有结果的可能性将不能被保证。

|Swarm节点数|法定票数|允许manager不可用个数|
|---|---|---|
|1|1|0|
|2|2|0|
|3|2|1|
|4|3|1|
|5|3|2|
|6|4|2|
|7|4|3|
|8|5|3|
|9|5|4|

举个例子，在一个Swarm中有5个节点，当失去3个时，就无法通过选举算法获得结果。因此就不能添加或者删除节点，知道其中一个不可用manager节点恢复为止。

可以将Swarm改变成为只有一个manager节点，但是不能将最后一个manager节点再降级成为worker节点，也就是说Swarm中至少要有一个manager节点。这样做仍然能够保证Swarm正常处理请求。将Swarm变成只有一个manager节点，是一个不安全的操作。如果失去唯一的一个manager，整个Swarm将不可用，直到我们使用参数`--fore-new-cluster`重启。

### 分布式manager节点

除了需要奇数个manager节点外，在部署manager节点时我们还需要留意数据中心的拓扑结构。最佳的容错方案是将manager节点分别部署在3个不同的可用区域。如果任何一个区域出现问题，Swarm仍然能够选举出Manager节点处理请求和重新平衡负载。

|manager节点个数|3个区域内分配个数|
|---|---|
|3|1-1-1|
|5|2-2-1|
|7|3-2-2|
|9|3-3-3|

### Manager-only节点

默认情况manager节点同时也扮演了worker节点的角色。这意味着调度器可以将task分配给一个manager节点来运行。 只要我们调度service时对CPU和内存采用**resouce constraints**，对于小型并且非关键集群分配task到manager节点的风险是相当低的。

然而，由于manager采用Raft consensus算法以一种一致的方式将数据进行复制，所以manager节点对数据复制操作对于资源匮乏显得很敏感。我们应该将manager节点上从一些有可能阻碍数据复制的进程中剥离出来，例如集群心跳检查，或者leader的选举过程。

为了避免对manager节点操作的影响，我们可以想worker节点那样，将manager节点的状态设置成为'DRAIN'：

```
docker node update --availability drain <NODE>
```

当节点状态为`DRAIN`时，调度器将该节点上的task重新分配到别的节点上运行，并会阻止新的task分配到该节点上。



