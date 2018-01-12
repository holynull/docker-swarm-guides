# 增加节点

在manager节点上创建了Swarm之后，我们就可以来添加worker节点了。

  1. 通过ssh登录到worker节点所在的主机上。本教程中则登录到worker1主机上。
    
  2. 执行创建Swarm时，运行命令`docker swarm init`输出产生的，用来加入Swarm的命令。我们将创建一个worker节点并加入到之前创建的Swarm中。
    
  ```
  $ docker swarm join \
  --token  SWMTKN-1-49nj1cmql0jkz5s954yi3oex3nedyz0fb0xx14ie39trti4wxv-8vxv8rssmk743ojnwacrr2e7c \
  192.168.99.100:2377

  This node joined a swarm as a worker.
  ```
    
  如果命令无法执行，可以在manager节点上执行下面的命令，重新获得以worker身份加入Swarm的命令。
    
  ```
  $ docker swarm join-token worker

  To add a worker to this swarm, run the following command:

  docker swarm join \
  --token SWMTKN-1-49nj1cmql0jkz5s954yi3oex3nedyz0fb0xx14ie39trti4wxv-8vxv8rssmk743ojnwacrr2e7c \
  192.168.99.100:2377
  ```
    
  3. 通过ssh登录到第二个要以worker身份加入Swarm的宿主机上。本教程中我们登录到worker2主机上。
    
  4. 执行创建Swarm时，运行命令`docker swarm init`输出产生的，用来加入Swarm的命令。我们将创建第二个worker节点并加入到之前创建的Swarm中。
    
  ```
  $ docker swarm join \
  --token SWMTKN-1-49nj1cmql0jkz5s954yi3oex3nedyz0fb0xx14ie39trti4wxv-8vxv8rssmk743ojnwacrr2e7c \
  192.168.99.100:2377

  This node joined a swarm as a worker.
  ```
    
  5. 通过ssh登录到manager节点，执行`docker node ls`命令，查看节点信息。
    
  ```
  ID                           HOSTNAME  STATUS  AVAILABILITY  MANAGER STATUS
  03g1y59jwfg7cf99w4lt0f662    worker2   Ready   Active
  9j68exjopxe7wfl6yuxml7a7j    worker1   Ready   Active
  dxn1zf6l61qsb1josjja83ngz *  manager1  Ready   Active        Leader
  ```
    
  `MANAGER`列标记哪些节点是manager节点。worker节点该列则为空。
    
  Swarm相关的管理命令，如`docker node ls`只能在manager节点上运行。
