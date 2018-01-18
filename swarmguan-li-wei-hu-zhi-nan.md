> **编者提示：**法定人数可以理解为超过参加选举的总人数的半数以上的一个整数。选举票数等于或者大于法定人数，选举才有投票结果，如果小于法定人数，则选举是没有投票结果的（投票的人是可以投弃权票或者缺席选举的）。例如，3个manager的法定人数就是2，其中有一个缺席，同样有可能有投票结果，只要剩下的2个manager投同样的票，投票数就会超过3的半数。4个manager中，法定人数是3，而不是2。与3个manager情况相同，当缺席1个人时还可能有投票结果。但是缺席2个人时，是没有办法得到一个投票结果的，因为票数如何也超不过4的半数。所以，3个manager和4个mananger的情况都只能允许1个manager缺席选举或投弃权票，两种情况在算法上要达到的结果是一样的。

# Swarm管理维护指南

当我们运行一个Docker Engine集群时，manage节点是管理Swarm和存储Swarm状态的关键组件。所以为了更好的管理维护，了解manager节点的一些关键特性是很重要的。

## Swarm中的manager节点

Swarm中的manager节点使用[Raft Consensus](https://docs.docker.com/engine/swarm/raft/)算法来管理Swarm的状态。为了管理Swarm，我们需要了解一些Raft的概念。

对于manager节点个数其实是没有限制的。manager节点数量需要从性能和容错性之间来权衡利弊。增加manager节点的数量可以更好的提高容错性。然而，大量的manager节点会降低数据写的性能，因为在Swarm状态更新时，更多的manager节点需要赞同状态更新（参考Raft Consensus算法，或者选举算法 ）。这意味着节点间需要大量的来回反复的网络通信。

Raft算法要求大部分的manager节点（这个数量在算法中称之为法定人数，quorum）同意Swarm的状态更新，才能使状态更新成功，例如节点的正佳和移除。节点之间的关系操作也同样受到这样的约束。

### Manager节点的法定人数（quorum）

如果Swarm中的mananger不能在选举中产生投票结果，Swarm就不能执行管理任务。如果我们有多个manager节点，那么节点数必须为2个以上。要保证正常运行，就必须保证半数以上的manager节点可用。而且manager的个数建议采用奇数个，因为n（n为奇数）个manager节点和（n+1）个manager节点在算法上产生的效果是一样的。3个manager节点和4个mananger节点，都只允许最多1个manager节点不可用，5个manager节点和6个manager节点都只允许最多2个节点不可用。

一旦Swarm中的manager节点不能产生投票结果，worker节点上的Task仍然继续运行。但是，Swarm的节点不能被添加、更新和移除，并且新的task不能启动，已存在的task不能被停止、移动和更新。

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

可以将Swarm改变成为只有一个manager节点，但是不能将最后一个manager节点再降级成为worker节点，也就是说Swarm中至少要有一个manager节点。这样做仍然能够保证Swarm正常处理请求。将Swarm变成只有一个manager节点，是一个不安全的操作。如果失去唯一的一个manager，整个Swarm将不可用，知道我们使用参数`--fore-new-cluster`重启。

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

## Worker节点负载均衡

通过添加节点来使集群的负载平衡。只要workder节点符合service的要求，service的task会被均匀的分配到不同的节点上。当限制service在指定类型的节点上运行时，例如指定节点CPU的个数或者内存的容量，节点如果没有达到这些指定的要求，task是不会在这样的节点上运行的。

## 监控Swarm健康

我们可以通过通过查询Docker API的HTTP端点`/nodes`来监控manager节点的健康状况。详情请参考[Nodes API docuemntation](https://docs.docker.com/engine/api/v1.25/#tag/Node)。

在命令行中运行`docker node inspect <id-node>`，来查询节点。例如，查询mananger节点的可达性：

```
docker node inspect manager1 --format "{{ .ManagerStatus.Reachability }}"
reachable
```

查询worker节点接收的task的状态：

```
docker node inspect manager1 --format "{{ .Status.State }}"
ready
```

上面的命令我们可以看到`manager1`节点可达性为`reachable`，作为workder几点的状态为`ready`。

`unreachable`状态则表示其他的manager节点服务到达这个节点。这种情况下就需要通过已下办法来恢复该节点：

- 重启Docker daemon

- 重启服务器

- 如果既不重启Docker daemon也不重启服务器，那么就需要添加一个新的manager节点，或者将一个worker节点升级成为一个manager节点。我们也需要通过命令`docker node demote <NODE>`和`docker node rm <id-node>`彻底移除该节点。

我们也可以在manager节点上执行`docker node ls`来查看节点：

```
docker node ls
ID                           HOSTNAME  MEMBERSHIP  STATUS  AVAILABILITY  MANAGER STATUS
1mhtdwhvsgr3c26xxbnzdc3yp    node05    Accepted    Ready   Active
516pacagkqp2xc3fk9t1dhjor    node02    Accepted    Ready   Active        Reachable
9ifojw8of78kkusuc4a6c23fx *  node01    Accepted    Ready   Active        Leader
ax11wdpwrrb6db3mfjydscgk7    node04    Accepted    Ready   Active
bb1nrq2cswhtbg4mrsqnlx1ck    node03    Accepted    Ready   Active        Reachable
di9wxgz8dtuh9d2hn089ecqkf    node06    Accepted    Ready   Active
```

## Manager节点的故障排查

不能从其他节点上将`raft`目录拷贝到mananger节点上，然后重启节点。对于每一个节点ID数据目录是唯一对应的。加入Swarm时每一个节点ID只能被节点使用一次。节点ID控件是全局唯一的。

重新将manager节点加入到集群中，需要如下操作：

1. 将节点降级为worker节点。`docker node demote <NODE>`

2. 将节点移除。`docker node rm <NODE>`

3. 将节点重新加入回来，并使用新的状态。`docker swarm join`

## 强行移除节点

有些情况下，需要在移除节点之前关闭节点。如果节点变成不可达，或者没有响应。我们可以像如下方法使用`--force`参数强行移除节点。

```
$ docker node rm node9

Error response from daemon: rpc error: code = 9 desc = node node9 is not down and can't be removed

$ docker node rm --force node9

Node node9 removed from swarm
```

强行移除节点之前，必须将节点先降级成为worker节点。降级Manager节点时，请确保剩下的manager个数为一个奇数。

## 备份

Docker的manager节点将Swarm的状态和日志存储在`/var/lib/docker/swarm/`目录下。在1.13或者更高版本中，这个目录下包含加密Raft日志的密钥。如果没有这些密钥，就没有办法恢复Swarm。

我们可以在任何一个manager节点上，按照下面的过程进行备份。

1. 如果Swarm`auto-lock`设置被打开，我们需要`unlock key`才能进行从备份恢复。如何多的`unlock key`请参看[Lock your swarm to protect its encryption key](https://docs.docker.com/engine/swarm/swarm_manager_locking/)。

2. 在备份之前在manager节点停止Docker，这样依赖在备份期间就不会有数据发生改变。不建议采用Manager在备份过程中一直运行的热备份，因为在恢复时结果是不可预知的。manager节点停止的同时产生的数据不会被备份。
    
    > **注意：**要确保manager节点的选举算法维持不变。在一个manager节点关闭时，如果这时有节点丢失，Swarm变得容易无法完成选举算法。所以需要权衡一下留下运行的manager节点的个数。如果经常有规律的进行备份，可以考虑创建一个5个manager节点的Swarm，这样的话就可以在备份时允许再失去一个manager节点，而不影响service的运行。
    
3. 备份目录`/var/lib/docker/swarm`

4. 重启manager节点

## 灾备恢复

### 从备份恢复

备份了Swarm之后，通过下面的过程将备份的数据恢复到一个新的集群上：

1. 在要恢复Swarm的主机上关闭Docker。

2. 移除目录`/var/lib/docker/swarm`。

3. 将之前备份的内容恢复到目录`/var/lib/docker/swarm`。

    > **注意：新的节点将跟以前节点一样的磁盘加密密钥。此时不能修改这个磁盘加密密钥。**
    在Swarm`auto-lock`打开的情况下，`unlock key`与之前的备份的`unlock-key`是一样的，也会要进行恢复。
    
4. 在恢复Swarm的节点上开启Docker。解锁Swarm如果需要的话。使用下面的命令重新初始化Swarm，这样的话这个节点将不会再去尝试链接之前的Swarm节点。

    ```
    $ docker swarm init --force-new-cluster
    ```
    
5. 检查确认Swarm的状态。也许这一步还要包含应用测试，或者通过玲玲`docker service ls`来查看确定service的状态。

6. 如果使用`auto-lock`，则需要轮换`unlock key`。详情请参看[rotate unlock key](https://docs.docker.com/engine/swarm/swarm_manager_locking/#rotate-the-unlock-key)。

7. 添加manager节点和worker节点，使集群容量达到跟之前一下样。

8. 在新的集群上恢复之前的应用备份。

























