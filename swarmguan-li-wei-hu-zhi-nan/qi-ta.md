# 其他

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

## 强行移除节点

有些情况下，需要在移除节点之前关闭节点。如果节点变成不可达，或者没有响应。我们可以像如下方法使用`--force`参数强行移除节点。

```
$ docker node rm node9

Error response from daemon: rpc error: code = 9 desc = node node9 is not down and can't be removed

$ docker node rm --force node9

Node node9 removed from swarm
```

强行移除节点之前，必须将节点先降级成为worker节点。降级Manager节点时，请确保剩下的manager个数为一个奇数。

## 强制平衡

通常情况下我们不需要重新平衡task在节点上的分配。只有当我们新加了节点，或者一个节点重新接入Swarm时，Swarm不会自动将已存在的task按照新的节点个数从新在节点之间分配。这种设计的初衷是防止增加节点时，如果task重新分配会造成客户端访问的阻断。

在Docker 1.13或者更高版本，我们可以在命令`docker service update`使用使用`--force`或者`-f`参数来从新分配task。这会造成Service的task滚动重启。



























