# 服务中task的数量

一旦我们在swarm中部署了service，我们就可以通过Docker CLI来指定运行service的container的数量。我们将这些container称之为task。

1. 通过ssh登录到manager节点。

2. 运行如下命令来改变正在运行的service的状态。

    ```
    $ docker service scale <SERVICE-ID>=<NUMBER-OF-TASKS>
    ```
    
    例如：
    
    ```
    $ docker service scale helloworld=5

    helloworld scaled to 5
    ```
    
3. 运行命令`docker service ps <SERVICE-ID>`，来查看修改后的service的状态：

    ```
    $ docker service ps helloworld

    NAME                                    IMAGE   NODE      DESIRED STATE  CURRENT STATE
    helloworld.1.8p1vev3fq5zm0mi8g0as41w35  alpine  worker2   Running        Running 7 minutes
    helloworld.2.c7a7tcdq5s0uk3qr88mf8xco6  alpine  worker1   Running        Running 24 seconds
    helloworld.3.6crl09vdcalvtfehfh69ogfb1  alpine  worker1   Running        Running 24 seconds
    helloworld.4.auky6trawmdlcne8ad8phb0f1  alpine  manager1  Running        Running 24 seconds
    helloworld.5.ba19kca06l18zujfwxyc5lkyn  alpine  worker2   Running        Running 24 seconds
    ```
    
    我们可以看到新生成了4个服务实例，分别分散部署在swarm的不同节点上，并且有一个部署在manager节点上。
    
4. 在不同的节点上运行`docker ps`可以查看每个节点上运行的task的情况。例如，我们在manager1节点上运行命令：

    ```
    $ docker ps

    CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
    528d68040f95        alpine:latest       "ping docker.com"   About a minute ago   Up About a minute                       helloworld.4.auky6trawmdlcne8ad8phb0f1
    ```