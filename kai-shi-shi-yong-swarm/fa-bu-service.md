# 发布service

在创建了Swarm之后，我们就可以发布一个service了。虽然我们之前创建了worker节点，但是对于发布service来说这并不是必须的。

1. 通过ssh登录到manager节点上，本教程中我们登录到manager1主机上。
    
2. 执行如下命令：

  ```
  $ docker service create --replicas 1 --name helloworld alpine ping docker.com

  9uk4639qpg7npwf3fn2aasksr
  ```  
  
  - `docker service create`命令创建了一个service
  
  - `--name`将service命名为`helloworld`
  
  - `--replicas`指定运行1个实例
  
  - `alpine ping docker.com`定义了service将以`Alpine Linux`为镜像的container运行，并在container内部执行命令`ping docker.com`
  
3. 执行`docker service ls`命令，查看正在运行的service。

  ```
  $ docker service ls

  ID            NAME        SCALE  IMAGE   COMMAND
  9uk4639qpg7n  helloworld  1/1    alpine  ping docker.com
  ```