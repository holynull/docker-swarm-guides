# DRAIN状态的节点

之前讲述的内容，都是在所有的节点都是可用的，状态都是`ACTIVE`。因此，manager可以将任务分配到所有节点上，所以到目前位置所有的节点都可以接受manager分配的任务。

有的时候，例如计划系统维护时，我们需要将节点的状态变成`DRAIN`。`DRAIN`状态将阻止节点接收manager分配的任务。同时也意味着，manager将停止该节点上的task，并将task转移到其他`ACTIVE`状态的节点上运行。

1. 通过ssh登录到manager节点。

2. 查看我们有哪些`ACTIVE`状态的节点：

    ```
    $ docker node ls

    ID                           HOSTNAME  STATUS  AVAILABILITY  MANAGER STATUS
    1bcef6utixb0l0ca7gxuivsj0    worker2   Ready   Active
    38ciaotwjuritcdtn9npbnkuz    worker1   Ready   Active
    e216jshn25ckzbvmwlnh5jr3g *  manager1  Ready   Active        Leader
    ```
    
3. 部署一个`redis`服务：

    ```
    $ docker service create --replicas 3 --name redis --update-delay 10s redis:3.0.6

    c5uo6kdmzpon37mgj9mwglcfw
    ```
    
4. 运行命令`docker service ps redis`查看task分配的情况：

    ```
    $ docker service ps redis

    NAME                               IMAGE        NODE     DESIRED STATE  CURRENT STATE
    redis.1.7q92v0nr1hcgts2amcjyqg3pq  redis:3.0.6  manager1 Running        Running 26 seconds
    redis.2.7h2l8h3q3wqy5f66hlv9ddmi6  redis:3.0.6  worker1  Running        Running 26 seconds
    redis.3.9bg7cezvedmkgg6c8yzvbhwsd  redis:3.0.6  worker2  Running        Running 26 seconds
    ```
    我们看见每一个node上都被分配了一个task。由于实际实验环境不同，也可能并不是这样分配的。
    
5. 运行命令`docker node update --availability drain <NODE-ID>`将节点状态设置为`DRAIN`。

    ```
    docker node update --availability drain worker1

    worker1
    ```
    
6. 检查节点状态：

    ```
    $ docker node inspect --pretty worker1

    ID:			38ciaotwjuritcdtn9npbnkuz
    Hostname:		worker1
    Status:
     State:			Ready
     Availability:		Drain
    ...snip...
    ```
    
    这里已经完成节点`DRAIN`状态的变更。
    
7. 运行命令`docker service ps redis`查看manager如何将task重新分配：

    ```
    $ docker service ps redis

    NAME                                    IMAGE        NODE      DESIRED STATE  CURRENT STATE           ERROR
    redis.1.7q92v0nr1hcgts2amcjyqg3pq       redis:3.0.6  manager1  Running        Running 4 minutes
    redis.2.b4hovzed7id8irg1to42egue8       redis:3.0.6  worker2   Running        Running About a minute
     \_ redis.2.7h2l8h3q3wqy5f66hlv9ddmi6   redis:3.0.6  worker1   Shutdown       Shutdown 2 minutes ago
    redis.3.9bg7cezvedmkgg6c8yzvbhwsd       redis:3.0.6  worker2   Running        Running 4 minutes
    ```
    
    manager为了使`redis`的状态保持跟最初设定一致，将`DRAIN`状态节点上的task转移到一个`ACTIVE`状态的节点上运行。
    
8. 运行命令`docker node update --availability active <NODE-ID>`使节点状态变回到`ACTIVE`。

    ```
    $ docker node update --availability active worker1

    worker1
    ```