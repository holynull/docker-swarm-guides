> **编者提示：**法定人数（quorum）可以理解为一个投非弃权票的人数，这个人数保证投票不会完全没有结果。选举结果是要求票数超过参加投票的总人数（包括投弃权票的人）的半数以上的人赞成的意向。投非弃权票的人数，如果小于法定人数，则选举是没有投票结果的。例如，3个manager的法定人数就是2，其中有1个弃权，同样有可能有投票结果，只要剩下的2个manager投同样的票，投票数就会超过3的半数。4个manager中，法定人数是3，而不是2。与3个manager情况相同，当1个人弃权时还可能有投票结果。但是2个人弃权时，是没有办法得到一个投票结果的，因为票数如何也超不过4的半数。所以，3个manager和4个mananger的情况都只能允许1个manager投弃权票，两种情况在算法上要达到的结果是一样的。在失去法定人数（quorum）的情况下，选举算法是无论如何也没有结果的。

# Swarm管理维护指南

当我们运行一个Docker Engine集群时，manage节点是管理Swarm和存储Swarm状态的关键组件。所以为了更好的管理维护，了解manager节点的一些关键特性是很重要的。













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

### 选举计算恢复

Swarm具有容错性，并且能够从临时的节点不可用（机器重启或者崩溃重启）中，或者其他瞬时的错误中恢复。然而，选举算法出问题时，Swarm是不能自动恢复的。worker节点上的task还在继续执行，但是负责管理的任务将不能运行，包括扩展、或者更新service，添加或者移除节点。最好解决方法是将不可用的manager节点恢复回来。如果无法恢复manager节点怎么办？

在一个N个manager节点的Swarm中，必须保证法定人数以上的节点可用。例如，5个manager，最少要3个manager可用才并能够互相通信。如果可用的manager的数量低于法定人数，就不能对Swarm进行任何管理操作。当试图进行管理操作时，会出现如下错误提示：

```
Error response from daemon: rpc error: code = 4 desc = context deadline exceeded
```

最佳的回复方法当然是将不可用的manager节点恢复回来。如果发布恢复manager节点，我们只能通过使用参数`--force-new-cluster`来恢复manager节点。这个操作将降级所有的manager节点，除了执行命令的节点。然后再手动将worker节点升级回manager节点，保持与原来一样个数的manager节点为止。

```
# From the node to recover
docker swarm init --force-new-cluster --advertise-addr node01:2377

```

## 强制平衡

通常情况下我们不需要重新平衡task在节点上的分配。只有当我们新加了节点，或者一个节点重新接入Swarm时，Swarm不会自动将已存在的task按照新的节点个数从新在节点之间分配。这种设计的初衷是防止增加节点时，如果task重新分配会造成客户端访问的阻断。

在Docker 1.13或者更高版本，我们可以在命令`docker service update`使用使用`--force`或者`-f`参数来从新分配task。这会造成Service的task滚动重启。

























