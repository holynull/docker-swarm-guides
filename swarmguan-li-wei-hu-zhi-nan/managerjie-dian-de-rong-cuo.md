# Manager节点的容错

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

## 分布式manager节点

除了需要奇数个manager节点外，在部署manager节点时我们还需要留意数据中心的拓扑结构。最佳的容错方案是将manager节点分别部署在3个不同的可用区域。如果任何一个区域出现问题，Swarm仍然能够选举出Manager节点处理请求和重新平衡负载。

|manager节点个数|3个区域内分配个数|
|---|---|
|3|1-1-1|
|5|2-2-1|
|7|3-2-2|
|9|3-3-3|

## Manager-only节点

默认情况manager节点同时也扮演了worker节点的角色。这意味着调度器可以将task分配给一个manager节点来运行。 只要我们调度service时对CPU和内存采用**resouce constraints**，对于小型并且非关键集群分配task到manager节点的风险是相当低的。

然而，由于manager采用Raft consensus算法以一种一致的方式将数据进行复制，所以manager节点对数据复制操作对于资源匮乏显得很敏感。我们应该将manager节点上从一些有可能阻碍数据复制的进程中剥离出来，例如集群心跳检查，或者leader的选举过程。

为了避免对manager节点操作的影响，我们可以想worker节点那样，将manager节点的状态设置成为'DRAIN'：

```
docker node update --availability drain <NODE>
```

当节点状态为`DRAIN`时，调度器将该节点上的task重新分配到别的节点上运行，并会阻止新的task分配到该节点上。

