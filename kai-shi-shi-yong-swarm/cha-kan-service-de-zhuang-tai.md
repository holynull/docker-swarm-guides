# 查看service的状态

在swarm中部署了service之后，我们就可以通过Docker CLI来查看正在运行的service的状态。

1. 通过ssh登录到manager节点上。

2. 运行命令`docker service inspect --pretty <SERVICE-ID>`查看service的详细信息。
我们来看一下`helloworld`服务的详细信息：

    ```
     [manager1]$ docker service inspect --pretty helloworld

     ID:		 9uk4639qpg7npwf3fn2aasksr
     Name:		helloworld
     Service Mode:	REPLICATED
      Replicas:		1
     Placement:
     UpdateConfig:
      Parallelism:	1
     ContainerSpec:
      Image:		alpine
      Args:	ping docker.com
     Resources:
     Endpoint Mode:  vip
     ```
 
    > 提示：如果需要返回json格式的数据，去掉参数`--pretty`就可以了。
 
    ```
     [manager1]$ docker service inspect helloworld
    [{
        "ID": "9uk4639qpg7npwf3fn2aasksr",
        "Version": {
            "Index": 418
        },
        "CreatedAt": "2016-06-16T21:57:11.622222327Z",
        "UpdatedAt": "2016-06-16T21:57:11.622222327Z",
        "Spec": {
            "Name": "helloworld",
            "TaskTemplate": {
                "ContainerSpec": {
                    "Image": "alpine",
                    "Args": [
                        "ping",
                        "docker.com"
                    ]
                },
                "Resources": {
                    "Limits": {},
                    "Reservations": {}
                },
                "RestartPolicy": {
                    "Condition": "any",
                    "MaxAttempts": 0
                },
                "Placement": {}
            },
            "Mode": {
                "Replicated": {
                    "Replicas": 1
                }
            },
            "UpdateConfig": {
                "Parallelism": 1
            },
            "EndpointSpec": {
                "Mode": "vip"
            }
        },
        "Endpoint": {
            "Spec": {}
        }
    }]
    ```
 
3. 通过运行命令``来查看service在哪些节点上运行。

    ```
    [manager1]$ docker service ps helloworld

    NAME                                    IMAGE   NODE     DESIRED STATE  LAST STATE
    helloworld.1.8p1vev3fq5zm0mi8g0as41w35  alpine  worker2  Running        Running 3 minutes
    ```
    
    可以看见`helloworld`的一个实例运行在`worker2`节点上。我们也有可能看见这个实例运行在manager节点上，因为manager节点也可以想worker节点一样执行task。
    
    我们通过这些服务的运行状态数据可以看到，服务正在运行的状态是否与我们最初的设置保持一致。
    
4. 在执行task的节点上，我们可以通过`docker ps`命令来查看在该节点上运行的container。
    
    我们在worker2节点上，可以看到如下内容：
 
     ```
      worker2]$docker ps

    CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
    e609dde94e47        alpine:latest       "ping docker.com"   3 minutes ago       Up 3 minutes                            helloworld.1.8p1vev3fq5zm0mi8g0as41w35
    ``` 

