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