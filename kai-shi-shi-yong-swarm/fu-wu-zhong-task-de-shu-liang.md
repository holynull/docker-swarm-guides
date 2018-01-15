# 服务中task的数量

一旦我们在swarm中部署了service，我们就可以通过Docker CLI来指定运行service的container的数量。我们将这些container称之为task。

1. 通过ssh登录到manager节点。

2. 运行如下命令来改变正在运行的service的状态。

    ```
    $ docker service scale <SERVICE-ID>=<NUMBER-OF-TASKS>
    ```